==========================================================================================
[이슈] Image upload doesn’t support the upload progress like in glance client
==========================================================================================

----------------------
이슈 소개
----------------------

스토리 보드: https://storyboard.openstack.org/#!/story/2007777

스토리 보드의 내용은 다음과 같습니다.


이미지 업로드시 ``glance client`` 의 ``--progress`` 옵션을 ``openstack client`` 에서도
지원을 했으면 좋겠다는 이슈입니다.

아마 ``glance client`` 에 있는 progress 기능을 ``openstack client``  로 이식하면 될 거 같습니다.

-----------------
이슈 재현하기
-----------------


재현을 위해 기존 devstack 서버에 ``glance client`` 설치를 하였습니다.

.. code-block:: shell

    apt install glance #패키지 관리자를 이용해 glance 를 설치
    source openrc admin # devstack 폴더에서 접속 계정을 admin로 설정


``glance client`` 가 제대로 설치가 되었는지, 그리고 ``openstack client`` 에서 이와 비슷하게 사용되는 
사용되는 커맨드는 무엇인지 찾기 위해 image list 를 조회하는 커맨드를 입력해보았습니다.


.. code-block:: shell

    stack@server1:~/devstack$ glance image-list
    +--------------------------------------+--------------------------+
    | ID                                   | Name                     |
    +--------------------------------------+--------------------------+
    | aa08f22f-d505-4aa3-80e4-49f34cae21e2 | 2u2buntu5                |
    | 3b6445d7-a301-40fa-82ca-205b341bf41e | cirros                   |
    +--------------------------------------+--------------------------+


.. code-block:: shell

    stack@server1:~/devstack$ openstack image list
    +--------------------------------------+--------------------------+--------+
    | ID                                   | Name                     | Status |
    +--------------------------------------+--------------------------+--------+
    | aa08f22f-d505-4aa3-80e4-49f34cae21e2 | 2u2buntu5                | active |
    | f5b65c06-d5aa-47b4-b304-ef95b3d31a9d | cirros                   | active |
    +--------------------------------------+--------------------------+--------+

``openstack clinet`` 에서는 Status 를 컬럼을 추가로 보여주는 것을 제외하고는 같은 결과 값을 내보내는 것을 확인할 수 있었습니다.

progress 재현
--------------------

같은 서버로 요청을 받아서 처리하는 것을 확인했으니, glance에 progress 옵션을 주어 이미지를 업로드 해보겠습니다.

.. code-block:: shell

    #서버에 업로드 하기 위한 이미지 다운
    stack@server1:~$ wget http://cloud-images-archive.ubuntu.com/releases/bionic/release-20200519.1/ubuntu-18.04-server-cloudimg-arm64.img

    #glance client를 사용해 다운 받은 이미지 업로드
    stack@server1:~$ glance image-create --name "ubuntu" --file /opt/stack/ubuntu-18.04-server-cloudimg-arm64.img --disk-format qcow2 --container-format bare --visibility public --progress
    [=============================>] 100%
    +------------------+----------------------------------------------------------------------------------+
    | Property         | Value                                                                            |
    +------------------+----------------------------------------------------------------------------------+
    | checksum         | 894d3c7009fc7ce476fdf2fabd403745                                                 |
    | container_format | bare                                                                             |
    | created_at       | 2020-09-01T20:48:49Z                                                             |
    | disk_format      | qcow2                                                                            |
    | id               | f78f5ce8-b6cc-4998-b4d5-ed6f71d11f7e                                             |
    | min_disk         | 0                                                                                |
    | min_ram          | 0                                                                                |
    | name             | ubuntu                                                                           |
    | os_hash_algo     | sha512                                                                           |
    | os_hash_value    | 4683e1da762e246c7e2ddaecb4830af1a31da0bd2b25c1554ed288e24b79a14c6e42cb5b4a32146c |
    |                  | fc0e7b61b64e3de4585be8fb0f4c4891ddba5290505d7c4a                                 |
    | os_hidden        | False                                                                            |
    | owner            | c4c153328c74498cba350e0db85c3a67                                                 |
    | protected        | False                                                                            |
    | size             | 327352320                                                                        |
    | status           | active                                                                           |
    | tags             | []                                                                               |
    | updated_at       | 2020-09-01T20:48:52Z                                                             |
    | virtual_size     | Not available                                                                    |
    | visibility       | public                                                                           |
    +------------------+----------------------------------------------------------------------------------+

