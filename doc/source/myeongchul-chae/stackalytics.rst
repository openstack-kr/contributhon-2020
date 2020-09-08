==============================
stackalytics 등록 방법
==============================

`stackalytics 저장소 <https://github.com/stackalytics/default_data/>`_ 를 Fork한다.

그런 다음, default_data.json에 자기 자신의 user 정보를 넣어야 한다.

User 정보는 launchpad id, ldap id, github id 순서대로 넣어야 한다.

계정 정보 예시는 다음과 같다.

.. code-block:: json

    {
        "launchpad_id": "cocahack",
        "github_id": "cocahack",
        "zanata_id": "jake_chae",
        "companies": [
        {
            "company_name": "*independent",
            "end_date": null
        }
        ],
        "user_name": "Myeongchul Chae",
        "emails": ["cocahack@naver.com", "rncchae@gmail.com"]
    }

계정 정보를 채웠다면 tox를 실행해서 정보를 잘 기입했는지 확인하면 된다.

.. code-block:: shell

    {
        "launchpad_id": "cocahack",
        "github_id": "cocahack",
        "zanata_id": "jake_chae",
        "companies": [
        {
            "company_name": "*independent",
            "end_date": null
        }
        ],
        "user_name": "Myeongchul Chae",
        "emails": ["cocahack@naver.com", "rncchae@gmail.com"]
    }

