=============================================================================
[이슈] $ openstack server show 에서 잘못된 형식의 json, yaml format output
=============================================================================

------------
이슈 소개
------------

openstack client를 통해 server show 명령을 수행하는 경우 기본적으로는
output format이 ``table`` 이고 그 경우 위와 같이 output이 줄바꿈형식으로 깔끔하게
출력됩니다.

   .. image:: images/bug_preview_1.png

하지만 문제는 위와 같이 table format이 아닌 `json`` 이나 `yaml`` 인 경우에도 단순히 줄바꿈과 쉼표를
이용해 출력이 된다는 것이었습니다.

보통은 ``json`` 이나 ``yaml`` 형태로 output을 얻는 경우는 이후에 그 output을 이용해
어떠한 프로그래밍적 로직을 수행하려하는 경우일텐데, 단순 줄바꿈과 쉼표를 이용하는 경우
output이 출력되는 경우에는 다시 output을 파싱해야하는 번거로움이 존재할 것입니다.

    .. image:: images/bug_preview_1.png

따라서 위의 사진처럼 output format이 ``json`` , ``yaml`` 인 경우에는 기존의 방식이 아닌, ``dictionary`` 나 ``object``, ``array``
형태로 output 출력하도록 패치했습니다.

------------------------
이슈 재현하기
------------------------

앞서 보여드린 사진과 같이 다음의 명령어를 수행할 경우, json, yaml에서도 단순히 줄바꿈과 쉼표를 통해
output을 출력하는 것을 보실 수 있습니다. (가독성을 위해 security_groups column만 이용해보겠습니다.)

.. code-block:: shell

    openstack server show umi0410 -c security_groups -f json
    {
      "security_groups": "name='default'\nname='tmp'"
    }

----------
코드 분석
----------

.. code-block:: python

    def take_action(self, parsed_args):
        compute_client = self.app.client_manager.compute
        server = utils.find_resource(compute_client.servers,
                                     parsed_args.server)

        if parsed_args.diagnostics:
            (resp, data) = server.diagnostics()
            if not resp.status_code == 200:
                self.app.stderr.write(_(
                    "Error retrieving diagnostics data\n"
                ))
                return ({}, {})
        else:
            data = _prep_server_detail(compute_client,
                                       self.app.client_manager.image, server,
                                       refresh=False)

        return zip(*sorted(data.items()))

``openstackclient/compute/v2/server.py`` 의 take_action 함수에서
API를 수행하고, result를 return하는 과정이 정의되어있습니다.
이 때 ``_prep_server_detail`` 이라는 함수가 ``server`` 객체의 정보를 담은
dictionary를 return 해줍니다.

하지만 문제는 dictionary로 변경하는 과정에서 줄바꿈형식으로 table처럼
formatting 하는 로직이 들어있었고, output format이 table이 아닌 경우는
이 부분이 불필요했습니다.

----------
코드 작성
----------

openstackclient/compute/v2/server.py 수정
========================================================================
.. code-block:: python

    # 변경한 코드
    def take_action(self, parsed_args):
    compute_client = self.app.client_manager.compute
    server = utils.find_resource(compute_client.servers,
                                 parsed_args.server)
    formatter_type = parsed_args.formatter

    if parsed_args.diagnostics:
        (resp, data) = server.diagnostics()
        if not resp.status_code == 200:
            self.app.stderr.write(_(
                "Error retrieving diagnostics data\n"
            ))
            return ({}, {})
    elif formatter_type == "table":
        data = _prep_server_detail(compute_client,
                                   self.app.client_manager.image, server,
                                   refresh=False)
    else:
        data = server.to_dict()
        data.update(
            {"properties": data.pop("metadata"),
             "volumes_attached":
                 data.pop("os-extended-volumes:volumes_attached")}
        )

    return zip(*sorted(data.items()))

따라서 result로서 사용되는 dictionary를 만드는 로직을 table인 경우와 그 외의
경우로 나눠서 진행하도록 하였습니다.

수정 내역에 따라서 test code도 수정
========================================================================

기존의 test code를 까보고 꽤나 놀라웠습니다. test code에서는 앞서 말씀드린
예상된 불편함이었던 output을 다시 파싱해서 테스트를 통과하는지를 판단하고있었습니다.

.. code-block:: python

    self.assertTrue(volumes_attached.startswith('id='))
        attached_volume_id = volumes_attached.replace('id=', '')

이런 식으로 ``dictionary`` 로서 ``["id"]`` 의 값을 확인하는 것이 아니라
단순 문자열 id=의 형태로 시작하는지를 판단하는 test도 있었고,

.. code-block:: python

    # Really, shouldn't this be a dict?
    self.assertEqual(
        "a='b', c='d'",
        cmd_output['properties'],
    )

이렇게 테스트 코드 내의 주석에서도 불편을 호소하는 경우가 있었습니다.

데브 스택을 설치하고 팀원들과 클라이언트에 대한 디버깅을 진행하는 다양한 방법을 다뤄보았습니다.

.. code-block:: python

    self.assertEqual(
        {'a': 'b', 'c': 'd'},
        cmd_output['properties'],
    )

따라서 저는 위와 같이 dictionary로서 접근할 수 있도록하였고,
Zuul에서 모든 test를 통과하도록 수정하였습니다.


----------------------------------
Gerrit에 review 요청
----------------------------------

    Fix miss formatting array to print in ShowServer

    Currently, When the formatter is not table and result
    of content contains list, the items of list were printed as a
    concatenate with comma.

    In this patch, if the output format is not table, the output
    of list will follow the output format.
    (e.g. json output = [ ])
    This will put right output format for not only table
    but also both json and yaml.

    Change-Id: Ibf88593b1935c7ff42a5512136e0eca9f8466343
    Story: 2007755


수정내역을 반영해 Gerrit에 review를 요청했고, https://review.opendev.org/#/c/746369/
에서 확인해볼 수 있습니다.

-----------------
느낀 점
-----------------

누군가가 올린 흥미로운 이슈를 찾는 것, 재현하는 것, 픽스하는 것은 재미있는 경험이었습니다.

그것을 위해 ``storyboard`` , ``launchpad`` 등에 가입하고, ``tox`` 를 통해 test를 돌려보고,
``Zuul`` 을 통해 다시 자동화테스트를 거쳐 ``gerrit`` 을 통해 review를 받는 일련의 과정들이
혼자였다면 어렵게만 느껴졌을텐데, 친절한 멘토님의 도움과 적극적인 팀원들의 참여가 있었기에
수월히 해낼 수 있었던 것 같아 감사했고, 재미있었습니다.