glance를 사용해서 업로드를 한 결과 ``[=============================>] 100%``
같은 형태로 진행률과 함께 업로드 되는 것을 볼 수 있습니다.

-----------
코드 분석
-----------

glance client는 `python-glanceclient <https://github.com/openstack/python-glanceclient>`_ 에 코드가 있습니다.
glance client 에서 progress 옵션을 어디에서 주는지 찾아보았습니다. 

전체 검색으로 ``--progress`` 를 검색해 봤습니다.

.. code-block:: python

    @utils.arg('--progress', action='store_true', default=False,
            help=_('Show upload progress bar.'))
    @utils.arg('id', metavar='<IMAGE_ID>',
            help=_('ID of image to upload data to.'))
    @utils.arg('--store', metavar='<STORE>',
            default=utils.env('OS_IMAGE_STORE', default=None),
            help='Backend store to upload image to.')
    def do_image_upload(gc, args):
        """Upload data for a specific image."""
        # 생략

다음과 glance client에서는 위와 같은 방법으로 옵션 인자(arguments)를 넣어 주는 듯 합니다. 
그리고 밑으로 좀 내리면 ``def do_image_upload()`` , ``def do_image_create_via_import()``, ``def do_image_create()`` 
등 여러 함수들을 볼 수 있습니다. 

이 중에 업로드 기능을 구현하고자 하므로, ``do_image_upload`` 함수를 살펴보겠습니다.
링크: `do_image_upload 함수 stable/ussuri 버전 
<https://github.com/openstack/python-glanceclient/blob/4c63903403d7ef7801c8e274b67f9647ff329991/glanceclient/v2/shell.py#L639>`_



.. code-block:: python

    def do_image_upload(gc, args):
        """Upload data for a specific image."""
        backend = None
        if args.store:
            backend = args.store
            # determine if backend is valid
            _validate_backend(backend, gc)

        image_data = utils.get_data_file(args)
        if args.progress:
            filesize = utils.get_file_size(image_data)
            if filesize is not None:
                # NOTE(kragniz): do not show a progress bar if the size of the
                # input is unknown (most likely a piped input)
                image_data = progressbar.VerboseFileWrapper(image_data, filesize)
        gc.images.upload(args.id, image_data, args.size, backend=backend)

``do_image_upload`` 함수는 ``python-glanceclient/glanceclient/v2/shell.py`` 에 위치하고 있습니다. 
여기에서 주목해야 할 부분은 ``if args.progress:`` 입니다.

우리가 ``--progress`` 옵션을 주었을 때만 해당 if 문이 ``참(True)`` 이 되게 됩니다. 

filesize를 구하고, None이 아닐 경우에만 ``image_data = progressbar.VerboseFileWrapper(image_data, filesize)`` 함수가 작동 되네요.

``progressbar.VerboseFileWrapper()`` 이 친구에게 인자값을 넘겨주고, 받은 데이터를 그대로 사용하는 것을 볼 수 있습니다. 직역하면 자세한 파일 감싸기(래퍼) 라는 이름을 가지고 있습니다.

저 친구를 어디에서 데리고 왔는지 ``progressbar.`` 가 `선언된 곳 <https://github.com/openstack/python-glanceclient/blob/4c63903403d7ef7801c8e274b67f9647ff329991/glanceclient/v2/shell.py#L23>`_ 을 찾아보겠습니다.

.. code-block:: python

    import json
    import os
    import sys

    from oslo_utils import strutils

    from glanceclient._i18n import _
    from glanceclient.common import progressbar


