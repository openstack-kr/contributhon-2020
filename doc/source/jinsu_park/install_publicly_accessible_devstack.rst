===========================================
퍼블릭하게 접속할 수 있는 DevStack 구축하기
===========================================

----
요약
----

데브 스택을 설치하고 팀원들과 클라이언트에 대한 디버깅을 진행하는 다양한 방법을 다뤄보았습니다. 
파이참과 vscode의 remote debugging을 이용한 경우가 대부분입니다. 
client를 원격에서 접속하는 것이 아닌, 원격의 interpreter와 client를 이용했던 이유는 크게 두 가지 정도입니다.

- DevStack 설치시에 기본적으로 외부 IP로 등록되는 IP가 우리가 public하게 접속할 수 있는 IP와 다르다.
    - 하지만 설정을 통해 외부 IP를 public IP로 변경할 수 있었습니다.

- 이런 저런 서버에 직접 접근할 일이 있는 경우는 원격 디버깅이 편하다.
    - 하지만 저의 컨트리뷰션 동안에는 주로 ``openstack client`` 에 자체에 대한 작업을 진행했기에, 이 경우가 해당되지 않았습니다.

따라서 저는 DevStack을 추가적인 몇 가지 설정과 함께 원격에서 접속할 수 있도록 설치하여
이후 이어질 PyCharm을 이용한 ``openstack client`` 디버깅 작업을 좀 더 수월하게 진행했습니다.

----------------------------
DevStack 설치 준비하기
----------------------------

https://docs.openstack.org/devstack/latest/ 를 바탕으로 install하되
`local.conf` 에서 우리는 remote에서도 접속이 가능하도록 설정을 해줘야한다.

https://docs.openstack.org/devstack/latest/configuration.html 은 configuration에
대한 문서인데, 이 부분을 보면 다음과 같은 내용이 존재합니다.

> ``HOST_IP`` is normally detected on the first run of stack.sh but often is indeterminate on later runs due to the IP being moved from an Ethernet interface to a bridge on the host. Setting it here also makes it available for openrc to set OS_AUTH_URL. HOST_IP is not set by default.

즉 이더넷 네트워크 인터페이스의 주 IP를 따라간다는 것인데, 원격에서도 우리의 server(devstack)으로

요청을 보내기 위해서는 cafe24 등의 ``public IP`` 로 ``host ip`` 를 변경해주어야하는 것입니다.

.. code-block::

    # local.conf
    [[local|localrc]]
    ADMIN_PASSWORD=secret
    DATABASE_PASSWORD=$ADMIN_PASSWORD
    RABBIT_PASSWORD=$ADMIN_PASSWORD
    SERVICE_PASSWORD=$ADMIN_PASSWORD

    HOST_IP=<183.x.x.x와 같은 public IP>
    disable_service etcd3

그리고 openstack client를 통해 API를 이용하기 위한 설정을 담는 ``openrc``를 작성해주세요.
이렇게 ``HOST_IP``를 수동으로 설정하는 경우 ``kubernetes``의 데이터 저장소인 ``etcd`` 설정에서
오류가 나더라구요. 자세히 리서치해보진 못했지만 우선은 etcd를 disable하는 정도로 넘어갑니다.

.. code-block::

    $ sudo ip addr add 183.x.x.x/32 dev ens3

그리고 자신의 Public IP를 server instance 자신도 자기의 IP로 인식할 수 있도록
위의 커맨드를 입력해 ens3 interface 에 Public IP를 secondary IP로 추가해줍니다.

(이 부분의 원리는 정확히 이해하지못했습니다. 아시는 분이 계시면 알려주시기바랍니다!)

.. code-block:: shell

    # openrc
    export OS_PROJECT_DOMAIN_ID=default
    export OS_USER_DOMAIN_ID=default
    export OS_PROJECT_NAME=demo
    export OS_TENANT_NAME=demo
    export OS_USERNAME=admin
    export OS_PASSWORD=secret
    export OS_AUTH_URL=http://183.x.x.x/identity
    export OS_IDENTITY_API_VERSION=3

위와 같이 devstack을 clone 받은 경로 안에 ``local.conf`` 를 수정해주세요.

