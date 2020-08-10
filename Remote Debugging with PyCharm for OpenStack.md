# Remote Debugging with PyCharm : for OpenStack

PyCharm professional에서 원격 디버깅 설정하기

(참조 : [https://www.jetbrains.com/help/pycharm/remote-debugging-with-product.html](https://www.jetbrains.com/help/pycharm/remote-debugging-with-product.html))

1. ~~Download openstackclient~~

    ~~github python-openstackclient 저장소에서 openstackclient 소스코드를 다운.~~

    ~~github repository url: [https://github.com/openstack/python-openstackclient](https://github.com/openstack/python-openstackclient)~~

    ```bash
    [somedirpath]$ git clone [https://github.com/openstack/python-openstackclient.git](https://github.com/openstack/python-openstackclient.git)
    ```

여기서부터 시작함.

1. PyCharm 원격 interpreter 설정

    ![https://user-images.githubusercontent.com/67255441/89796952-44c28500-db65-11ea-96fd-7e7ae08125fd.png](https://user-images.githubusercontent.com/67255441/89796952-44c28500-db65-11ea-96fd-7e7ae08125fd.png)

    프로젝트 이름을 정함.

    원격에 존재하는 python3 인터프리터를 사용할 것이니 Existing interpreter를 선택!

    오른쪽에 ... 클릭하면

    ![https://user-images.githubusercontent.com/67255441/89797051-64f24400-db65-11ea-81ca-9b05030881cf.png](https://user-images.githubusercontent.com/67255441/89797051-64f24400-db65-11ea-81ca-9b05030881cf.png)

    SSH Interpreter 설정을 해줄것임.

    Host : cafe24 server host

    Username : root (*root를 사용해 인증하려면 리모트 서버의 .ssh/authorized_keys 파일을 수정해야함, 뒤에 나옴)

    Key pair 선택

    ![https://user-images.githubusercontent.com/67255441/89797132-83f0d600-db65-11ea-883b-5ba1d41e99fc.png](https://user-images.githubusercontent.com/67255441/89797132-83f0d600-db65-11ea-883b-5ba1d41e99fc.png)

    cafe24에서 받은 SSH Key pair 선택

    디폴트로 python2가 설정되어 있지만 python3으로 변경(3만 붙여주면 됨!)

    ![https://user-images.githubusercontent.com/67255441/89797185-9539e280-db65-11ea-9be1-f5883c08070d.png](https://user-images.githubusercontent.com/67255441/89797185-9539e280-db65-11ea-9be1-f5883c08070d.png)

    Remote project location은 openstackclient가 설치된 path로 설정

    ![https://user-images.githubusercontent.com/67255441/89797214-9ff47780-db65-11ea-8835-5d81f10211ad.png](https://user-images.githubusercontent.com/67255441/89797214-9ff47780-db65-11ea-8835-5d81f10211ad.png)

    Remote project location: /usr/local/lib/python3.6/dist-packages/openstackclient

    * root로 SSH Interpreter를 인증할 경우

    root사용자로 인증 진행 하려면 remote 서버(183.x.x.x)에서 /.ssh/authorized_keys를 수정해야 함

    ```bash
    $ sudo su
    root@:~# vim ~/.ssh/authorized_keys

    ssh-rsa 앞 까지 주석을 지워주면 됨
    ---
    ssh-rsa AAAA 
    ...
    +n Generated-by-Nova
    ---
    ```

    - Path mappings: 
    마지막으로 로컬 openstackclient [shell.py](http://shell.py) 위치와 원격 openstackclient shell.py를 연결
    [some_local_dirpath]/openstackclient=[some_remote_dirpath]/openstackclient

        ![https://user-images.githubusercontent.com/67255441/89797266-af73c080-db65-11ea-8455-31e8af2fefda.png](https://user-images.githubusercontent.com/67255441/89797266-af73c080-db65-11ea-8455-31e8af2fefda.png)

2. 실행/디버그 환경설정

    Add configurations을 클릭

    ![https://user-images.githubusercontent.com/67255441/89797300-bac6ec00-db65-11ea-9d94-38adc9cd4b5d.png](https://user-images.githubusercontent.com/67255441/89797300-bac6ec00-db65-11ea-9d94-38adc9cd4b5d.png)

    Python 환경 추가

    openstack server list를 실행하는 환경을 구성할 것임.

    ![https://user-images.githubusercontent.com/67255441/89797333-c4e8ea80-db65-11ea-8ae0-cc03cd4464bf.png](https://user-images.githubusercontent.com/67255441/89797333-c4e8ea80-db65-11ea-8ae0-cc03cd4464bf.png)

    - Script path: remote 서버의 openstack 명령어를 실행해주는 실행파일 위치
    remote server에서 openstack 명령어 실행파일 위치를 조회.

        ```jsx
        $ which openstack
        /usr/local/bin/openstack
        ```

    - Parameters: openstack으로 실행할 명령어 인자
    help, server list...
    [https://docs.openstack.org/python-openstackclient/pike/cli/command-list.html](https://docs.openstack.org/python-openstackclient/pike/cli/command-list.html) 참조
    - Environment variables: 

    OS_AUTH_URL은 http://127.0.0.1/identity로 채우면 됨.

    ![https://user-images.githubusercontent.com/67255441/89798417-1e9de480-db67-11ea-9c9e-86009ee22991.png](https://user-images.githubusercontent.com/67255441/89798417-1e9de480-db67-11ea-9c9e-86009ee22991.png)

    ![https://user-images.githubusercontent.com/67255441/89798434-252c5c00-db67-11ea-8497-9d0a5f33ed6e.png](https://user-images.githubusercontent.com/67255441/89798434-252c5c00-db67-11ea-8497-9d0a5f33ed6e.png)

    - Python interpreters: 
    step 1에서 생성한 원격 interpreter로 자동 설정되어 있음

        안되어 있다면 step 1을 진행하면 됨

    - Path mappings: 
    마지막으로 로컬 openstackclient [shell.py](http://shell.py) 위치와 원격 openstackclient shell.py를 연결
    [some_local_dirpath]/openstackclient=[some_remote_dirpath]/openstackclient

3. run 실행하면 openstack server list가 실행되고 결과가 출력

![https://user-images.githubusercontent.com/67255441/89797407-e053f580-db65-11ea-8422-b30e711835d8.png](https://user-images.githubusercontent.com/67255441/89797407-e053f580-db65-11ea-8422-b30e711835d8.png)