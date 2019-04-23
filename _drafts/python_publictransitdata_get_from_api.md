---
layout: post
title: "Python 기반으로 버스 및 지하철 데이터 가져오기"
description: ""
comments: true
keywords: "python, node.js, topis, restful"
---

## 1. 데이터

* api [주소](https://www.data.go.kr/dataset/15000193/openapi.do?mypageFlag=Y)
* 지하철 데이터 [api](http://data.seoul.go.kr/dataList/datasetView.do?infId=OA-101&srvType=A&serviceKind=1)
* 지하철 데이터 [종합](https://www.data.go.kr/dataset/15015807/openapi.do)

## 2. js call api

* 5 가지 [방법](https://www.twilio.com/blog/2017/08/http-requests-in-node-js.html)
* js xml2js [link](https://www.npmjs.com/package/xml-js)
* json insert to sql [link](https://www.opentechguides.com/how-to/article/nodejs/124/express-mysql-json.html)
* node js to psql [link](https://node-postgres.com/)
 
* js callback 함수 [이해하기](https://yubylab.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-%EC%BD%9C%EB%B0%B1%ED%95%A8%EC%88%98-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

node js를 기반으로 restful api 데이터 가져오기는 실패.
비동기적으로 함수가 실행되어 되어 천여개의 경로의 점 좌표를 가져오는 방법을 구현하지 못함.

## 3. Python call api data

* Bulk Insert Psql [링크](https://brownbears.tistory.com/297)
* BUlk Insert Stackoverflow [질답](https://stackoverflow.com/questions/2271787/psycopg2-postgresql-python-fastest-way-to-bulk-insert)

## 4. PostgreSQL PostGIS Functions

* 기본 [문서](https://postgis.net/docs/PostGIS_Special_Functions_Index.html#NewFunctions)
* postgreSQL insert perforamance [link](https://stackoverflow.com/questions/12206600/how-to-speed-up-insertion-performance-in-postgresql)