-----------------------
DevStack 설치하기
-----------------------

.. code-block:: shell

    $ ./stack.sh

clone 받은 devstack 경로 안에 ``local.conf`` 를 잘 작성해주었다면
devstack 경로 안에서 ``./stack.sh`` 를 입력하는 것 만으로도 devstack 구축이 완료됩니다.

.. code-block:: shell

    $ source openrc

설치가 완료된 후에는 위의 command를 통해 openrc를 적용시켜주세요.

   .. image:: images/security.png

이후 필요에 따라 보안그룹 혹은 방화벽을 수정해줘야합니다.
기본적으로 SSH는 열려있었을 것이고, 브라우저 접속을 위한 80포트,
openstack client를 이용한 API 수행을 위한 TCP 6060포트와 9696포트를
추가로 열어줘야했던 것으로 기억합니다.

-----------------------
원격에서 접속해보기
-----------------------

   .. image:: images/login.png

우선은 기본적으로 devstack이 잘 뜨는지 확인해보기위해 웹브라우저로 접속해봅니다.
http://자신의PublicIP 를 통해 접속해봅시다.

기본적으로는 admin/secret을 통해 접속할 수 있으니 로그인도 해봅시다.

예시) openstack server list
=======================================

..  code-block:: shell

    $ openstack user list
    +----------------------------------+-----------+
    | ID                               | Name      |
    +----------------------------------+-----------+
    | 40ee1fb2103c40d5b077a98a0318d225 | admin     |
    | 5cd3c4104d214e0992f671ae7408a001 | demo      |
    | 5b4ee20cae26492db5809f2fd8f15659 | alt_demo  |
    | 02d8ea737a3248bb94a9a61fe53d0921 | nova      |
    | be0b5fa743ab4a588708515c5c5c7645 | glance    |
    | cf795d9a36804446b88a4d8b5fccf41c | cinder    |
    | 3bf64dd3cc0c4f20ab78cc10d239d242 | neutron   |
    | ea2ead4f752a4b668421fada1cadb78a | placement |
    +----------------------------------+-----------+

예시) 🌟 Pycharm에서 Python Default Interpreter로 디버깅해보기
==============================================================================

.. code-block:: shell

    $ git clone https://github.com/openstack/python-openstackclient

``python-openstackclient`` 를 clone 받아준 뒤에 PyCharm으로 열어줍니다.

    .. image:: images/debug_1.png

    .. image:: images/debug_2.png

위와 같이 openrc 에 있던 데이터들 중 export를 제외해서 복사한 뒤 ``ADD CONFIGURATION`` 에서
Python interpreter를 설정해줍니다. Script Path는 openstackclient/shell.py의 절대경로를 입력해주세요.

    .. image:: images/debug_3.png

    .. image:: images/debug_4.png

그럼 위의 사진들과 같이 remote debugging 없이도, openstack client 자체가 remote openstack server로
요청을 보내기 때문에 Default 혹은 그 외의 Local interpreter로도 디버깅 작업이 가능해집니다.

-----------------
느낀 점
-----------------

이 글의 주요한 내용은 아니지만, 먼저 vscode와 pycharm을 통해 remote debugging 관련
내용을 정리해주신 팀원들께 정말 감사했고, 솔선수범하여 문서를 정리해주고, 알려주시던 모습들이
기억에 남습니다.
그리고 거기서 더 나아가 '원격에서 접속이 불가능했다'는 근본적인 불편 사항을 해결하기 위해
흔쾌히 도움을 주신 멘토님께 감사합니다.
당시 그 원리는 자세히 몰랐으나, 멘토님이 제시해 주신 해결 방안에 네트워크와 관련된 내용이
포함되어있었는데, 문득 다시 한 번 기본기와 네트워크의 중요성을 느낄 수 있었던 기회였던 것 같습니다.
만약 이 방법을 이용하지 않고, 단순히 remote debugging 만을 이용했다면 virtual env를 이용하거나
``openstack client`` 를 재설치하거나 git을 이용하는 등의 다양한 작업에서
제약이 따랐을 것 같은데, 이 방법으로 DevStack을 설치한 덕에 편리하게 작업을 수행할 수 있었습니다.

