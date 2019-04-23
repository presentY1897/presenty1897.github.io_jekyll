---
layout: post
title: "ITS node-link 데이터를 이용한 PostgreSQL 기반 경로 탐색 지도 만들기"
description: "PostgreSQL은 기본적인 경로탐색 알고리즘을 포함하는 pgRouting을 제공합니다. ITS에서 제공하는 전국의 도로망 데이터를 이용해 경로를 찾는 웹페이지를 구축하여봤습니다. ㅎㅎ"
comments: true
keywords: "postgreSQL, pgRouting, ITS, GeoSERVER, GIS"
tags: postgreSQL
---

# 요약

와! 첫 포스팅입니다.

처음이니 그나마 제가 자주 다루었던 주제에 관해 게시해보고자 합니다.

> 바로 _대중교통과 자가용의 최단 경로 찾기_ 입니다.

서울시를 배경으로 대중교통(버스, 지하철)과 자가용을 이용한 최단경로를 찾는 기능을 구현하는 과정을 설명합니다.
사용하는 툴과 데이터는 모두 오픈 소스를 사용합니다.

## 환경

### 개발 도구

다음과 같은 개발 도구가 사용됩니다. 이 포스트에서는 윈도우 환경(Window 10)을 기준으로 작성한 점을 참고하시기 바랍니다.
다른 운영체제 이용자는 Python이나 기타 오픈 소스 활용에 능숙하실테니 괜찮을 겁니다. 도구 뒤에 작성된 버전은 제가 사용한 버전을 나타냅니다.

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
