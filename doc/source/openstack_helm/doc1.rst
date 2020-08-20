Openstack helm 개발 환경 구성 
=====

해당 문서는 "https://www.careyscloud.ie/openstack_kube" 페이지를 참고 하여 작성되었습니다.

시작하기 전에 시스템이 최신 소프트웨어로 업데이트되었는지 확인하고 몇 가지 종속성을 설치해야 한다.

.. code::
 
    # sudo apt-get update -y
    # sudo apt-get upgrade -y
    # sudo apt-get install ca-certificates git make jq nmap curl uuid-runtime -y
 


Github에서 openstack-helm-infra 및 openstack-helm 리포지토리를 복제

.. code::
 
    # git clone https://github.com/openstack/openstack-helm-infra.git
    # git clone https://github.com/openstack/openstack-helm.git

다음 스크립트는 모두 컴퓨터의 openstack-helm 디렉토리에서 실행한다. . 이러한 각 스크립트의 내용은 문서에 설명되어 있지만 주로 Helm 차트를 사용하여 필요한 서비스를 설치한다.

첫 번째는 하나의 노드 kubernetes 클러스터를 시작하고 실행하고 Helm을 배포하여 Helm 차트를 사용하여 일부 이후 단계에서 Openstack 서비스를 배포 할 수 있도록하는 스크립트이다.

.. code::
 
    # cd openstack-helm
    #./tools/deployment/developer/common/010-deploy-k8s.sh

Kubernetes 및 Helm이 성공적으로 배포되면 다음 스크립트는 python-pip를 사용하여 Openstack 및 Heat 클라이언트를 모두 설치한다.

이러한 클라이언트는 Openstack과 함께 명령 줄 인터페이스를 제공한다.

이 스크립트는 또한 모든 필수 Openstack 서비스에 대한 모든 Helm 차트를 빌드한다.

.. code::
 
    # ./tools/deployment/developer/common/020-setup-client.sh
Ingress 를 배포하는 스크립트를 실행한다.

.. code::
 
    # ./tools/deployment/developer/common/030-ingress.sh
Openstack 클라우드의 공유 스토리지의 경우 Ceph 스토리지 또는 NFS Provisioner를 사용할 수 있다. 간단하게 배포하기 위하여 NFS를 사용 하기 위하여 NFS 를 배포하기 위한 스크립트를 실행 한다.

.. code::
 
    # ./tools/deployment/developer/nfs/040-nfs-provisioner.sh
A database is required by each of the Openstack services. MariaDB is the database that is deployed in this solution

각 Openstack 서비스에 데이터베이스로 사용 할 MariaDB를 배포 한다.

.. code::
 
    # ./tools/deployment/developer/nfs/050-mariadb.sh
RabbitMQ 를 배포 한다.

.. code::
 
    # ./tools/deployment/developer/nfs/060-rabbitmq.sh
Memcached를 배포 한다.

.. code::
 
    # ./tools/deployment/developer/nfs/070-memcached.sh
다음 스크립트를 실행하여 Keystone을 배포한다. Keystone은 Openstack과 함께 제공되는 잘 알려진 ID 서비스로 사용자 및 클라우드의 다른 서비스를 인증하는 데 사용된다


.. code::
 
    # ./tools/deployment/developer/nfs/080-keystone.sh
Horizon으로도 알려진 Openstack Dashboard는 다음 스크립트를 사용하여 배포한다.

.. code::
 
    # ./tools/deployment/developer/nfs/100-horizon.sh
Glance 배포를 배포한다.

.. code::
 
    # ./tools/deployment/developer/nfs/120-glance.sh
OpenvSwitch는 모든 Openstack 배포에서 거의 표준이 된 가상 스위치로 이를 통해 사용자는 하나의 클라우드 내에서 많은 수의 가상 네트워크를 만들 수 있다.

OpenvSwitch를 아래 스크립트로 배포한다.

.. code::
 
    # ./tools/deployment/developer/nfs/140-openvswitch.sh
Glance 서비스가 가동되고 실행되면 Libvirt를 배포 할 차례이다. Libvirt는 QEMU와 인터페이스하는 데 사용된다.

.. code::
 
    # ./tools/deployment/developer/nfs/150-libvirt.sh
compute-kit 스크립트는 Nova 서비스와 Neutron 서비스를 모두 포함 한다. Nova 서비스는 Openstack의 컴퓨팅 서비스이고 Neutron은 라우터와 같은 네트워크 개체 생성을 관리하는 네트워킹 서비스이다.

.. code::
 
    # ./tools/deployment/developer/nfs/160-compute-kit.sh
공용 네트워크에 대한 게이트웨이 설정

.. code::
 
    # ./tools/deployment/developer/nfs/170-setup-gateway.sh
이 모든 과정을 마치면 이제 Openstack 대시 보드를 https : //horizon.openstack.svc.cluster.local에서 사용할 수 있다