제일 최상단에서 import 하고 있었습니다. from glanceclient.common 에서 import 했으므로 
`해당 코드 <https://github.com/openstack/python-glanceclient/blob/4c63903403d7ef7801c8e274b67f9647ff329991/glanceclient/common/progressbar.py#L53>`_ 
로 이동해보겠습니다.

.. code-block:: python

    class VerboseFileWrapper(_ProgressBarBase):
    """A file wrapper with a progress bar.
    The file wrapper shows and advances a progress bar whenever the
    wrapped file's read method is called.
    """

    def read(self, *args, **kwargs):
        data = self._wrapped.read(*args, **kwargs)
        if data:
            self._display_progress_bar(len(data))
        else:
            if self._show_progress:
                # Break to a new line from the progress bar for incoming
                # output.
                sys.stdout.write('\n')
        return data


`` _ProgressBarBase`` 를 상속받은 ``VerboseFileWrapper`` 클래스를 사용했네요.

``openstack-clinet`` 에 ``python-glanceclient/glanceclient/common/progressbar.py`` 를 이식하면 progressbar를 사용할 수 있을거 같습니다.

------------
코드 작성
------------

이제 코드 이식을 위해 openstack-client 로 가보겠습니다.

.. code-block:: python

    class CreateImage(command.ShowOne):
        _description = _("Create/upload an image")

        deadopts = ('size', 'location', 'copy-from', 'checksum', 'store')

        def get_parser(self, prog_name):
            parser = super(CreateImage, self).get_parser(prog_name)
            # TODO(bunting): There are additional arguments that v1 supported
            # that v2 either doesn't support or supports weirdly.
            # --checksum - could be faked clientside perhaps?
            # --location - maybe location add?
            # --size - passing image size is actually broken in python-glanceclient
            # --copy-from - does not exist in v2
            # --store - does not exits in v2
            parser.add_argument(
                "name",
                metavar="<image-name>",
                help=_("New image name"),
            )
            # 생략

openstack-clinet 에서 이미지는 
`class createImage <https://github.com/openstack/python-openstackclient/blob/5b25ea899e023bcd3d7384cca943f9844bcb0b79/openstackclient/image/v2/image.py#L184>`_
클래스의 ``def get_parser()`` 에서 인자(arguments)를 추가 시켜줄 수 있고, ``def take_action()`` 에서 실제 이미지 업로드가 됩니다.

``def get_parser()`` 에서 --progress 옵션을 추가시켜 보겠습니다.

.. code-block:: python

    def get_parser(self, prog_name):
    
    #생략
    public_group.add_argument(
        "--shared",
        action="store_true",
        help=_("Image can be shared"),
    )
    parser.add_argument(
        "--progress",
        action="store_true",
        default=False,
        help=_("Show upload progress bar."),
    )

``--progress`` 옵션을 추가 시켰습니다.

.. code-block:: python

    def take_action(self, parsed_args):

        # open the file first to ensure any failures are handled before the
        # image is created. Get the file name (if it is file, and not stdin)
        # for easier further handling.
        (fp, fname) = get_data_file(parsed_args)
        info = {}

        if fp is not None and parsed_args.volume:
            raise exceptions.CommandError(_("Uploading data and using "
                                            "container are not allowed at "
                                            "the same time"))
        if fp is None and parsed_args.file:
            LOG.warning(_("Failed to get an image file."))
            return {}, {}
        elif fname:
            kwargs['filename'] = fname
        elif fp:
            kwargs['validate_checksum'] = False
            kwargs['data'] = fp
        
        # 생략
        if parsed_args.volume:
            #생략
        else:
            image = image_client.create_image(**kwargs)



`(fp, fname) = get_data_file(parsed_args) <https://github.com/openstack/python-openstackclient/blob/5b25ea899e023bcd3d7384cca943f9844bcb0b79/openstackclient/image/v2/image.py#L394>`_ 
에서 파일 포인터와 파일 이름을 가져오는 것을 확인 할 수 있습니다.

