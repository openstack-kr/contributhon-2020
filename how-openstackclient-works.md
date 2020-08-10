이번주는 오픈스택을 설치하고 openstackclient 코드를 분석했습니다.
코드와 함께 보려면 다음 링크를 참고하세요. [코드와 함께 편하게 보기](https://medium.com/@wlckd90/openstack-%EB%AA%85%EB%A0%B9%EC%96%B4-%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0-a6ab2e987500)

openstack server list 명령어를 입력했을 때 어떠한 순서로 코드가 진행 되는지 알아보겠습니다. 어떤 코드를 실행하면서 결과를 출력하는지, | ID | Name | Status | Networks | Image | Flavor | 외에 다른 항목들은 어떻게 추가하는지에 대해 이야기해보겠습니다.

명령어 실행과 출력 예시
위와 같이 openstack server list를 입력하면 서버 리스트를 볼 수 있습니다. 이 과정을 분석해봅니다.
1. openstack 명령어를 사용하므로 이 명령어가 어디에 있는지 알아야합니다. which를 사용하면 이 명령어의 위치를 알 수 있습니다.

openstack 명령어 위치
파일을 확인해보겠습니다. cat을 사용하면 파일의 내용을 출력할 수 있습니다.

openstack 파일 내용
2. 메인함수가 하는 일은 openstackclient.shell의 main을 실행하는 것 뿐입니다. openstackclient 패키지가 어디에 있는지 확인해봅니다.

openstackclient 위치
3. openstackclient 패키지를 분석해보면 어떻게 서버 리스트가 출력되는지 알 수 있을 것 같습니다. 터미널보다 소스코드 편집기를 이용하는게 더 편한 분들은 https://opendev.org/openstack/python-openstackclient 여기서 python-openstackclient를 clone하고 tag 5.3.1 코드를 보셔도 됩니다.
4. openstackclient에서 shell.py를 분석해봅니다.
main을 import 했으므로 openstack server list 명령어를 실행했을 때 main 함수가 실행되는지 확인해보겠습니다.

shell.py의 main함수를 위와 같이 수정하겠습니다. 이렇게 수정하고 싶다면 sudo vi shell.py를 입력합니다.
원하는 곳으로 이동할 때 쓰는 방법으로 첫째는 해당 줄로 이동하는 것입니다. 142번째 줄에 있으므로 esc -> 142 -> G를 입력하면 커서가 142번 줄로 이동합니다.
두번째 방법은 main이라는 키워드로 검색하는 것입니다. esc -> / -> main를 입력하고 n(next)을 입력하여 다음으로 매칭되는 main을 계속 찾아갈 수 있습니다.
이렇게 코드를 수정하고 실행해보겠습니다.

추가한 코드가 출력되었습니다. 이런식으로 코드를 분석하면서 실제로 실행되는지 print로 확인할 수 있습니다.
5. 여기서 더 들어가면 osc_lib의 shell.py, cliff의 app 등등 끝까지 타고 올라가는데 이는 오픈스택과 멀어지는 뇌절의 길입니다. 다시 openstackclient로 돌아와서 구조를 살펴보고 어디로 파고들지 생각해봅시다.
6. openstackclient의 구조는 이렇습니다. identity, image, network, object, volume등은 첫시간에 설명들은 5가지 요소들 인 것 같습니다.

openstackclient의 구조
그 중에서 openstack server list 명령어는 compute와 가장 관련있을 것 같습니다. compute를 자세히 보면

compute의 구조
client.py라는 파일이 있고 v2안에 여러개가 있습니다. 이 구조는 object든 network든 마찬가지였습니다. openstack server list 명령을 실행했을 때 compute client가 생성되는지 확인하기 위해 다음과 같이 코드를 수정합니다. object폴더에도 client.py를 아래처럼 수정해줍니다.

compute/client.py 수정

결과 확인
‘compute client는 필요할까?’ 가 출력된 것을 확인했습니다! ‘compute client는 필요할까?’는 출력되었지만, ‘object client는 필요할까?’는 출력되지 않은 것으로 보아 compute의 client.py만 필요한 것 같습니다.
7. compute의 client.py가 실행되는 것을 파악했으니 이제 v2에 있는 파일들을 보겠습니다. 대부분의 클래스 들이 from osc_lib.command import command를 상속 받아 구현했고 get_parser, take_action 함수들이 있는 것을 확인할 수 있습니다. 함수 이름을 보아하니, get_parser로 args를 파싱하고 그에 맞는 데이터를 take_action으로 가져오지 않을까 추측할 수 있습니다.
8. 그럼 v2에 있는 파일들 중 server list 명령어가 실행될 때 어떤 함수가 실행되는걸까요? vscode에서 ⌘ + ⇧ + F 를 입력하여 search 기능을 사용해봅니다. 여기에 server를 입력해봅시다.
Image for post
140개의 파일에서 3761개 매칭…
많은 결과가 나옵니다. 그럼 여기서 server는 결과가 너무 많으니 server list, server_list, list server 등 결과를 좁힐 수 있는 것들로 검색해보는 것도 좋습니다. 하지만 저는 일단 보기로 했습니다. setup.cfg파일을 보겠습니다.
9. 이 파일을 보니 저희가 찾던 server_list가 있습니다. 비록 server list랑은 다르지만, 코드상으로 표현할 때는 스페이스를 언더바로 바꿔서 표현했나보다 하고 그러려니 해봅니다. 왠지 이 코드를 보니 server_list는 openstackclient.compute.v2.server에 ListServer를 실행한다. 라고 말하는 것 같습니다. 그럼 print 해봅시다.

10. compute.v2.server에서 ListServer 클래스 코드를 조금 수정합니다.

이렇게 수정하고 코드를 실행하면 ArugmentParser에 openstack server list 명령어가 담겨 있고 table에는 출력으로 나오는 table의 헤더(ID, Name, Status 등)와 이에 필요한 데이터를 가져올 제네레이터가 있는 것을 알 수 있습니다.

11. 이제 다 왔습니다. openstack server list은 server.py의 ListServer class에서 출력 결과를 만듭니다. 그럼 우리 입맛대로 한가지 항목을 추가해봅시다.

take_action 함수의 마지막 부분입니다. 콘솔에 나오는 테이블 형식의 출력이 table 변수에 있으니, 여기에 항목을 추가해봅시다. 그럼 column_headers와 columns에 ‘Flavor ID’를 추가해봅니다.

ListServer class의 take_action에 있는 columns와 column_headers에 Flavor ID 항목을 추가합니다. ‘Flavor ID’ 항목을 추가한 이유는 else문 위에 if parsed_args.long: 부분에 columns에 있었기 때문입니다. 코드를 분석하는 중 또 하나 알게된 것은 ‘openstack server list’ 명령어 말고 openstack server list — long’ 명령어를 입력하면 좀 더 많은 columns들이 출력되고 그 항목들이 바로 if parsed_args.long:에 있는 항목들 입니다.

짜잔! 테이블의 가장 오른쪽에 우리가 추가한 Flavor ID가 출력되었습니다.
여기까지 openstack server list 커맨드 실행 시 코드 분석과 출력 결과를 바꿔보는 것에 대해 알아봤습니다.
