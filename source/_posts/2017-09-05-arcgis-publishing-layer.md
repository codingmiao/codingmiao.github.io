---
layout: post
title:  "arcgis 发布图层"
categories:
- gis
- 工具和软件
tags:  arcgis
author: liuyu
date: 2017/09/05 20:46:25
---
好久没用arcgis了，发布图层的步骤老是忘-_-，简单记录一下吧

# 制作地图文档
## 添加数据

打开ArcMap，新建一个地图文档，右键点击”图层”->”添加数据”，选择需要加入的数据

![此处输入图片的描述][1]

若加入的是tif之类的文件，且需要去除黑边，可右键此图层->属性，勾选”显示背景值”，并设为无颜色:

![此处输入图片的描述][2]


## 修改坐标系

右键点击”图层”->”属性”，设置为需要的坐标系
![此处输入图片的描述][3]


最后将其存为mxd文档。


# 发布服务
## 预先配置一下arcgis server

打开ArcCatalog，在左侧目录树的"gis服务器"下选择"添加arcgis server"，然后根据自己arcgis server的连接信息，一路下一步，添加一个server。
然后打开arcserver的web控制台，大概路径像这样：
http://localhost:6080/arcgis/manager/index.html

我们需要配置"站点"选项卡下的几个东西：

### 缓存目录

在"目录"下，修改缓存目录的位置到一个剩余磁盘空间比较大的路径，也可以添加多个缓存目录，在后面的步骤里为不同的图层指定不同的缓存目录。
![此处输入图片的描述][4]

### Data Store里注册文件夹

做这步操作的原因是，即使你在本机上制作mxd/sd文档并发布到本机的arcgis server，arcserver仍然会把mxd用到的数据文件上传到arcserver的目录下，这就很蛋疼了。。
于是，我们需要把数据文件所在的文件夹注册进来，这样mxd用到这个文件夹中的文件时就不需要上传了。
![此处输入图片的描述][5]

## 发布地图文档

好了，现在可以开始发布刚才制作好的mxd了，在ArcMap中，点击"文件"->"共享为"->"服务"，看到如下界面:
![此处输入图片的描述][6]

选择"发布服务"的话，就能直接把mxd发成图层，"保存服务定义文件"则会生成一个sd文档。过程大同小异，一路下一步，设置一些常规信息，来到这一步：
![此处输入图片的描述][7]

特别说一下缓存这里

### 缓存配置

也就是瓦片地图切片。
![此处输入图片的描述][8]

缓存的高级设置下面，可以选择切片的缓存目录，能选择的目录就是前文中arcgis server里配置过的。
高级设置的高级按钮(谁起的名字-_-)里，可以设置切片的存储是松散型还是紧凑型，松散型切出来就是一张张图片，级别较高时图片数量会非常大，所以一般我们不太去用这种格式；紧凑型格式的解析可以参考[这个帖子][9]，需要注意的7是，低版本的arcserver切出来的bundle、bundlex文件是分开的。


好啦，配置完成，点一下分析按钮，没有错误的话点击发布，稍等片刻就文档就发布到arcserver上了。如果配置了切片，切片任务会在arcserver的后台执行，所以关掉arcmap、catalog也没问题。随后，可以在catalog里右键此图层->查看缓存状态->详细信息 来查看切片进度。
![此处输入图片的描述][10]


  [1]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-05-arcgis-publishing-layer-1
  [2]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-5-arcgis-publishing-layer-2
  [3]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-5-arcgis-publishing-layer-3
  [4]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-5-arcgis-publishing-layer-4
  [5]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-5-arcgis-publishing-layer-5
  [6]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-5-arcgis-publishing-layer-6
  [7]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-5-arcgis-publishing-layer-7
  [8]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-5-arcgis-publishing-layer-8
  [9]: http://www.cnblogs.com/ssai2015/p/4926323.html
  [10]: http://7xlvcv.com1.z0.glb.clouddn.com/2017-09-5-arcgis-publishing-layer-9
