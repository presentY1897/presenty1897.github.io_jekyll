---
layout: post
title: "ITS 데이터 가져오기"
description: "ITS 데이터 가져오기"
comments: true
keywords: "postgreSQL, pgRouting, ITS, GeoSERVER, GIS"
tags: postgreSQL
---

# ITS 데이터 가져오기

처음으로 할 일은 ITS 데이터를 가져와 데이터베이스에 넣는 일입니다.
우선은 [ITS 표준노드링크 사이트][ITS데이터]에서 가장 최신의 전국표준노드링크 데이터를 받습니다.
*제가 받은 데이터는 19년 1월 10일자 전국표준노드링크입니다.*
ITS 표준노드링크는 주기적으로 에러 데이터를 갱신하니 최신 데이터가 가장 좋습니다.

![ITS 데이터 구조](/assets/postimages/PublicTransitRouting/0_0_ITS_.png){: width="100% height ="100%}
*다운로드 받은 압축 파일은 위와 같은 형태입니다.*  

ITS 노드 링크 데이터는 당연하겠지만, 노드와 링크로 구성되어있습니다.
ITS에서 정의한 노드[^1]와 링크[^2]는 해당 사이트에서 확인할 수 있습니다.
단순하게 노드는 교차로, 링크는 도로와 같은 유형으로 이해하셔도 됩니다.
여기서 링크 데이터(MOCT_LINK)를 사용합니다.

*압축파일에서 MOCT_LINK 파일들만 해제하셔도 됩니다.*  

파일의 기본 좌표계(ESPG:102082)를 이용하면 정확한 원인~~아마 윈도우 OS 혹은 postgreSQL이 해당 좌표계를 지원하지 않는 것으로 보입니다~~은 모르겠으나 postgreSQL에 정상적으로 입력되지 않습니다.
따라서 QGIS를 이용하여 좌표계를 WGS84(ESPG:4326)로 변경합니다.

## PostgreSQL 설치

이제 ITS MOCT_LINK 데이터를 넣기 위해서 postgreSQL을 설치합니다.
설치는 해당 사이트에서 파일을 받아 순서대로 클릭만 하면 됩니다.
딱 하나 PostGIS 관련 확장기능을 설치해야하는 점을 주의하세요.(stack builder에서 postgis extension 관련 기능을 설치해주세요.)

![Stack Builder Check](/assets/postimages/PublicTransitRouting/0_1_stack_builder.png){: width="100% height ="100% .center}
*PostGIS에 포함된 pgRouting 기능을 이용하여 경로 탐색을 수행할 겁니다.*

## PostgreSQL 데이터베이스 생성

PostgreSQL의 설치가 성공적으로 마무리 됬다면, 이제 LINK를 넣을 데이터베이스를 구축합니다.

![create database](/assets/postimages/PublicTransitRouting/0_2_shell_createdatabase.png){: .center}
*PostgreSQL Shell에서 생성한 예시*

```sql
CREATE DATABASE ITS_SAMPLE;
```

이어 추가 기능으로 postgis와 pgrouting을 새로 만든 DB인 ITS_SAMPLE에 추가해줍니다.
확장기능을 설치하기 위해서는 해당 데이터베이스로 접근을 해야합니다.
따라서 **\c its_sample**을 입력하여 새로 만든 데이터베이스에 접속합니다.
또 앞서 설치한 postgreSQL의 확장 기능들을 만든 데이터베이스에 설치해줍니다.

![create extension](/assets/postimages/PublicTransitRouting/0_3_shell_createextension.png){: .center}

```console
\c its_sample
```

```sql
CREATE EXTENSION postgis;
CREATE EXTENSION pgrouting;
```

## 데이터 집어넣기

데이터는 앞서 postgreSQL에서 확장 기능으로 설치한 postGIS를 활용합니다. PostGIS에는 shp파일을 읽어서 sql 입력 문으로 변환해주는
shp2pgsql 기능을 지원합니다. 사용방법은 postgreSQL이 설치된 폴더의 bin에서 cmd로 명령하거나, 환경변수를 변환하여 shp2pgsql을 사용하면 됩니다.

