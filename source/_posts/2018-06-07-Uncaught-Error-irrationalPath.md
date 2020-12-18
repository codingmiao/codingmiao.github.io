---
layout: post
title:  "dojo报错Uncaught Error: irrationalPath原因及解决"
categories: 奇葩问题
tags:  dojo 
author: liuyu
date: 2018/06/07 20:46:25
---

最近写了一堆amd风格的js库供其他同事调用，有同事反映说用dojo.js调的时候会报这个错:
```
init.js:4 Uncaught Error: irrationalPath
    at s (init.js:4)
    at Ea (init.js:13)
    at Ta (init.js:14)
    at Ba (init.js:15)
    at hb (init.js:21)
    at init.js:21
    at c (init.js:4)
    at gb (init.js:21)
    at d (init.js:20)
    at HTMLScriptElement.<anonymous> (init.js:23)
```
排查了好久，终于找到了原因，一句话解释就是../不能调用配置根路径以上的内容。

详细还原一下出错过程，我提供的js库路径如下
```
xxx.com/
    root/
        a/
            a1.js
        b/
            b1.js
```
a1.js引用了b1.js:
```
define(['../b/b1'], function (b1) {
    b1.xxx()
```

同事是这样调用的
```
var dojoConfig = {
    parseOnLoad: true,
    packages: [
        {
            "name": "ext",
            "location": "xxx.com/root/a"
        }
    ]
};

require([
        "ext/a1"
    ],
    function (a1) {
        a1.ooxx();
    }
);

```
出于安全性考虑，dojo只允许调用根路径(xxx.com/root/a)下的资源,a.js中通过../去引用xxx.com/root/b/b.js，所以就出错了。

正确地姿势应该是这样:
```
var dojoConfig = {
    parseOnLoad: true,
    packages: [
        {
            "name": "ext",
            "location": "xxx.com/root"
        }
    ]
};

require([
        "ext/a/a1"
    ],
    function (a1) {
        a1.ooxx();
    }
);

```

完o(*￣︶￣*)o
