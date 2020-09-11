===================================================
openstack keystone token system all scope 400 error
===================================================
오픈스택에서 api를 요청 할 경우 여러가지 방법이 있음,  
그 중에서도 python requests 모듈을 통해 토큰을 발급하고 api를 요청하는 과정에서 이상한 부분을 발견하여 문서로 작성

오픈스택 keystone에서 토큰을 활용할 수 있는 방안은 다음과 같다 `keystone api  <https://docs.openstack.org/api-ref/identity/v3/?expanded=password-authentication-with-scoped-authorization-detail>`_ 

  .. code-block:: 

    {
        "auth": {
            "identity": {
                "methods": [
                    "password"
                ],
                "password": {
                    "user": {
                        "id": "ee4dfb6e5540447cb3741905149d9b6e",
                        "password": "devstacker"
                    }
                }
            },
            "scope": {
                "system": {
                    "all": true
                }
            }
        }
    }

위 json은 openstack에서 keystone을 통해서 token 발행을 할 경우 사용되는 json body정보임 
위 방식으로 api를 요청 했을 경우 nova, neutron, cinder, keystone 등 api 요청을 했을 때 정상적으로 결과물을 받아 올 수 있는걸 확인함

   .. code :: 

       http://controller:8776/v3/7699fd0d3e5b44fe8871af7bad08df21/volumes/detail 
       {'X-Auth-Token': 'gAAAAABfLaJUuts0vAH_UsYEDT8QFN2X0jR3yL75R__UKNOZqo0jUNQHPSh1GduYfgl_6KxbuME-3pPSIj4h9k76wgh- 
       Old1dBl81vpPLOc-9GdRY5E6xUBSXQM3a4IscdkmEzpSkys9bitQQDo3yTUPSdndkDEzGg'}

       {'badRequest': {'message': 'Malformed request url', 'code': 400}}

그러나 cinder의 경우에는 api요청을 했을 때 400에러가 나오는것을 확인 하였음 이러한 400 error는 cinder, trove, swift, maila 등의 서비스에서 위와 같은 똑같은 상황이 나타나는것을 확인 함

    .. code::
   
        curl -g -i -X GET http://controller:8776/v3/7699fd0d3e5b44fe8871af7bad08df21/volumes/detail -H "Accept: 
        application/json" -H "OpenStack-API-Version: volume 3.59" -H "User-Agent: python-cinderclient" -H "X-Auth- 
        Token: {SHA256}1dc35186f7e05f9a211214a9da71e4408c254c1bc2a1b745c085bdc2091b482a"

위 요청은 openstack volume list --debug 명령어를 통해 디버깅을 했을 때 cinder가 api 요청을 어떻게 수행하는지 확인 하였고 정상적인 결과물이 출력되는 것을 확인 할 수 있음

또한 token에 대한 권한 확인을 하기 위해서 openstack token issue --debug를 통해 일반적은 token은 어떻게 발급하는지 확인 함 

    .. code-block:: 

        Using parameters {'username': 'admin', 'project_name': 'admin', 'user_domain_name': 'Default', 'auth_url': 
       'http://controller:5000/v3', 'password': '***', 'project_domain_name': 'Default'}

위와 같은 방식으로 token을 발급하는 것 확인 하엿고 이러한 디버깅을 통해서 테스트 코드를 작성


    .. code-block:: python

        import requests
        def test_create_credentials_token(key, mode) : #토큰 생성
            print("Create Credentials Token")
            token = ""
            if mode == 1 :
                data = '{"auth":{"identity":{"methods":["password"],"password":{"user":                                                    
                {"id":"'+key['OS_ADMIN_ID']+'","password":"'+key['OS_PASSWORD']+'"}}},"scope":{"project": 
                {"id":"'+key['OS_PROJECT']+'"}}}}'
            elif mode == 2 :
                data = '{"auth":{"identity":{"methods":["password"],"password":{"user": 
               {"id":"'+key['OS_ADMIN_ID']+'","password":"'+key['OS_PASSWORD']+'"}}},"scope":{"system":{"all":true}}}}'

            res = requests.post(key['OS_AUTH_URL']+'identity/v3/auth/tokens', data=data)
            return res.headers['X-Subject-token']

        def set_api(OS_TOKEN, URL) : # 서비스로 api 전송
            headers = {
                'X-Auth-Token': OS_TOKEN,
            }
            response = requests.get(URL, headers=headers)

            return response

            if __name__ == "__main__":
                key = {
                    "OS_ADMIN_ID" : "b76cd03ce71446e79fa47b707397e9a2", # admin id == openstack user list
                    "OS_PASSWORD" : "ADMIN_PASS",# 설치 시 입력 했던 passwd
                    "OS_PROJECT" : "1fb596f7cb454ec9a7d6533af8ce1826", # demo project id == openstack project list / admin project로 해도 상관 없음
                    "OS_AUTH_URL" : "http://192.168.1.8/", # 설치 시 입력했던 ip ex) 192.168.1.8/
                }
                token = test_create_credentials_token(key,1)
                print("scope : project : ", token)
                result = set_api(token, key['OS_AUTH_URL']+"volume/v3/"+key['OS_PROJECT']+"/volumes/detail")
                print(result.json())

                token = test_create_credentials_token(key, 2)
                print("2 : ", token)
                result = set_api(token, key['OS_AUTH_URL']+"volume/v3/"+key['OS_PROJECT']+"/volumes/detail")
                print(result.json())