```console
shp2pgsql -c -s 4326 -I file tableName > link.sql
psql -U username -d its_sample -f link.sql
```

혹은

```console
shp2pgsql -c -s 4326 -I file tableName || psql -U username -d its_sample
```

shp2pgsql의 각 추가 속성에 대한 설명은 [postGIS 문서](https://postgis.net/docs/using_postgis_dbmanagement.html#shp2pgsql_usage)에서 설명되어 있습니다(또 [shp2pgsql 이용해서 shp파일을 postgresql db에 얹혀보기](https://zipeya.tistory.com/entry/shp2pgsql-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-shp%ED%8C%8C%EC%9D%BC%EC%9D%84-postgresql-db%EC%97%90-%EC%96%B9%ED%98%80%EB%B3%B4%EA%B8%B0)에서도 잘 설명하고 있습니다). 예시에서 사용한 옵션과 변수는 다음과 같습니다.
마지막의 '> link.sql'은 산출물을 link.sql으로 저장하라는 의미입니다.

|이름|설명|
|-------|-------|
| -c | 새로운 데이터베이스를 생성, 이 옵션이 기본 |
| -s | geometry의 경위도좌표계를 뒤에 오는 숫자(SRID)로 정의 |
| -I | geometry column에 인덱스 생성 |
| **file** | 변환할 파일 명 (cmd의 커서 위치에 따라 작성) |
| **tableName** | 입력할 테이블 명 |
| ---- | ---- |

~~-D 벌크 입력시 오류가 발생하는 데 이유는 아직 찾지 못했습니다.~~

## 데이터 확인

최종적으로 데이터의 입력이 문제 없이 이루어졌다면, shell이나 pgadmin을 통해서 입력한 데이터를 확인할 수 있습니다.

```console
\dt
```

![shell table check](/assets/postimages/PublicTransitRouting/0_4_shell_data_check.png){: .center}

link 테이블이 새로 생성되었습니다.

```console
\d link
```

![shell data check](/assets/postimages/PublicTransitRouting/0_5_shell_data_check.png){: .center}

link 테이블의 속성을 확인하면 위와 같이 나타납니다.

```sql
SELECT COUNT(*) FROM public.link;
```

![shell row count check](/assets/postimages/PublicTransitRouting/0_6_shell_data_check.png){: .center}

만약에 동일한 시점의 데이터를 받으셨다면 총 45663개의 link가 들어가 있을 겁니다.

## link 데이터 전처리

다만 입력한 데이터를 바로 pgRouting의 경로 탐색 함수에 적용하긴 어렵습니다.
pgRouting에는 dijkstra와 a*를 비롯한 여러 경로 탐색 알고리즘들이 구현되어 있습니다.
이 함수들은 공통적으로 graph를 이용하고 있는데, link 테이블을 graph에 맞춰 변환할 필요가 있습니다.

만약에 지금까지 다루던 link와 달리 vertex의 양 끝과 노드의 연결에 관한 데이터가 없이 단순히 형상만 존재한다면, pgRouting의 [pgr_createTopology](https://docs.pgrouting.org/2.0/en/src/common/doc/functions/create_topology.html)(혹은 [Create a Network Topology](https://workshop.pgrouting.org/0.6.1/en/chapters/topology.html)를 참고하세요)가 필요합니다.
하지만 link에는 링크의 시작 노드와 끝 노드에 관한 정보가 있으니 이에 관한 인덱스와 길이 열만 변환해줍니다.

```sql
ALTER TABLE link ALTER COLUMN;
```

## 주석

[^1]: 차량이 도로를 주행함에 있어 속도의 변화가 발생되는 곳을 표현한 곳(교차로, 교량시종점, 고가도로시종점, 도로의 시종점, 지하차도시종점, 터널시종점, 행정경계, IC/JC)
[^2]: 속도변화 발생점이 노드와 노드를 연결한 선을 의미하며 실세계에서의 도로(도로, 교량, 고가도로, 지하차도, 터널 등등)

[ITS데이터]:http://nodelink.its.go.kr/data/data01.aspx