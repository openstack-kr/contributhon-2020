================================================
VSCode에서 openstackclient 원격 디버깅 설정하기
================================================

로컬 설정
----------

1. VSCode에서 Remote Development 부가기능을 설치한다.

   .. image:: images/install-remote-development.png


2. VSCode 좌측 하단에 있는 초록색 버튼을 클릭한 후 Remote-SSH: Connet to Host...를 클릭한다.

   .. image:: images/remote-select-button.png

   .. image:: images/server-connect-list.png

3. 이후 ssh 연결 명령어를 통해 서버에 접속한다.

   .. image:: images/server-ip.png

   이때 서버는 **root 계정** 으로 접속해야한다.

   .. code:: 

      ssh -i <pem 파일 경로> root@서버주소

원격지 설정
------------

1. 정상적으로 원격지에 접속이 되었다면 서버에 Python 부가기능을 설치해야한다

   .. image:: images/remote-python-plugin.png

2. 원격지에서 작업을 진행할 OpenStackClient 경로를 선택해 열어준다.

   .. image:: images/fileopen.png

3. vscode에서 디버깅을 하기위해서는 디버깅 설정파일인 .vscode/launch.json 파일을 생성해야한다.

   -  .vscode/launch.json파일을 생성하는 방법에는 그림처럼 create a lauch.json file 버튼을 클릭후 launch.json 파일을 만드는 방법과

      .. image:: images/debugging.png
   
   - 직접 .vscode 폴더 생성 후 launch.json 파일을 생성하는 방법이 있다.
      
      .. image:: images/vscode.png

      


4. launch.json파일의 설정 값을 다음 이미지와 같이 바꿔준다

   .. image:: images/setting.png
   

   `.vscode/launch.json`

   .. code::
   
      {
       "version": "0.2.0",
       "configurations": [
           {
               "name": "Python: Current File",
               "type": "python",
               "request": "launch",
               "program": "/usr/local/bin/openstack",
               "args": [
                   "server list"
               ],
               "console": "integratedTerminal",
               "env": {
                   "OS_PROJECT_NAME": "demo",
                   "OS_TENANT_NAME": "demo",
                   "OS_USERNAME": "demo",
                   "OS_PASSWORD": "secret",
                   "OS_REGION_NAME": "RegionOne",
                   "OS_IDENTITY_API_VERSION": "3",
                   "OS_AUTH_TYPE": "password",
                   "OS_AUTH_URL": "http://127.0.0.1/identity",
                   "OS_VOLUME_API_VERSION": "3",
                   "OS_USER_DOMAIN_ID": "default",
                   "OS_PROJECT_DOMAIN_ID": "default",
                   "CINDER_VERSION": "3"
               },
               "justMyCode": false,
               "gevent": true,
               "redirectOutput": true
           }
         ]
      }
   
      {
         "python.pythonPath": "/usr/bin/python3"
      }
   

5. 이후 브레이크 포인트를 찍고 f5를 눌러 디버깅을 실행해 보면 정상적으로 디버깅이 되는것을 확인할 수 있다.

   .. image:: images/check-debug.png
   

   



참고자료
=========

https://joonghyunlee.github.io/remote-debugging-cinder-1

https://www.notion.so/Remote-Debugging-with-PyCharm-for-OpenStack-ceb3c9a708a5437d84df8ba6edd8143c


추가
=====

* root 사용자로 ssh 로그인 진행하기

  .. code::

   $ sudo su
   root@:~# vim ~/.ssh/authorized_keys
  
  ssh-rsa 앞 까지 주석을 지워주면 됨
  