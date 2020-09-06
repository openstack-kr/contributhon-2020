==============================
Contribution: 첫 번째 이슈
==============================

첫 컨트리뷰션이 될 이슈를 `이것 <https://github.com/openstack-kr/contributhon-2020/issues/30>`_ 으로 결정했다.

-----------------------
이슈 분석 및 문제점
-----------------------

오픈스택 대시보드의 설정 화면에서 비밀번호를 변경하는 경우, ``500 Internal Server Error`` 가 발생한다. 문제를 찾기 위해 오픈스택 대시보드를 관리하는 프로젝트인 horizon의 로그 (/var/log/apache2/horizon_error.log) 를 확인했다.

.. code:: shell

    2020-09-05 07:55:33.775492 DEBUG keystoneauth.session GET call to identity for http://192.168.1.10/identity/v3/users/a6afb486ad9b4200a4ed37b6865f4e65 used request id req-7d8bed77-9849-4d5e-8a6c-5963bffbfea4
    2020-09-05 07:55:33.775714 DEBUG openstack_dashboard.api.keystone Creating a new keystoneclient connection to http://192.168.1.10/identity/v3.
    2020-09-05 07:55:33.835818 mod_wsgi (pid=18843): Exception occurred processing WSGI script '/opt/stack/horizon/openstack_dashboard/wsgi.py'.
    2020-09-05 07:55:33.835884 Traceback (most recent call last):
    2020-09-05 07:55:33.835899   File "/usr/local/lib/python3.6/dist-packages/django/core/handlers/wsgi.py", line 150, in __call__
    2020-09-05 07:55:33.835901     start_response(status, response_headers)
    2020-09-05 07:55:33.835910 ValueError: unicode object contains non latin-1 characters

로그를 확인하니 값을 넘겨줄 때 어디선가 인코딩에 문제가 생기는 것 같아 보였다.
아니나 다를까 대시보드의 언어 설정을 영어로 하면 아무 문제 없이 잘 동작하고, 한국어 및 다른 언어로 하면 500 에러가 발생했다. 

-------------
코드 분석
-------------

오픈스택 대시보드에서 패스워드 변경시 horizon 프로젝트에서 호출되는 코드는 아래와 같다.
(openstack_dashboard/dashboards/settings/password/forms.py)

.. code-block:: python3

    @sensitive_variables('data')
    def handle(self, request, data):
        user_is_editable = api.keystone.keystone_can_edit_user()
        user_id = request.user.id
        user = api.keystone.user_get(self.request, user_id, admin=False)
        options = getattr(user, "options", {})
        lock_password = options.get("lock_password", False)
        if lock_password:
            messages.error(request, _('Password is locked.'))
            return False

        if user_is_editable:
            try:
                api.keystone.user_update_own_password(request,
                                                      data['current_password'],
                                                      data['new_password'])
                response = http.HttpResponseRedirect(settings.LOGOUT_URL)
                msg = _("Password changed. Please log in again to continue.")
                utils.add_logout_reason(request, response, msg)
                return response
            except Exception as ex:
                exceptions.handle(request,
                                  _('Unable to change password: %s') % ex)
                return False
        else:
            messages.error(request, _('Changing password is not supported.'))
            return False

문제가 발생하는 부분은 위 파일의 77번 라인인 ``utils.add_logout_reason(request, response, msg)`` 이다. 코드에서는 위 함수로 메시지를 넘겨주게 되고, 해당 코드를 따라가 보면 아래와 같다.
(horizon/utils/functions.py)

.. code-block:: python3

  def add_logout_reason(request, response, reason, status='success'):
    # Store the translated string in the cookie
    lang = translation.get_language_from_request(request)
    with translation.override(lang):
      reason = str(reason)
      response.set_cookie('logout_reason', reason, max_age=10)
      response.set_cookie('logout_status', status, max_age=10)

코드에서는 사용자별로 설정해둔 언어에 따라서 전달받은 메시지를 번역하고 그 메시지를 쿠키에 담는다. 이게 에러가 발생하는 원인이 되는데, 그 이유는 쿠키에 번역된 메시지(한글)가 담기기 때문에 django에서 해석을 못 하게 되면서 500 에러가 발생하는 것으로 보인다.

---------------------
생각해본 해결 방법
---------------------

1. 메시지를 미리 인코딩해서 쿠키에 담기
2. 메시지를 꺼내는 부분을 건드려 보기