테스트 코드 작성


    .. code::
  
        Create Credentials Token
        scope : project :  gAAAAABfMlP3R8cKFu6PynJyNatvlHdKBI0EwH7OYpqIQ_Mm4pPUu5GRGZTwGrVeoG2yzU- 
        5QlJB6aluIsEUAhQJ_5G7S1Jx1hh8V3CefFvo0oTbpi8NToh3LdgMaHEuThWOoPKkVFvJkJolVXEPjvSylcKIcfJimBdwai_cUX9e0w4c4encyI8
        Create Credentials Token
        2 : gAAAAABfMlP4EkqICPEl5JhGF8qZ9WkBuCBo0ht8XB9NTBvTPNFmD6gU3rolJEVecg5byzi_3PtorWhYuPmBJ9W8QwPpV9ZQP769zeDBRdS3B9SWlLTNgTU7PRbhIFF4RDh32Rcvb3BL65pSif6-cSq7PpAxE2rbdw
        {'badRequest': {'code': 400, 'message': 'Malformed request url'}}

위와 같이 1번째 project를 넣은 경우엔느 정상적으로 잘 생행 되었으며 2번째 경우는 400 error가 마찬가지로 떨어지는것을 확인 함 

원인 분석

원인은 `cinder git <https://github.com/openstack/cinder/blob/master/cinder/api/openstack/wsgi.py#L888>`_ 
에서 확인 할 수 있음

    .. code-block:: python

        project_id = action_args.pop("project_id", None)
        context = request.environ.get('cinder.context')
        if (context and project_id and (project_id != context.project_id)):
            msg = _("Malformed request url")
            return Fault(webob.exc.HTTPBadRequest(explanation=msg))

여기서 keystone / neutron / nova 등은 api Endpoint URI에 project id가 포함된게 없어서 그 if 문에서 검사하는 조건들이 모두 None임
그러나 cinder 는 api   Endpoint uri 에 project id가 있기 때문에 context.project_id = None 인데.
argument로 넘어온 project_id는 uri에 있는 project id 라서 if문에서 false가 됨
그럼 왜 context.project_id가 None인가?
-> 토큰의 scope가 프로젝트 단위가 아니라 system이니까.. 당연히 지금 토큰으로 처리하는 요청은 project id가 없으므로 400에러 발생

이러한 이슈는 2018년 버그 리포팅에도 올라왔음 `버그 <https://bugs.launchpad.net/cinder/+bug/1745905>`_ 

그에 따라서 문서를 추적해봤을 때 

    .. code :: 

        The authorization scope, including the system (Since v3.10), a project, or a domain (Since v3.4). If multiple scopes are specified in the same request (e.g. project and domain or domain and system) an HTTP 400 Bad Request will be returned, as a token cannot be simultaneously scoped to multiple authorization targets. An ID is sufficient to uniquely identify a project but if a project is specified by name, then the domain of the project must also be specified in order to uniquely identify the project by name. A domain scope may be specified by either the domain’s ID or name with equivalent results.

위와 같은 방법을 사용 시 400 에러가 발생한다고 나와있음 추가적으로 이러한 이슈를 해결해보기 위해서 메일링 리스트를 작성

    .. code ::

        Message: 1
        Date: Sun, 23 Aug 2020 21:18:32 +0900
        From: Mingi Jo <jomin0613@gmail.com>
        To: openstack-discuss@lists.openstack.org
        Subject: [keystone] openstack token auth scpore system Question
        Hi, I'm studying OpenStack.If you use OpenStack and use it with a
        keystone token on all computers,If there is a project in the endpoint
        URL, the api request cannot be made properly.The error message is
        output at 400, and the request fails. We've looked into this, and I've
        found out,https://bugs.launchpad.net/cinder/+bug/1745905Here's the bug
        reporting, and I think it's done with the paperwork.However, various
        services such as cinder, swift, and probe are required to include
        projects in the endpoint url of the installation guide, which is
        considered contradictory.Is there any way to fix this?
        -------------- next part --------------
        An HTML attachment was scrubbed...
        URL: <http://lists.openstack.org/pipermail/openstack-discuss/attachments/20200823/5f1612ec/attachment-0001.html>

이메일을 작성하였고 명확한 답변은 아직 안온상태

2020-09-11 추가
http://eavesdrop.openstack.org/meetings/keystone/2020/keystone.2020-09-08-16.58.log.html

keystone 미팅에 참석하여 어떻게 문제사항을 수정해야할지 힌트를 얻어 추가적인 코드리뷰가 가능할 것으로 보임




