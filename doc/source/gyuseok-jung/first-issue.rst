==========================================================================
[이슈] server resize --flavor --wait waits for wrong success statuses 해결
==========================================================================

이슈 소개
----------

.. code:: bash

    $ openstack server resize --flavor m1.small --wait vm1

명령어 실행이 실패 할 경우 실패 메세지가 떠야할꺼 같지만 실제로는 Complete 메세지가 나오며
실제 flavor를 확인하는 명령어 실행 시

.. code:: bash

    $ openstack server show vm1 -f value -c status -c flavor
    m1.tiny (1)
    ACTIVE

처럼 이전 상황으로 돌아가는 현상이 있다.

재현 방법
----------

서버의 사양을 뛰어넘는 크기로 flavor를 바꿔본다
제공된 cafe24 서버의 사양은 CPU 4코어 / RAM 8GB / SSD 30GB이다 이때 
CPU 8코어 / RAM 16GB / 디스크 160GB 가 필요한 m1.xlarge flavor 로 변경을 진행해 본다.

.. code:: bash

    $ openstack server resize --flavor m1.xlarge --wait vm1
    Complete

에러가 발생해야할것으로 예상 되지만 Complete 메세지가 뜬것을 확인할 수 있다.

확인을 위해 현재 flavor를 확인해보면

.. code:: bash

    $ openstack server show vm1 -f value -c status -c flavor
    m1.tiny (1)
    ACTIVE

와 같이 변경 전 상태임을 확인 할 수 있다.


문제가 발생한 코드 부분
------------------------

.. code:: python

    if parsed_args.wait:
        if utils.wait_for_status(
            compute_client.servers.get,
            server.id,
            success_status=['active', 'verify_resize'],
            callback=_show_progress,
        ):
            self.app.stdout.write(_('Complete\n'))
        else:
            LOG.error(_('Error resizing server: %s'),
                server.id)
            self.app.stdout.write(_('Error resizing server\n'))
            raise SystemExit

접근 방법
---------

.. code:: python

    utils.wait_for_status(
        compute_client.servers.get,
        server.id,
        success_status=['active', 'verify_resize'],
        callback=_show_progress,
    ):
    
확인 결과 flavor가 변경되지 않은 상태에서도 success_status가 active로 나와 위 코드의 리턴값이 true로 나온다.
이때 정상적으로 flavor가 변경이 되었다면 success_status가 verify_resize로 나오게 된다. 
만약 verify_resize가 아니라 active인 상태에서 flavor가 변경되었는지 아닌지를 확인해 보면 해결이 가능할 것 같다.

``server`` 객체에 지금 서버의 flavor id 가 있는것을 확인하였고
``flavor`` 객체에 지금 바꾸려고 하는 flavor id 가 있는것을 확인하였다.
따라서 서버의 ``verify_resize`` 상태만 확인이 가능하면 

.. code:: python

    if flavor.id != server_flavor_id:
        self.app.stdout.write(_('Error resizing server\n'))

위 코드로 해결이 가능하다.
