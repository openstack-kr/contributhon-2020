==============================
Contribution: 첫 번째 이슈
==============================

첫 컨트리뷰트가 될 이슈를 `이것 <https://storyboard.openstack.org/#!/story/2007860>`_ 으로 결정했다.

-----------
이슈 분석
-----------

``openstack server create`` 명령어로 인스턴스를 생성할 때, --image-property 옵션을 사용할 수 있었다.
property라는 단어가 ``openstack image show`` 로 이미지 정보를 볼 때 나오는 properties와 관련이 있다고 생각해서 cirros 이미지를 찾아보았다.

.. code-block:: shell

    $ openstack image show  cirros-0.5.1-x86_64-disk
    +------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Field            | Value                                                                                                                                                                                                                                                                                                                                                                            |
    +------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | checksum         | 1d3062cd89af34e419f7100277f38b2b                                                                                                                                                                                                                                                                                                                                                 |
    | container_format | bare                                                                                                                                                                                                                                                                                                                                                                             |
    | created_at       | 2020-08-15T12:28:18Z                                                                                                                                                                                                                                                                                                                                                             |
    | disk_format      | qcow2                                                                                                                                                                                                                                                                                                                                                                            |
    | file             | /v2/images/56aad641-dd16-44af-86f0-b61ec980709c/file                                                                                                                                                                                                                                                                                                                             |
    | id               | 56aad641-dd16-44af-86f0-b61ec980709c                                                                                                                                                                                                                                                                                                                                             |
    | min_disk         | 0                                                                                                                                                                                                                                                                                                                                                                                |
    | min_ram          | 0                                                                                                                                                                                                                                                                                                                                                                                |
    | name             | cirros-0.5.1-x86_64-disk                                                                                                                                                                                                                                                                                                                                                         |
    | owner            | e90e398ff160422b84557c3924f775fe                                                                                                                                                                                                                                                                                                                                                 |
    | properties       | hw_rng_model='virtio', os_hash_algo='sha512', os_hash_value='553d220ed58cfee7dafe003c446a9f197ab5edf8ffc09396c74187cf83873c877e7ae041cb80f3b91489acf687183adcd689b53b38e3ddd22e627e7f98a09c46', os_hidden='False', os_version='0.5.1', owner_specified.openstack.md5='', owner_specified.openstack.object='images/cirros-0.5.1-x86_64-disk', owner_specified.openstack.sha256='' |
    | protected        | False                                                                                                                                                                                                                                                                                                                                                                            |
    | schema           | /v2/schemas/image                                                                                                                                                                                                                                                                                                                                                                |
    | size             | 16338944                                                                                                                                                                                                                                                                                                                                                                         |
    | status           | active                                                                                                                                                                                                                                                                                                                                                                           |
    | tags             |                                                                                                                                                                                                                                                                                                                                                                                  |
    | updated_at       | 2020-08-15T12:33:37Z                                                                                                                                                                                                                                                                                                                                                             |
    | visibility       | public                                                                                                                                                                                                                                                                                                                                                                           |
    +------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

이 결과에서 나오는 프로퍼티를 사용해 VM을 생성하는 방식을 여러 번 시도했다. 여기서 한 가지 문제점을 발견했다.

--------
문제점
--------

이미지가 여러 개 있을 때, 프로퍼티로 이미지를 선택하는 것이 제대로 동작하지 않는 것을 발견했다. 사용한 명령어는 다음과 같다.

.. code-block:: shell

    $ openstack image set --os-version 0.5.1 cirros-0.5.1-x86_64-disk
    $ openstack image save cirros-0.5.1-x86_64-disk --file cirros.image
    $ openstack image create cirros --file cirros.image

    +------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
    | Field            | Value                                                                                                                                      |
    +------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
    | container_format | bare                                                                                                                                       |
    | created_at       | 2020-08-17T10:48:18Z                                                                                                                       |
    | disk_format      | raw                                                                                                                                        |
    | file             | /v2/images/53cbd213-a39c-4d88-b8e1-5e14a512e292/file                                                                                       |
    | id               | 53cbd213-a39c-4d88-b8e1-5e14a512e292                                                                                                       |
    | min_disk         | 0                                                                                                                                          |
    | min_ram          | 0                                                                                                                                          |
    | name             | cirros                                                                                                                                     |
    | owner            | e1c2fb4e069e4a5a8935bef43df00e85                                                                                                           |
    | properties       | os_hidden='False', owner_specified.openstack.md5='', owner_specified.openstack.object='images/cirros', owner_specified.openstack.sha256='' |
    | protected        | False                                                                                                                                      |
    | schema           | /v2/schemas/image                                                                                                                          |
    | status           | queued                                                                                                                                     |
    | tags             |                                                                                                                                            |
    | updated_at       | 2020-08-17T10:48:18Z                                                                                                                       |
    | visibility       | shared                                                                                                                                     |
    +------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
    $ openstack server create cirros-test \
    --flavor 1 \
    --key-name key \
    --image-property os_version=0.5.1

    No images match the property expected by --image-property
    
위의 명령어들을 요약하면 다음과 같다.

1. 기존 이미지(cirros-0.5.1-x86_64-disk)에 ``os_version`` 이라는 프로퍼티를 설정한다.
2. 기존 이미지를 저장한 뒤, 그 파일로 새로운 이미지를 등록한다.
3. 서버를 생성할 때 --image-property에 ``os_version`` 프로퍼티를 사용한다.

분명히 ``os_version`` 을 설정했음에도 불구하고 이미지를 선택하지 못하는 것을 확인할 수 있었다.

