====================================================================================================================================
[이슈 코멘트] $ openstack availability zone list 의 wrong available status
====================================================================================================================================

------------
이슈 소개
------------

    During a recent infrastructure test exercise we completely pulled the plug on a whole AZ. To our surprise, openstack availability zone list reported all AZs as available even if one of them was clearly down.
    This is confusing for tools that check on that output to detect full AZ failures.

Issue: https://storyboard.openstack.org/#!/story/2006816

``Availability zone`` 을 수동으로 비활성화시켰음에도, ``$ openstack availability zone list`` 에서는 AZ가 Healthy한 state로 나온다는
Issue가 있었습니다. 이 Issue를 제 환경에서 실행해보았고, 저는 bug가 재현되지 않았기에,
저의 내용을 추가적으로 Comment에 기입해주었습니다.

------------------
이슈 재현하기
------------------

    우선 Aggregate란.

    Host aggregates provide a mechanism to group hosts according to certain criteria.

aggregate를 만들어주고, 거기에 host를 추가함으로써 availability zone을 만들 수 있고,
host를 disable 함으로써 availability zone이 not available 하게 만드는 흐름입니다.

.. code-block:: bash

    $ openstack aggregate create dev-aggregate
    $ openstack aggregate set --zone dev-zone dev-aggregate
    $ openstack aggregate add host dev-aggregate openstack-master
    $ openstack availability zone list
    +-----------+-------------+
    | Zone Name | Zone Status |
    +-----------+-------------+
    | dev-zone  | available   |
    | internal  | available   |
    | nova      | available   |
    | nova      | available   |
    | nova      | available   |
    +-----------+-------------+

    $ nova service-list
    $ nova service-disable <service_id>

    $ openstack availability zone list
    +-----------+---------------+
    | Zone Name | Zone Status   |
    +-----------+---------------+
    | internal  | available     |
    | dev-zone  | not available |
    | nova      | available     |
    | nova      | available     |
    | nova      | available     |
    +-----------+---------------+

--------------------------------------------
버그가 재현되지 않았던 이유 추측
--------------------------------------------

아마도

Nova가 자신이 API를 수행하면서는 AZ의 availability를 체크를 하지만,
random unhealthy state에 대한 반영은 하지 않을 수도 있겠다는 생각을 했습니다.

따라서 혹시 저와 같이 수행했을 때에도 Bug가 재현되는지, Issue 제기자의 좀 더 자세한
상황이 어땠는지를 여쭤보는 Comment를 달아 다른 사람들과 소통해보고자 했습니다.

-----------------------
Issue에 Comment 작성
-----------------------

    .. image:: images/comment_1.png

    .. image:: images/comment_2.png

---------
느낀 점
---------

사실 버그가 재현되어서 하나 버그 픽스를 해보고싶었지만, 저의 방법에서는
재현되지 않아서 조금 아쉬웠습니다.
또한 Availability Zone이라는 것이 개인적인 실습 수준에서는 쉽게 접근하기 어려운 내용이라
어려웠던 점이 있었던 것 같습니다.

하지만 꼭 버그 픽스나 merge가 아니더라도 이런 식으로 소소하게 기여를 하다보면,
점점 큰 기여를 할 수 있는 밑거름이 될 것이라 생각합니다!