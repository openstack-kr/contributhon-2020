===============================
Openstack_CLI
===============================

오픈스택을 이용하는 방법은 크게 3가지입니다.
---------------------------------------------

1. 웹 콘솔을 이용하는 방법
2. api를 이용하는 방법
3. cli를 이용하는 방법

1,3번은 결국 openstack api를 호출하게 됩니다.
우리는 그동안 웹 콘솔을 이용해서 오픈스택을 이용했고, 이제 openstack 이라는 명령어를 통해 간단하게 인스턴스 목록을 조회해봅니다.
먼저 웹 콘솔에서 로그인했던 것 처럼 cli에서 인증할 수 있는 정보를 환경변수에 저장해야합니다.

devstack 폴더를 보시면은. openrc 라는 파일이 있어요. 이제 여러분들이 공부해야하는 내용들이 여기서부터 나옵니다.

4-1. Openstack CLI 위한 환경설정
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   $pwd
   /opt/stack
   
   stack@devstack-master:~$ ls
   bin          data              glance    logs     nova          sandbox
   bindep-venv  devstack          horizon   neutron  placement     tempest
   cinder       devstack.subunit  keystone  noVNC    requirements
   
   stack@devstack-master:~$ cd devstack/
   stack@devstack-master:~/devstack$ ls | grep openrc
   openrc
   
   stack@devstack-master:~/devstack$ file openrc
   openrc: Bourne-Again shell script, ASCII text executable
   
   stack@devstack-master:~/devstack$ cat openrc | more
   #!/usr/bin/env bash
   #
   # source openrc [username] [projectname]
   #
   # Configure a set of credentials for $PROJECT/$USERNAME:
   #   Set OS_PROJECT_NAME to override the default project 'demo'
   #   Set OS_USERNAME to override the default user name 'demo'
   #   Set ADMIN_PASSWORD to set the password for 'admin' and 'demo'
   
   # NOTE: support for the old NOVA_* novaclient environment variables has
   # been removed.

.. code-block:: none

   stack@devstack-master:~/devstack$ source openrc demo
   WARNING: setting legacy OS_TENANT_NAME to support cli tools.
   
   stack@devstack-master:~/devstack$ source openrc admin
   WARNING: setting legacy OS_TENANT_NAME to support cli tools.
   
   
   //$source openrc admin
   //$source openrc demo
   // 를 통해 인증정보를 환경변수에 저장해줍니다.


4-2. Openstack CLI 명령어를 사용하여, 대쉬보드에서 생성한 server 를 확인
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

이제 명령어를 통해 오픈스택의 CLI로 빠집니다.

.. code-block:: none

   stack@devstack-master:~/devstack$ openstack
   (openstack)

여기서 server list 하면 우리가 아까 대쉬보드에서 만든 서버가 보여요.

.. code-block:: none

   stack@devstack-ussuri:~/devstack$ openstack server list
   +--------------------------------------+------+--------+---------------------------------------------------------------------+-------+---------+
   | ID                                   | Name | Status | Networks                                                            | Image | Flavor  |
   +--------------------------------------+------+--------+---------------------------------------------------------------------+-------+---------+
   | baad8346-1541-4f12-a7f3-a41681398679 | test | ACTIVE | private=fd82:98dc:7575:0:f816:3eff:fec7:61d8, 10.0.0.22, 172.24.4.3 |       | m1.nano |
   +--------------------------------------+------+--------+---------------------------------------------------------------------+-------+---------+


근데 얘는 , 어떻게 정보를 가지고 오느냐? 
오픈스택 클라이언트(openstackclient)라는 패키지가 설치되어있고,
Keystone한테 가서 Token을 얻어오고 이후에 Nova-API로 이 정도를 얻어온다.

이 과정은 Debug Option을 넣음으로써 눈으로 확인 할 수 있다. openstack server list --debug

.. code-block:: none

   stack@devstack-master:~/devstack$ openstack server list --debug


4-3. 그렇다면 실행시킨 "openstack" 이라는 명령어는 어디에 있는가?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   stack@devstack-master:~/devstack$ whereis openstack
   openstack: /etc/openstack /usr/local/bin/openstack
   
   stack@devstack-master:~/devstack$ which openstack
   /usr/local/bin/openstack


명령어의 위치를 확인하였습니다. 내용을 확인해봅니다.

.. code-block:: none

   stack@devstack-ussuri:~/devstack$ cat /usr/local/bin/openstack
   
   #!/usr/bin/python3.6
   # -*- coding: utf-8 -*-
   import re
   import sys
   from openstackclient.shell import main
   if __name__ == '__main__':
      sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
      sys.exit(main())
   
   
   stack@devstack-ussuri:~/devstack$


파일을 열어보니 openstackclient 라는 패키지에서 main 함수를 실행하는 것이 전부입니다.
그럼 아까 우리가 실행시켰던 openstack server list가 저 패키지 안에 있다는 뜻입니다.

저 패키지가 저장된 경로는 다음과 같이 확인할 수 있습니다.

.. code-block:: none

   stack@devstack-ussuri:~/devstack$ python3
   
   Python 3.6.9 (default, Jul 17 2020, 12:50:27)
   [GCC 8.4.0] on linux
   Type "help", "copyright", "credits" or "license" for more information.
   >>> import openstackclient
   >>> openstackclient
   <module 'openstackclient' from '/usr/local/lib/python3.6/dist-packages/openstackclient/__init__.py'>


이제 우리가 시도해볼 것 입니다.
openstack server list라는 명령어를 실행했을 때,
openstack이라는 명령어 부터 ~ 화면에 출력되어 반환하는 과정까지
어떤 파일들을 건드리는지 Tracing 하면 됩니다.

처음 하시는 분들이 있을 수도 있으니 물고기를 잡는 방법을 알려드리겠습니다.

* 먼저 단어들로 소스코드 전체를 뒤지세요.가장 잘 찾을 수 있는 방법이에요.
* 이 코드를 건드릴 거 같다? => print문 삽입. 
* 지름길로 갈게요. 컴퓨트에 서버가 있어요
* grep "def list(" -R
* recursive 하게.

우리는 이걸로 컨트리뷰션 할 거에요.
