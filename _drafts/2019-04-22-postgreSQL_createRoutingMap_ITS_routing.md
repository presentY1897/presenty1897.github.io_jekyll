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

QGIS에서 구축한 link 테이블을 연결하고, 필터에 다음 구문을 넣어 맞게 움직이는 지 봅시다.

```sql
"gid" IN ( SELECT id2 AS gid FROM pgr_dijkstra('
                SELECT gid AS id,
                         source::integer,
                         target::integer,
                         length::double precision AS cost
                        FROM link',
                30, 60, false, false) a LEFT JOIN link b ON (a.id2 = b.gid)
)
```
[!쿼리 입력 이미지]()
[!결과 이미지]()

다만, 해당 데이터는 실제로 완전하진 않습니다. 몇몇 오류 데이터들이 존재하고 있으며 이는 만약에 이 데이터를 이용해 정밀한 결과를 얻고자 한다면 문제가 될 수 있습니다. 이를 해결하는 방법은 나중에 포스팅하겠습니다. 우선은 pgRouting을 활용하는 데 집중합시다.

## GeoSERVER 구축

이제 만든 데이터를 웹에서 사용할 수 있게 GeoSERVER를 활용합니다. GeoSERVER는 지도 개체를 제공하는 서버를 구축할 수 있는 오픈 소스입니다.
설치는 순서에 맞춰서 진행하면 됩니다. 아래 소스는 [이 예제]()를 따라 수행하였고, 데이터에 맞춰 일부 수정하였습니다.

우선 GeoSERVER에 geometry를 보내줄 function을 제작합니다. 

```sql

```

~~이 함수는 두 점의 위치를 받아 각각 가장 가까운 노드를 찾아 해당 노드 간의 최단 경로를 반환합니다.~~

설치가 끝나면 start geoserver를 실행하여 geoserver를 실행하고, geoadmin을 실행하여 관리자 모드로 들어갑니다.

[!Start Geoserver]()

[!Geo Admin]()

기본 관리자 id는 admin, password는 geoserver입니다.

관리자로 접속하여, 저장소를 만듭니다.

이제 레이어를 생성합니다.

서버단은 마무리 되었습니다.

## html파일 생성

이제 간단한 웹페이지를 만들어 link에서 찾은 경로를 지도에 띄워봅니다.

js는 osm3를 이용합니다. 참고한 소스는 구버전을 기준으로 작성되어있어 실제로 작동하지 않는 부분이 많아 부분적으로 수정하였습니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>ol3 pgRouting client</title>
    <meta charset="utf-8">
    <link rel="stylesheet" href="https://cdn.rawgit.com/openlayers/openlayers.github.io/master/en/v5.3.0/css/ol.css" type="text/css">
    <style>
      #ol-map {
        width: 100%;
        height: 100%;
      }
    </style>
    <script src="https://cdn.rawgit.com/openlayers/openlayers.github.io/master/en/v5.3.0/build/ol.js"></script>
  </head>
  <body>
    <div id="ol-map">
        <div id="start-point">start</div>
        <div id="final-point">final</div>
    </div>

    <script>

        var map = new ol.Map({
            target: 'ol-map',
            layers: [
            new ol.layer.Tile({
                source: new ol.source.OSM()
            })
            ],
            view: new ol.View({
            center: [14134888.1446, 4517672.9700],
            zoom: 10
            })
        });

        var params = {
            LAYERS: 'pgrouting:pgrouting',
            'TILED': true,
            FROMAT: 'image/png'
        };
        var startPoint = new ol.Overlay({
            map: map,
            element: document.getElementById('start-point')
        });
        var finalPoint = new ol.Overlay({
            map: map,
            element: document.getElementById('final-point')
        });

        var transform = ol.proj.getTransform('EPSG:3857', 'EPSG:4326');

        map.on('click', function(event) {
            var coordinate = event.coordinate;
            if (startPoint.getPosition() == undefined) {
                // first click
                startPoint.setPosition(coordinate);
            } else if (finalPoint.getPosition() == undefined) {
                // second click
                finalPoint.setPosition(coordinate);

                // transform the coordinates from the map projection (EPSG:3857)
                // into the server projection (EPSG:4326)
                var startCoord = transform(startPoint.getPosition());
                var finalCoord = transform(finalPoint.getPosition());
                var viewparams = [
                'x1:' + startCoord[0], 'y1:' + startCoord[1],
                'x2:' + finalCoord[0], 'y2:' + finalCoord[1]
                ];
                params.viewparams = viewparams.join(';');

                // we now have the two points, create the result layer and add it to the map
                result = new ol.layer.Image({
                source: new ol.source.ImageWMS({
                    url: 'http://localhost:8080/geoserver/pgrouting/wms',
                    params: params,
                    serverType: 'geoserver'
                })
                });
                map.addLayer(result);
                startPoint = new ol.Overlay({
                    map: map,
                    element: document.getElementById('start-point')
                }); 
                finalPoint = new ol.Overlay({
                    map: map,
                    element: document.getElementById('final-point')
                });
            }
        });
    </script>
  </body>
</html>
```

이제 약간 느리지만 웹페이지에서 ITS Link를 따라 두 공간을 연결하는 최단 경로를 찾을 수 있습니다. :)

[!실행 예시]()

## 주석