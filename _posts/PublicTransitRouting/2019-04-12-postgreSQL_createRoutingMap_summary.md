---
layout: post
title: "ITS node-link 데이터를 이용한 PostgreSQL 기반 경로 탐색 지도 만들기"
description: "PostgreSQL은 기본적인 경로탐색 알고리즘을 포함하는 pgRouting을 제공합니다. ITS에서 제공하는 전국의 도로망 데이터를 이용해 경로를 찾는 웹페이지를 구축하여봤습니다. ㅎㅎ"
comments: true
keywords: "postgreSQL, pgRouting, ITS, GeoSERVER, GIS"
tags: postgreSQL
---

# 요약

첫 포스팅입니다. 주제는 _대중교통과 자가용의 최단 경로 찾기_ 입니다.
서울시를 배경으로 대중교통이나 자가용을 이용한 경로탐색 지도를 만드는 과정을 설명합니다.
최대한 깔끔[^1]하게 설명하고자 노력했습니다.

포스팅 내용은 크게 두 가지로 나뉩니다. **도로망을 이용한 자가용 경로 탐색**과 **대중교통 API를 이용한 대중교통 경로 탐색**입니다.
도로망을 이용한 자가용 경로 탐색은 ITS의 Node, Link데이터를 기반한 도로망과 pgRouting의 dijkstra으로 구현합니다.
자가용에서 pgRouting을 이용하는 부분은 [pgRouting의 예제 문서](https://workshop.pgrouting.org/2.0.2/en/index.html)를 매우 많이 참고하였습니다.

대중교통 API를 이용한 대중교토 경로 탐색은 API를 이용하여 대중교통 망과 운행시간표를 취득하고, [RAPTOR(Round-Based Public Transit Routing)](https://www.microsoft.com/en-us/research/wp-content/uploads/2012/01/raptor_alenex.pdf)로 경로를 탐색합니다.
대중교통 파트의 소스는 대부분 제가 작성하여 이 포스트에 첨부하였습니다. 대중교통 경로탐색 알고리즘인 RAPTOR는 해당 문서를 참고하여 구현하였습니다.

툴과 데이터는 모두 오픈 소스를 사용합니다. 처음에는 제가 다시 보기 편한 정도로 작성하려고 했었는데, 혹시나 이런 과정이 필요한 사람이 있을까 싶어 조금 더 자세하게 설명합니다.

## 환경

### 개발 도구

다음과 같은 개발 도구가 사용됩니다. 이 포스트에서는 윈도우 환경(Window 10)을 기준으로 작성합니다. 도구 뒤에 작성된 버전은 제가 사용한 버전을 나타냅니다.

* Visual Studio Code: 코드 작성 도구
* [Python >= 3.7.3](https://www.python.org/downloads/): API로 데이터 획득
* [PostgreSQL >= 11](https://www.postgresql.org/download/): 데이터베이스 구축 및 경로 탐색 알고리즘 구축
* [QGIS >= 3.6.1](https://qgis.org/ko/site/forusers/download.html): 데이터 확인
* [GeoSERVER >= 2.15.0](http://geoserver.org/): 데이터베이스의 데이터를 지도에 가시화하기 위한 서버

### 데이터

이 포스팅은 다음의 데이터를 활용합니다. 데이터는 문서 작성 시점을 기준으로 가져왔습니다.
따라서 게시된 내용과 실제 데이터의 차이가 존재할 수 있습니다.

* [ITS 노드 링크 데이터](http://nodelink.its.go.kr/data/data01.aspx)
* [서울시 버스 API](https://www.data.go.kr/dataset/15000193/openapi.do?mypageFlag=Y)
* [서울메트로 지하철 API](https://www.data.go.kr/dataset/15015807/openapi.do)

### 참고 문서

이 포스팅을 쓰면서 참고한 문서들입니다.

## 주석

[^1]: 정말 이게 최선입니다!