---
layout: post
title: "pgRouting으로 경로 찾기"
description: "Data.go.kr에서 버스 데이터 가져오기"
comments: true
keywords: "postgreSQL, pgRouting, ITS, GeoSERVER, GIS"
tags: postgreSQL
---

# 버스 API

서울시, 정확히는 [TOPIS]()는 서울 안에서 움직이는 대부분의 버스 관련 데이터를 api로 제공하고 있습니다. 이 데이터를 가져와 경로 탐색에 활용해봅시다.

## api 얻기

api는 [서울시]와 [정부]에서 얻을 수 있습니다. 신청 방법은 가입 후에 해당 데이터를 신청하여 얻을 수 있습니다.

## API 구조

버스 API는 크게 버스 운행 정보, 버스 노선 정보로 구성되어 있습니다. 버스 운행 정보는 실시간으로 운영되는 버스 정보를 제공합니다(예를 들어 정류장까지 n번 버스가 도착하기까지 남은 시간). 버스 노선 정보는 현재 구축된 버스 노선과 그에 관련된 여러 정보를 제공합니다(예를 들어 버스 노선이 지나가는 정류장).

우선은 버스 노선에 관한 정보로 버스 정류장과 노선, 버스 형상정보를 얻어옵니다.

## Python을 이용한 API 취득

버스 API는 Restful입니다. Python에서 request를 이용해 데이터를 취득합니다.

### database

postgreSQL에는 데이터를 집어넣을 세 가지 테이블을 만듭니다.
우선은 버스 API에서 제공하는 모든 데이터를 가져옵니다.

```sql
```

### python

코드는 크게 데이터를 가져오는 부분과 postgreSQL에 집어넣는 부분으로 이뤄집니다.

```python
```

### 실행 결과


### 데이터 수정