-----------
코드 분석
-----------

발견한 문제점을 토대로 코드를 분석했다. 분석해야 할 파일은  python-openstackclient 프로젝트에서 openstackclient/compute/v2/server.py 파일이며, VM을 생성하는 함수는 ``CreateServer`` 클래스의 ``take_action()`` 이다. 

``CreateServer`` 클래스의 ``take_action()`` 를 살펴보니, ``_match_image()`` 라는 함수가 ``take_action()`` 내부에 정의되어 있었고, 여기서 --image-propert 옵션과 관련된 로직이 실행되고 있었다. 해당 함수는 아래와 같은 모습이다.

.. code-block:: python3

    def _match_image(image_api, wanted_properties):
    image_list = image_api.images()
    images_matched = []
    for img in image_list:
        img_dict = {}
        # exclude any unhashable entries
        for key, value in img.items():
            try:
                set([key, value])
            except TypeError:
                pass
            else:
                img_dict[key] = value
        if all(k in img_dict and img_dict[k] == v
                for k, v in wanted_properties.items()):
            images_matched.append(img)
        else:
            return []
    return images_matched

이 함수의 문제점은 --image-property로 명시한 조건과 일치하지 않는 이미지가 존재하면 필터링된 이미지 리스트(위 함수에서는 ``images_matched`` )가 아닌 빈 리스트를 리턴한다는 것이다. 이렇게 되면 --image-property 조건과 일치하는 이미지가 있어도 무시될 가능성이 존재하며, 그 경우가 바로 위에서 재현된 버그와 같은 것이다. 

이 문제는 빈 리스트를 반환하는 코드 두 줄을 제거하여 수정했다.

.. code-block:: python3

    def _match_image(image_api, wanted_properties):
    image_list = image_api.images()
    images_matched = []
    for img in image_list:
        img_dict = {}
        # exclude any unhashable entries
        for key, value in img.items():
            try:
                set([key, value])
            except TypeError:
                pass
            else:
                img_dict[key] = value
        if all(k in img_dict and img_dict[k] == v
                for k, v in wanted_properties.items()):
            images_matched.append(img)
    return images_matched

---------------
또 다른 문제점
---------------

이슈를 올렸던 작성자가 직접 `리뷰 <https://review.opendev.org/#/c/740455/3>`_ 도 올린 것을 뒤늦게 확인했다. 
코드를 살펴보니 내가 발견한 것과는 다른 문제점을 발견한 것을 알게 되었다.

예를 들면, 이미지 프로퍼티 중 ``owner_specified.openstack.object`` 라는 키를 --image-property 조건으로 넣으면 이미지가 생성되지 않는다.

.. code-block:: shell

    $ openstack image list
    +--------------------------------------+--------------------------+--------+
    | ID                                   | Name                     | Status |
    +--------------------------------------+--------------------------+--------+
    | 56aad641-dd16-44af-86f0-b61ec980709c | cirros-0.5.1-x86_64-disk | active |
    +--------------------------------------+--------------------------+--------+

    $ openstack server create --flavor 1 --key-name key --image-property owner_specified.openstack.object=images/cirros-0.5.1-x86_64-disk --network private cirros-test

    No images match the property expected by --image-property

이전과는 달리, 이미지가 하나만 있는데도 서버 생성에 실패했다. 왜 이런 문제가 발생하는지 알아보기 위해 다시 디버깅을 시도했다.

.. code-block:: python3

    def _match_image(image_api, wanted_properties):
    image_list = image_api.images()
    images_matched = []
    for img in image_list:
        img_dict = {}
        # exclude any unhashable entries
        for key, value in img.items():
            try:
                set([key, value])
            except TypeError:
                pass
            else:
                img_dict[key] = value
        if all(k in img_dict and img_dict[k] == v
                for k, v in wanted_properties.items()):
            images_matched.append(img)
    return images_matched

이 코드에서, ``img`` 오브젝트는 이미지의 여러 프로퍼티를 저장한 딕셔너리였다. 딕셔너리의 키를 순회하면서 하나의 set으로 만들 수 있는 것만 --image-property에 사용할 수 있는 키 값의 대상이 되는 것이었다. 

그런데 앞서 사용한 ``owner_specified.openstack.object`` 프로퍼티는 ``img`` 의 ``properties`` 라는 키에 딕셔너리로 저장된 값 중 하나였다. 따라서 ``properties`` 라는 키에 저장된 프로퍼티는 --image-property 필터에 사용할 수 없었던 것이다.

------------
리뷰 작성!
------------

두 가지 경우 모두 --image-property가 제대로 동작하지 않는 원인이기 떄문에 둘 다 수정할 필요가 있었다.
내가 수정한 코드는 다음과 같다.

.. code-block:: python3

    def _match_image(image_api, wanted_properties):
    image_list = image_api.images()
    images_matched = []
    for img in image_list:
        img_dict = {}
        # exclude any unhashable entries
        img_dict_items = list(img.items())
        if img.properties:
            img_dict_items.extend(list(img.properties.items()))
        for key, value in img_dict_items:
            try:
                set([key, value])
            except TypeError:
                pass
            else:
                img_dict[key] = value
        if all(k in img_dict and img_dict[k] == v
                for k, v in wanted_properties.items()):
            images_matched.append(img)
    return images_matched

그리고 이 두 가지 문제점을 포함한 테스트 케이스 하나를 작성한 다음, gerrit에 리뷰를 작성했다.

- `리뷰 링크 <https://review.opendev.org/#/c/746405/1/openstackclient/compute/v2/server.py>`_
