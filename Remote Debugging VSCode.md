# Remote Debugging with VSCode for OpenStackClient

VSCode에서 openstackclient 원격 디버깅 설정하기

## 로컬 설정

1. VSCode에서 Remote Development 부가기능을 설치한다.

   <img width="1099" alt="원격 서버 부가기능" src="https://user-images.githubusercontent.com/9061758/89763897-66544a00-db2e-11ea-9b08-b56001b8a63e.png">

2. VSCode 좌측 하단에 있는 초록색 버튼을 클릭한 후 Remote-SSH: Connet to Host...를 클릭한다.

   <img width="217" alt="원격 연결 버튼" src="https://user-images.githubusercontent.com/9061758/89763899-66ece080-db2e-11ea-8d9d-1d70fbe93874.png">

   <img width="856" alt="서버연결 선택지" src="https://user-images.githubusercontent.com/9061758/89763892-65231d00-db2e-11ea-8e00-1267f0d2d3d3.png">

3. 이후 ssh 연결 명령어를 통해 서버에 접속한다.

   <img width="734" alt="서버 ip 입력" src="https://user-images.githubusercontent.com/9061758/89763890-648a8680-db2e-11ea-96ec-b9f0dd9900f2.png">

   이때 서버는 <u>root 계정</u>으로 접속해야한다.

   `ssh -i <pem 파일 경로> root@서버주소`

## 원격지 설정

1. 정상적으로 원격지에 접속이 되었다면 서버에 Python 부가기능을 설치해야한다

   <img width="801" alt="원격지 파이썬 부가기능 설치" src="https://user-images.githubusercontent.com/9061758/89764877-373ed800-db30-11ea-9812-64441373691c.png">

2. 원격지에서 작업을 진행할 OpenStackClient 경로를 선택해 열어준다.

   <img width="1015" alt="파일 오픈" src="https://user-images.githubusercontent.com/9061758/89763903-681e0d80-db2e-11ea-8e44-8f2420b5da10.png"> 

3. vscode에서 디버깅을 하기위해서는 디버깅 설정파일인 launch.json 파일을 수정해야한다.

   - 해당 파일은 .vscode 폴더안에 있으며 그림처럼 create a lauch.json file 버튼을 클릭후 launch.json 파일을 만드는 방법과
   <img width="218" alt="디버깅" src="https://user-images.githubusercontent.com/9061758/89765206-d532a280-db30-11ea-8ef4-dddf0b407c71.png">
   
   - 직접 .vscode 폴더 생성 후 launch.json 파일을 생성하는 방법이 있다.
   <img width="171" alt="vscode" src="https://user-images.githubusercontent.com/9061758/89768143-30b35f00-db36-11ea-9d51-b98ba11ebc4e.png">


4. launch.json파일의 설정 값을 다음 이미지와 같이 바꿔준다

   <img width="885" alt="설정값" src="https://user-images.githubusercontent.com/9061758/89763894-65bbb380-db2e-11ea-980d-4f953592d3c9.png">

   

   `.vscode/launch.json`

   ```json
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
   ```

5. 이후 브레이크 포인트를 찍고 f5를 눌러 디버깅을 실행해 보면 정상적으로 디버깅이 되는것을 확인할 수 있다.

    <img width="1554" alt="디버깅 완료" src="https://user-images.githubusercontent.com/9061758/89763869-5f2d3c00-db2e-11ea-8829-21b1242d4336.png">

   

* root 사용자로 ssh 로그인 진행하기

  ``` shell
  $ sudo su
  root@:~# vim ~/.ssh/authorized_keys
  
  ssh-rsa 앞 까지 주석을 지워주면 됨
  ```



## 참고자료

https://joonghyunlee.github.io/remote-debugging-cinder-1

https://www.notion.so/Remote-Debugging-with-PyCharm-for-OpenStack-ceb3c9a708a5437d84df8ba6edd8143c

