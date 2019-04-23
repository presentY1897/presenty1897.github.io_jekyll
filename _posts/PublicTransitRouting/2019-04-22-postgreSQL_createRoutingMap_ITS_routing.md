---
layout: post
title: "pgRouting으로 경로 찾기"
description: "앞서 구축한 link 데이터를 이용하여 pgRouting의 경로를 탐색"
comments: true
keywords: "postgreSQL, pgRouting, ITS, GeoSERVER, GIS"
tags: postgreSQL
---

## pgRouting dijkstra

이제 경로 탐색 알고리즘에서 가장 기초적인 dijkstra를 사용해봅시다.

```sql
SELECT pg_dijkstra()
```

이제 잘 들어가는지 확인해봐야죠.

QGIS를 켜서 확인해봅시다.

QGIS에서 구축한 link 테이블을 연결하고
필터에 다음 구문을 넣어 맞게 움직이는 지 봅시다.

```sql

```

적당히 넣어 봤더니 그러저럭 잘 되는 것 같습니다.

[!쿼리 입력 이미지]()
[!결과 이미지]()

다만, 해당 데이터는 실제로 완전하진 않습니다. 몇몇 오류 데이터들이 존재하고 있으며 이는 만약에 이 데이터를 이용해 정밀한 결과를 얻고자 한다면 문제가 될 수 있습니다. 이를 해결하는 방법은 나중에 포스팅하겠습니다. 우선은 pgRouting을 활용하는 데 집중합시다.

## GeoSERVER 구축

이제 만든 데이터를 웹에서 사용할 수 있게 GeoSERVER를 활용합니다. GeoSERVER는 지도 개체를 제공하는 서버를 구축할 수 있는 오픈 소스입니다.
설치는 순서에 맞춰서 진행하면 됩니다.

설치가 끝나면 start geoserver를 실행하여 geoserver를 실행하고, geoadmin을 실행하여 관리자 모드로 들어갑니다.

아마 admin의 이름과 비밀번호를 바꾸지 않았다면, 입니다.

로그인 한 뒤에 다음을 따라서 수행하면 서버에서 데이터를 내보낼 준비가 끝났습니다.

## html파일 생성

이제 간단한 웹페이지를 만들어 link에서 찾은 경로를 지도에 띄워봅니다.

html은 osm3를 이용합니다. 참고는 [서울 버전 pgRouting 예제]()를 참고하였습니다. 다만 해당 소스는 구버전을 기준으로 작성되어있어 실제로 작동하지 않는 부분이 많아 부분적으로 수정하였습니다.

```html
```

이제 약간 느리지만 웹페이지에서 ITS Link를 따라 두 공간을 연결하는 최단 경로를 찾을 수 있습니다. :)