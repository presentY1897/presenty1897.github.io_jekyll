---
layout: post
title: "ITS node-link 데이터를 이용한 PostgreSQL 기반 경로 탐색 지도 만들기"
description: "PostgreSQL은 기본적인 경로탐색 알고리즘을 포함하는 pgRouting을 제공합니다. ITS에서 제공하는 전국의 도로망 데이터를 이용해 경로를 찾는 웹페이지를 구축하여봤습니다. ㅎㅎ"
comments: true
keywords: "postgreSQL, pgRouting, ITS, GeoSERVER, GIS"
tags: postgreSQL
---

# 요약

와! 첫 포스팅입니다. 정말 기쁘네요. 정말로요.

대망의 첫 주제는 _대중교통과 자가용의 최단 경로 찾기_ 입니다. ~~짝짝짝~~
대학원에서 대중교통 네크워크에서 최단 경로를 찾는 비스무리한 연구들을 해왔고, 회사에 취직하고서 비슷한 작업을 이어가게 되었습니다.
그래서 새로 해야하는 부분과 해왔던 부분을 최대한 깔끔[^1]하게 정리하고자 합니다.

이 포스팅의 전체 주제는 서울시를 공간적 배경으로 대중교통(버스, 지하철)과 자가용을 이용한 최단경로를 찾는 기능을 구현하는 과정을 설명합니다.
웹 서버 구축에서 길찾기 알고리즘, 웹페이지까지 포함하고 있습니다.
간단하게 비유하면 네이버 맵, 카카오 지도, 구글 맵에서 길찾기 기능 하위 호완 버전이죠.
툴과 데이터는 모두 오픈 소스를 사용합니다. 즉 누구나 따라할 수 있다는 말이죠. ~~그리고 아무도 책임지지 않는 다는 말이기도 합니다. 저를 포함해서요~~

## 환경

### 개발 도구

다음과 같은 개발 도구가 사용됩니다. 이 포스트에서는 윈도우 환경(Window 10)을 기준으로 작성한 점을 참고하시기 바랍니다.
~~어차피 리눅스 쓰시는 분들은 알아서 잘 활용 하실테니까 문제가 없을 것 같습니다~~ 도구 뒤에 작성된 버전은 제가 사용한 버전을 나타냅니다.

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