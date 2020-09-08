------------------------------------------------------
[학습] openstack 테스트코드를 Tox 사용해 테스트하기
------------------------------------------------------

tox 는 파이썬의 자동화 테스팅 도구입니다.


이 글은 ubntu 18.04에서 환경에서 진행하였습니다.


.. code-block:: shell

    pip install tox

먼저 tox를 설치해줍니다.


.. code-block:: shell

    git clone https://opendev.org/openstack/python-openstackclient
    cd python-openstackclient

테스트를 진행할 openstack 프로젝트를 clone 한 뒤에 폴더로 이동해줍니다.


.. code-block:: shell

    ubuntu@devstack-master:~/python-openstackclient$ cat tox.ini
    [tox]
    minversion = 3.2.0
    envlist = py37,pep8
    skipdist = True
    # Automatic envs (pyXX) will only use the python version appropriate to that
    # env and ignore basepython inherited from [testenv] if we set
    # ignore_basepython_conflict.
    ignore_basepython_conflict = True

tox.ini 파일을 보면 tox 설정들을 볼 수 있습니다. 





전체 테스트
-------------

.. code-block:: shell

    tox

tox 를 입력하면 전체 테스트가 돌아갑니다.


.. code-block:: shell

    tox -e py

tox 만으로 동작하지 않는다면, 위와 같이 py 환경변수를 같이 입력해주세요.


.. code-block:: shell

    tox -e py37

이런식으로 테스팅 파이썬 환경을 직접 지정해줄 수도 있다고 하는데, python3.7 을 따로 설치 해보았지만 저는 제대로 동작하지 않았습니다. 혹시 테스팅 되는 분은 알려주시면 감사하겠습니다. (py만 주었을 때 가장 잘 작동하는 것으로 보아 확실하지는 않지만 자동으로 환경을 맞추어 실행하는게 아닐까 추측해봅니다.) 



단위 테스트 예시
--------------------

One common activity is to just run a single test, you can do this with tox simply by specifying to just run py37 tests against a single test:

.. code-block:: shell

    tox -e py37 -- cinder.tests.unit.volume.test_availability_zone.AvailabilityZoneTestCase.test_list_availability_zones_cached


Or all tests in the test_volume.py file:

.. code-block:: shell

    tox -e py37 -- cinder.tests.unit.volume.test_volume


You may also use regular expressions to run any matching tests:

.. code-block:: shell

    tox -e py37 -- test_volume


출처: docs.openstack.org/cinder/latest/contributor/testing.html , docs.openstack.org/kolla/latest/contributor/running-tests.html



대략 위와 같은 느낌으로 tox -e 파이썬 환경(py,36 py37..)을 입력해주고, 뒤에는 테스트할 파일이나 클래스, 함수 등의 경로를 입력해주면 됩니다. 





단위 테스트 실습
-----------------------

clone한 pytonh-openstackclinet 에서 테스트를 직접 단위 테스트를 수행해보도록 하겠습니다.



python-openstackclient/openstackclient/tests 위치에서 tree 명령어로 구조를 보면 다음과 같습니다.

.. code-block:: shell

    stack@server1:~/tmp/python-openstackclient/openstackclient/tests$ pwd
    /opt/stack/tmp/python-openstackclient/openstackclient/tests

    stack@server1:~/tmp/python-openstackclient/openstackclient/tests$ tree
    .
    ├── __init__.py
    ├── __pycache__
    │   └── __init__.cpython-36.pyc
    ├── functional
    │   ├── __init__.py
    │   ├── base.py
    │   ├── common
    │   │   ├── __init__.py
    │   │   ├── test_args.py
    │   │   ├── test_availability_zone.py
    │   │   ├── test_configuration.py
    │   │   ├── test_extension.py
    │   │   ├── test_help.py
    │   │   ├── test_module.py
    │   │   ├── test_quota.py
    │   │   └── test_versions.py
    │   ├── compute
    │   │   ├── __init__.py
    │   │   └── v2
    │   │       ├── __init__.py
    │   │       ├── common.py
    │   │       ├── test_agent.py
    │   │       ├── test_aggregate.py

만약 우리가 tests 폴더 전체를 테스트하고 싶다고 가정해보겠습니다.

.. code-block:: shell

    stack@server1:~/tmp/python-openstackclient$ tox -e py openstackclient.tests
    stack@server1:~/tmp/python-openstackclient$ tox -e py tests

이와 같이 실행시켜 주면 됩니다.

tox를 실행하는 위치는 같은 프로젝트 내부라면 어디든 상관없습니다.



.. code-block:: shell

    stack@server1:~/tmp/python-openstackclient$ tox -e py unit.compute.v2.test_server
    만약 tests/unit/test_shell.py 파일을 실행시키고자 한다면 위와 같이 작성해주시면 됩니다.

.py 를 제외하고 .을 사용해 경로를 지정해주어야합니다.





.. code-block:: shell

    stack@server1:~/tmp/python-openstackclient$ tox -e py unit.compute.v2.test_server

TestServerCreate 클래스만을 테스트 하기 위해서는 위와 같이 작성해주시면 됩니다.




.. code-block:: shell

    stack@server1:~/tmp/python-openstackclient$ tox -e py unit.compute.v2.test_server

TestServerCreate 클래스의 test_server_create_no_options 메소드를 실행키기 위해서는 위와 같이 작성해주시면 됩니다.
