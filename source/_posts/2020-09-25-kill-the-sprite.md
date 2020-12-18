---
layout: post
title:  "干掉mapbox里的雪碧图(sprite)"
categories:
- gis
- 前端
tags:  mapbox
author: liuyu
date: 2020/09/25 20:46:25
---


# 什么是雪碧图

雪碧图源于前端的小图标展现技术，将小图标和背景图像合并到一张图片上，然后利用css的背景定位来显示需要显示的图片部分。
对于含有大量小图标的页面，雪碧图把多个图片请求合并成了一个，大幅提高了加载性能。
![雪碧图](/myimgs/20200925/1.jpg)

mapbox中也采用了类似的思想：当我们配置了多个symbol类型的图层时，为了减少图标的请求数，mapbox也提供了雪碧图的加载方式

```js
        var map = new mapboxgl.Map({
            container: 'map',
            style: {
                "version": 8,
                "sources": {},
                "layers": [],
                "sprite": baseUrl + "/assets/sprites/mysprites",//配置一个雪碧图的url路径
            },
            //...
        });
```
配置了sprite字段后，mapbox就会去加载雪碧图png和一个json配置：
```js
{
  "icon1": {
    "x": 324,
    "y": 308,
    "width": 20,
    "height": 11,
    "pixelRatio": 1,
    "sdf": false
  },
  "icon2": {
    "x": 221,
    "y": 464,
    "width": 15,
    "height": 15,
    "pixelRatio": 1,
    "sdf": false
  },
  //...
}
```
随后，我们就可以在图层中使用这个图标了:
```js
map.addLayer(
            {
                "id": "mylayer",
                "type": "symbol",
                "source": "mysource",
                "source-layer": "mylayersource",
                "layout": {
                    "icon-image": "icon1",
                }
            }

)
```

# 用雪碧图来配图标的缺点

想在雪碧图上改图标就不那么容易，尤其是我这种大老粗的程序员，还要买杯奶茶去劳烦美工妹子帮忙。
嗯，当然也可以写段代码去修改它，或者直接找一些相关的开源工具来解决。

然而，矢量瓦片的一大卖点就是用户可以在前端自由定制样式，要去修改图片这个操作一定程度上限制了矢量瓦片样式的灵活性。

同时，目前mapbox仅支持传入一张雪碧图，假如用户自己的图层也配了一份雪碧图，那还需要把两个雪碧图合起来，也就是说图层的自由组合也受到了限制、

# 用单个图标来替代雪碧图

## 单个图标的使用
基于上述问题，在更灵活的样式配置的场景下，使用单个图标比雪碧图整合要方便的多：
```js
let url = 'http://xxx.png'
let iconId = 'icon1'
map.loadImage(url, function (err, img) {
            if (map.hasImage(iconId)) {
                map.updateImage(iconId, img)
            } else {
                map.addImage(iconId, img)
            }
            map.addLayer({
                                         "id": "mylayer",
                                         "type": "symbol",
                                         "source": "mysource",
                                         "source-layer": "mylayersource",
                                         "layout": {
                                             "icon-image": iconId,
                                         }
            })
})
```
需要更换/新增图标时，只要改一下图标url之类的代码即可。

## 解决单个图标加载的性能问题
前面提到，使用雪碧图是为了减少图片请求数量从而提高加载性能，当图标数量很多时，我们传多个url去请求图片就不合适了。

幸好loadImage函数传入的url允许是一个base64字符串，例如:
```
data:image/png;base64,R0lGODlhHAAmAKIHA...f394uLiwAAAP===
```

于是，我们可以编写一个服务，批量传入图标id，批量返回图标base64字符串，从而将多个请求合并为一个
```
#请求url
http://xxx/getIcons?id=icon1,icon2

#返回数据
{
    icon1:'data:image/png;base64,R0lGODlhHAAmAKIHA...f394uLiwAAAP===',
    icon2:'yH5B…EoqQqJKT1TRk1V7S2xYJADs='
}

```
然后，再编写一段对应的前端加载函数即可：

```js
    /**
     * 从自定义服务批量加载图片
     * @param map
     * @param ids
     * @param cb
     */
    function loadBatch(map, ids, cb) {
        let uuids = '';
        for (let iconId in ids) {
            const uuid = ids[iconId];
            uuids += uuid + ','
        }
        $.ajax({
            type: 'POST',
            url: "http://xxx/getIcons",
            data: {ids: uuids},
            cache: true,
            dataType: 'json',
            success: function (json) {//获取数据
                const notFinds = [];
                const urls = {};
                for (let iconId in ids) {
                    const uuid = ids[iconId];
                    const base64 = json[uuid]
                    if (!base64) {
                        notFinds.push(iconId)
                    } else {
                        urls[iconId] = base64
                    }
                }
                loadFromUrls(map, urls, cb)
            }

        });
    }

    /**
     * 批量加载url图片
     * @param map
     * @param urls 请求urls {<id1>:<url1>,<id2>:<url2>,...}
     * @param cb 回调函数 {imgs:{},errs:{}}
     */
    function loadFromUrls(map, urls, cb) {
        const res = {
            imgs: {},
            errs: {}
        }
        let num = Object.keys(urls).length

        function doReturn() {
            num--;
            if (num == 0) {
                cb(res)
            }
        }

        function subCb(iconId, img) {
            res.imgs[iconId] = img;
            doReturn();
        }

        function subErrCb(iconId, err) {
            res.errs[iconId] = err;
            doReturn();
        }

        for (let iconId in urls) {
            const url = urls[iconId];
            loadFromUrl(map, iconId, url, subCb, subErrCb)
        }
    }

    /**
     * 加载url
     * @param map
     * @param iconId 图标id
     * @param url url 可以是base64url
     * @param cb 加载成功回调
     * @param errCb 加载错误回调
     */
    function loadFromUrl(map, iconId, url, cb, errCb) {
        map.loadImage(url, function (err, img) {
            if (err) {
                if (errCb) {
                    errCb(iconId, err)
                } else {
                    throw err
                }
            }
            if (map.hasImage(iconId)) {
                map.updateImage(iconId, img)
            } else {
                map.addImage(iconId, img)
            }
            cb(iconId, img)
        })
    }

```

# 总结
虽然当下的web开发讲究前后端分离，
但是从雪碧图的这个例子可以看出，作为一个giser，在主修前端or后端的一方时，还是要多去同时了解下前后端的知识，才能设计出更高效、灵活的方案。
