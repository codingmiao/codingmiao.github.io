---
layout: post
title:  "geoserver gwc"
categories: gis
- gis
- 工具和软件
tags:  geoserver
author: liuyu
date: 2016/11/28 20:46:25
---

 **- 简介**

gwc，是“Geo Web Cache”的简称。它是一种带缓存的地图服务，但与TileLayer的工作机制又不太一样。我们先回顾一下两种常规的地图服务模式：

DynamicLayer：动态图，即每次接收到用户的出图请求后，根据用户传过来的地图范围，查询范围内的要素并绘制成图片返回给用户。由于每次出图请求都需要查询和绘制，效率一般不会很高。

TileLayer：根据指定的切图原点，切图范围等参数，预先将绘制好瓦片图并保存起来，当用户请求某个范围内的地图时，计算此范围覆盖了哪些瓦片并将这些切片返回给用户。由于瓦片是预先绘制好的，直接返回就行了，所以速度很快。TileLayer也有很多缺点，实时性不高，每次切图很耗时(几十天甚至几个月一点都不稀奇)，而且占用很大的磁盘空间，某级别下的某些区域用户可能根本不会去访问，这就白白浪费了切图的时间和磁盘。

gwc是结合DynamicLayer和TileLayer的一种折衷方案：先不做预切图(当然也可以做，这个在下面实际使用时再具体说明)，当用户发送某个范围的出图请求时，计算范围内需要哪些切片，然后去缓存文件夹中找，没找到的话先临时生成一个丢到缓存文件夹里，把这些切片作适当的缩放和裁剪，再拼起来返回给用户。


----------

 **- 如何使用**
 
首先，可以在web.xml文件里配置一下gwc切片目录：
>  &lt;context-param&gt;
   &lt;param-name&gt;GEOWEBCACHE_CACHE_DIR&lt;/param-name&gt;
   &lt;param-value&gt;你的geowebcache切片的目录&lt;/param-value&gt;
  &lt;/context-param&gt;

不配的话，tomcat里运行默认是存在tomcat的temp目录下。
 
 
 然后进入geoserver的管理界面，我们看到Tile Cacing下一共五个子目录：
**Tile LayersTile Layers**
gwc图层的配置、查看、预切图等
**Caching Defaults**
一些默认设置
**Gridsets**
切图网格的设置
**Disk Quota**
磁盘存储设置，比如设置最大磁盘占用50g，当缓存文件超过50g后，会按最近最少使用之类的规则清理掉一些旧的
**BlobStores**
允许我们指定存储位置、磁盘blocksize等，以更合理地使用磁盘

我们先在Gridsets里，按照所用的地图信息配置一个切图网格：
![此处输入图片的描述][2]
遗憾的是不能直接修改坐标原点，导致切出来的瓦片不能直接拷贝出来丢给arcgis之类的使用，当然这个可以通过自定义坐标系来解决。


选择需要gwc的图层，
![](http://7xlvcv.com1.z0.glb.clouddn.com/c4428e71-931d-4902-afe2-ecc84f4ea2eb)
  
好了，准备工作完成，下面可以开始使用了：
点击Tile Layers,Preview下拉框选择一个想要的图片格式和grid，就可以预览了(预览页面右键查看源代码，就有openlayers调用gwc的方法了^_^)。


如果想对指定区域进行预切片，只要点击Seed/Truncate，选好参数就可以开切了，不过geoserver切图性能堪忧，可以考虑用arcgis、gdal之类的切好了再丢到gwc缓存目录里。


 **- 关于resolution**
 
 比较一下gwc和wms请求图片的url：
 

    gwc:
    geoserver/gwc/service/wms?LAYERS=xxx&...&SRS=EPSG%3A3857&BBOX=xxx
    wms:
    geoserver/<workspace>/wms?LAYERS=xxx&...&SRS=EPSG%3A3857&BBOX=xxx
    
我们发现，gwc除了前缀url路径不一样，后面的请求参数和wms几乎是一样的，也就是说，gwc并不是像TileLayer那样直接把切片返回到客户端，而是需要通过bbox去计算需要哪些切片，然后做适当的拼接、裁剪、缩放。

如果同一个空间参考系配置了多个grid，geoserver会去计算请求图片的resolution与哪个grid更接近就用哪个grid。

如果请求图片的resolution与grid的resolution相差较大，会导致图片过大地缩放，然后图片质量就不好了，这是网上找的一张图：
![](http://7xlvcv.com1.z0.glb.clouddn.com/114be441-b6ae-4630-b0e5-b0f5446cf2a1)
  
 如果resolution相差再大一些，geoserver就直接抛个异常说resolution相差不能超过10%了。
 
 所以，在使用gwc时，请尽量按照客户端地图的resolution来配置一套grid，以获得最佳浏览效果。
