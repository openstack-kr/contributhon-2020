==========================================
[학습] OpenStack 환경 설정 - DevStack 설치
==========================================

-------------------------------
DevStack이란?
-------------------------------
  DevStack is a series of extensible scripts used to quickly bring up a complete OpenStack environment based on the latest versions of everything from git master. It is used interactively as a development environment and as the basis for much of the OpenStack project’s functional testing.

  `OpenStack documentation - devstack <https://docs.openstack.org/devstack/latest/>`_

문서에서 설명한 그대로 DevStack은 git 기반으로 OpenStack 실행 환경을 빠르게 구현 할 수 있다. 실제 릴리즈된 버전에서 버그를 재현하고, 수정 및 기여하기 위해서는 DevStack으로 OpenStack 실행 환경을 구현해 놓아야 한다.

-------------------------------
DevStack 설치 과정
-------------------------------
`공식 문서 <https://docs.openstack.org/devstack/latest/>`_ 에서 설치하는 과정을 잘 설명해 놓았다. 특정 구간에서의 문제만 조심하면 빠르고 쉽게 DevStack을 설치 할 수 있을 것이다.

1. Linux 기반의 시스템 준비
==========================================
DevStack은 다음과 같은 운영체제(latest version)를 지원한다. 

* Ubuntu
* Fedora
    * CentOS/RHEL 8
* OpenSUSE


2. sudo 권한이 있는 non-root 계정 준비 
==========================================
.. code-block:: none
    
    $ sudo useradd -s /bin/bash -d /opt/stack -m stack
    $ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
    $ sudo su - stack

3. git을 이용한 DevStack 설치
==========================================
.. code-block:: none
    
    $ git clone https://opendev.org/openstack/devstack
    $ cd devstack

4. local.conf 작성
==========================================
git clone을 해온 devstack 디렉토리 내에 local.conf를 작성해주도록 하자.

local.conf는 상황에 따라서 적절히 조정해주도록 하자.

.. code-block:: none
    
    ADMIN_PASSWORD=secret
    DATABASE_PASSWORD=$ADMIN_PASSWORD
    RABBIT_PASSWORD=$ADMIN_PASSWORD
    SERVICE_PASSWORD=$ADMIN_PASSWORD
    HOST_IP=[접근하는 시스템의 공인IP]

5. stack.sh를 이용한 설치 진행
==========================================
1~4번 과정을 정상적으로 마쳤다면 git 디렉토리 내에서 stack.sh를 실행하면 정상적으로 설치가 진행된다.

.. code-block:: none

    ./stack.sh #디렉토리 내에서 실행하면 자동으로 설치가 진행된다.

6. 네트워크 설정 조정
==========================================
방화벽상에서 OpenStack에서 사용하는 포트를 개방해주어야 한다. firewall-cmd를 통해서 포트 개방을 해주도록 하자.

    22(ssh), 80(HTTP), 6080