이렇게 가져온 데이터를 kwargs['data']에 넣는데, 이렇게 넣은 kwargs['data']는 
`image = image_client.create_image(**kwargs) <https://github.com/openstack/python-openstackclient/blob/5b25ea899e023bcd3d7384cca943f9844bcb0b79/openstackclient/image/v2/image.py#L471>`_ 
에 전달 되면서 서버에 이미지 data가 업로드 되게 됩니다.

fp 를 VerboseFileWrapper로 감싸면 progressbar 를 구현할 수 있을것이라고 생각하고 접근하였습니다.

이를 위해 ``python-openstackclient/openstackclient/common/`` 위치에 ``python-glanceclient/glanceclient/common/progressbar.py`` 를 복사한 
``progressbar.py`` 를 만들었습니다.

glance-client의 `progressbar.py <https://github.com/openstack/python-glanceclient/blob/master/glanceclient/common/progressbar.py>`_ 를 그대로 옮겨왔다고 생각하시면 됩니다.


.. code-block:: python

    from openstackclient.common import progressbar

    #생략

    def take_action(self, parsed_args):

        # open the file first to ensure any failures are handled before the
        # image is created. Get the file name (if it is file, and not stdin)
        # for easier further handling.
        (fp, fname) = get_data_file(parsed_args)
        info = {}

        if fp is not None and parsed_args.volume:
            raise exceptions.CommandError(_("Uploading data and using "
                                            "container are not allowed at "
                                            "the same time"))
        if fp is None and parsed_args.file:
            LOG.warning(_("Failed to get an image file."))
            return {}, {}
        if parsed_args.progress:
            kwargs['validate_checksum'] = False
            kwargs['data'] = progressbar.VerboseFileWrapper(fp, os.path.getsize(fname))


``if parsed_args.progress:`` 가 나올 경우, VerboseFileWrapper 를 사용해 파일 포인터를 감싸서 kwargs['data']에 넣어 주었습니다.

.. code-block:: shell

    stack@server1:~$ openstack image create ubuntu123 --file /opt/stack/ubuntu-18.04-server-cloudimg-arm64.img --disk-format qcow2 --container-format bare --public --progress
    base_proxy
    [=============================>] 100%
    +------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Field            | Value                                                                                                                                                                                                 |
    +------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | container_format | bare                                                                                                                                                                                                  |
    | created_at       | 2020-09-02T12:50:17Z                                                                                                                                                                                  |
    | disk_format      | qcow2                                                                                                                                                                                                 |
    | file             | /v2/images/5f70aa73-7f14-4244-8696-383fabb1d30a/file                                                                                                                                                  |
    | id               | 5f70aa73-7f14-4244-8696-383fabb1d30a                                                                                                                                                                  |
    | min_disk         | 0                                                                                                                                                                                                     |
    | min_ram          | 0                                                                                                                                                                                                     |
    | name             | ubuntu123                                                                                                                                                                                             |
    | owner            | c4c153328c74498cba350e0db85c3a67                                                                                                                                                                      |
    | properties       | os_hidden='False', owner_specified.openstack.md5='', owner_specified.openstack.object='images/ubuntu123', owner_specified.openstack.sha256='', self='/v2/images/5f70aa73-7f14-4244-8696-383fabb1d30a' |
    | protected        | False                                                                                                                                                                                                 |
    | schema           | /v2/schemas/image                                                                                                                                                                                     |
    | status           | queued                                                                                                                                                                                                |
    | tags             |                                                                                                                                                                                                       |
    | updated_at       | 2020-09-02T12:50:17Z                                                                                                                                                                                  |
    | visibility       | public                                                                                                                                                                                                |
    +------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

이미지를 업로드 할 시 openstack-client 에서도 progressbar([=============================>] 100%) 가 보이는 것을 확인할 수 있습니다.

이후 테스트 코드 작성, 코드 최적화, 테스트, 이슈 등록에 대한 글을 쓰도록 하도록 하겠습니다.

마지막으로 삽질하고 있을 때 조언과 많은 도움을 주신 멘토님께 감사드립니다.
