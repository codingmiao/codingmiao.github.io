---
layout: post
title:  "Tomcat启动报错ZipException: error in opening zip file及解决"
categories:
- java
- 异常
tags:  奇葩问题
author: liuyu
date: 2017/11/30 20:46:25
---

今天早上到公司，照例从git更新了一下项目，然后开始干活，发现tomcat起不来了，报错如下:

```
java.util.zip.ZipException: error in opening zip file
```

打开zip文件出错？？然而项目里没有什么zip文件啊。。于是，我猜想，项目里，能用zip工具打开的，只有jar包了吧。

ok，删掉WEB-INF/lib下所有的jar，tomcat能启动了(当然，此时项目由于缺少依赖的jar无法正常启动)。

emmmmm，这下问题找到了，应该是一个或几个jar包坏掉了，接下来就是找到坏掉的jar，可是项目好大啊，上百个jar包怎么排查呢？当然是二分法查找了，时间复杂度只有O(logn)根本不虚好么2333 :dog: 。通俗点说就是先删掉一半jar包，如果报错找不到某个类，说明有问题的jar在删掉的那一半jar里，然后从删掉那一半里再缩小范围。

数分钟后，找到了两个jar，果然用zip工具打不开-_-!，maven仓库里找到它们，删掉对应的文件夹，重新下载，问题解决~ :hatched_chick: 

最后，吐槽一下这两个jar是怎么坏掉的，我司安全管理较严格，上网需要在电脑上插一个usb key，没这个key就会把你访问的网址转到一个"你尚未通过安全校验，请先巴拉巴拉"的一个页面。

然后悲剧就这样发生了，大概是上午更新完代码后，maven正在下载依赖时，我不小心碰到了usb key导致key断开了 :eyes: ，然后，下载下来的pom就变成这样了：
![233][1]

整个人都不好了啊喂-_-!

<<<<<<< Updated upstream
本文发布于[http://blog.wowtools.org/2017/11/30/zip-exception][2],转载请注明出处。


  [1]: http://7xlvcv.com1.z0.glb.clouddn.com/5f94f725-4061-48b8-bb5d-25aa1e932967
  [2]: http://blog.wowtools.org/2017/11/30/zip-exception/
=======


  [1]: http://7xlvcv.com1.z0.glb.clouddn.com/5f94f725-4061-48b8-bb5d-25aa1e932967
>>>>>>> Stashed changes
