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

-----------------------
DevStack 설치하기
-----------------------

---------------------------
원격에서 접속하기 위한 설정
---------------------------

.. code-block:: shell

    # openrc.sh

    export OS_PROJECT_DOMAIN_ID=default
    export OS_USER_DOMAIN_ID=default
    export OS_PROJECT_NAME=demo
    export OS_TENANT_NAME=demo
    export OS_USERNAME=admin
    export OS_PASSWORD=secret
    export OS_AUTH_URL=http://183.111.252.230/identity
    export OS_IDENTITY_API_VERSION=3

.. code-block:: shell

    $ source openrc.sh

위의 command들을 이용해 openstack을 이용하기 위한 설정을 입력해주는 
``openrc.sh`` 를 작성해주시고 ``source`` 를 통해 적용해주세요.

-----------------------
원격에서 접속해보기
-----------------------

예시) openstack server list
=======================================

..  code-block:: shell

  $ openstack server list -c Name
                  
  +----------------------------------+
  | Name                             |
  +----------------------------------+
  | 92c925cf41e64d57b9aeb3c71c13dceb |
  | fe165d01854542299ab90a71f6617bd0 |
  | jinsutance-a                     |
  +----------------------------------+


-----------------
느낀 점
-----------------
이 글의 주 내용은 아니지만, 먼저 vscode와 pycharm을 통해 remote debugging 관련
내용을 정리해주신 팀원들께 정말 감사했고, 솔선수범하여 문서를 정리해주고, 알려주시던 모습들이
기억에 남습니다.
그리고 거기서 더 나아가 '원격에서 접속이 불가능했다'는 근본적인 불편 사항을 해결하기 위해
흔쾌히 도움을 주신 멘토님께 감사합니다.
당시 그 원리는 자세히 몰랐으나, 멘토님이 제시해 주신 해결 방안에 네트워크와 관련된 내용이
포함되어있었는데, 문득 다시 한 번 기본기와 네트워크의 중요성을 느낄 수 있었던 기회였던 것 같습니다.
만약 이 방법을 이용하지 않고, 단순히 remote debugging 만을 이용했다면 virtual env를 이용하거나
``openstack client`` 를 재설치하거나 git을 이용하는 등의 다양한 작업에서
제약이 따랐을 것 같은데, 이 방법으로 DevStack을 설치한 덕에 편리하게 작업을 수행할 수 있었습니다.