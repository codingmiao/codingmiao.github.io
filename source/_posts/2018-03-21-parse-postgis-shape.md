---
layout: post
title:  "postgis空间数据格式解析，及如何绑定变量"
categories:
- gis
- 服务端
tags:  PostgreSQL
author: liuyu
date: 2018/03/21 20:46:25
---
当我们往postgis的shape字段中插入一条数据
```
INSERT INTO test1 (shape)
VALUES (GeomFromEWKT('SRID=4326;POINT(116.39 39.9)'));
```
读出来的shape是这样的:
```
0101000020E6100000295C8FC2F5185D403333333333F34340
```
显然，这是一个wkb格式的字符串嘛

于是，在java中，我们只需用jts的WKBReader类，便可直接将其解析为geometry对象
```
WKBReader wkbReader = new WKBReader();
......此处略去jdbc的一堆套路
PGobject obj = (PGobject) rs.getObject("shape");
byte[] bytes = WKBReader.hexToBytes(obj.getValue());
Geometry geo = wkbReader.read(bytes);
System.out.println(geo);
```

我们发现，postgreSQL用PGobject来描述自定义数据类型，而空间数据的type是geometry，所以，我们可以这样绑定变量来做预解析：
```
        double[] bbox = new double[]{
                90, 20, 120, 30
        };
        Coordinate[] coords = new Coordinate[]{
                new Coordinate(bbox[0], bbox[1]),
                new Coordinate(bbox[2], bbox[1]),
                new Coordinate(bbox[2], bbox[3]),
                new Coordinate(bbox[0], bbox[3]),
                new Coordinate(bbox[0], bbox[1]),
        };
        LinearRing shell = gf.createLinearRing(coords);
        Geometry env = gf.createPolygon(shell);
        PGobject param = new PGobject();
        //sql及绑定变量这样写  "select * from test1 ? && shape", param
```
