============
PDB 사용하기
============

PDB란?
------

pdb는 The Python Debugger로 Python 프로그램을 위한 대화형 소스 코드
디버거입니다.

소스 라인 레벨에서 breakpoint 을 설정하여 디버깅할 수 있습니다.

PDB 사용방법
------------

1. Lib를 import 합니다.

.. code:: python

   import pdb

2. Breakpoint 설정할 코드 라인에 다음 명령어를 입력합니다.

.. code:: python

   pdb.set_trace()

3. Pdb를 실행합니다.

.. code:: shell

   $ python3 -m pdb 파일-이름.py

**Ex**

.. code:: shell

   stack@ubuntu:~/devstack$ python3 -m pdb /usr/local/lib/python3.6/dist-packages/openstackclient/compute/v2/server.py
   > /usr/local/lib/python3.6/dist-packages/openstackclient/compute/v2/server.py(16)<module>()
   -> """Compute v2 Server action implementations"""
   (Pdb) list
    11     #   WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
    12     #   License for the specific language governing permissions and limitations
    13     #   under the License.
    14     #
    15
    16  -> """Compute v2 Server action implementations"""
    17
    18     import argparse
    19     import getpass
    20     import io
    21     import logging

\*\* 저는 ``openstack server list`` 명령어 입력 후 ``Class ListServer``
에서의 Code Flow를 보기 위해 해당 Class 안에 breakpoint를 설정하였고,
``openstack server list`` 를 실행하였습니다. 이렇게 되면 첫번째
breakpoint인 ``pdb.set_trace()`` 라인에 멈춥니다.

4. (다음) breakpoint로 이동합니다.

.. code:: shell

   (Pdb) continue

``continue`` 는 다음 breakpoint로 이동합니다. 단축어로 ``c``\ 를
입력해도 됩니다.

**Ex**

.. code:: shell

   (Pdb) continue
   > /usr/local/lib/python3.6/dist-packages/openstackclient/compute/v2/server.py(1142)ListServer()
   -> _description = _("List servers")
   (Pdb) list
   1137                        raise SystemExit
   1138
   1139
   1140    class ListServer(command.Lister):
   1141        pdb.set_trace()
   1142 ->     _description = _("List servers")
   1143
   1144        def get_parser(self, prog_name):
   1145            parser = super(ListServer, self).get_parser(prog_name)
   1146            parser.add_argument(
   1147                '--reservation-id',

server.py의 16 Line에서 breakpoint 다음 Line인 1142 Line으로
이동했습니다.

PDB 명령어
----------

해당 문서에는 간단히 사용할 수 있는 몇개의 명령어만 있습니다. 저 자세한
설명은 `Doc <https://docs.python.org/3/library/pdb.html>`__ 를
참고해주세요.

1. line ( l )
~~~~~~~~~~~~~

.. code:: shell

   (Pdb) list

주변 소스코드를 출력하고 현재 라인이 화살표로 표시됩니다.

2. next ( n )
~~~~~~~~~~~~~

.. code:: shell

   (Pdb) next

현재 문장을 실행하고 다음 문장으로 이동합니다.

3. step ( s )
~~~~~~~~~~~~~

.. code:: sh

   (Pdb) step

함수 내부로 이동합니다.

4. return ( r )
~~~~~~~~~~~~~~~

.. code:: shell

   (Pdb) return

현재 함수의 return이 나올 때까지 실행합니다.

5. break ( b )
~~~~~~~~~~~~~~

.. code:: shell

   (Pdb) break line_number

해당 line에 breakpoint를 만들 수 있습니다.

6. continue ( c )
~~~~~~~~~~~~~~~~~

.. code:: shell

   (Pdb) continue

다음 breakpoint까지 실행하며 다음 breakpoint가 없다면 끝까지 실행합니다.

7. args ( a )
~~~~~~~~~~~~~

.. code:: shell

   (Pdb) args

현재 함수의 매개변수들을 출력합니다.

8. quit ( q )
~~~~~~~~~~~~~

.. code:: shell

   (Pdb) quit

프로그램 실행을 중단하고 디버거를 종료합니다.
