---
layout:     post
title:      静态地图后端渲染(mapbox-gl-native)
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:

    - 矢量切片
    - GIS

---

在Leaflet应用程序中，我们可以使用leaflet-image和一堆其他组件如leaflet.draw、html2canvas、FileSaver等，将地图渲染为前端生成的图像。但是，这种方法非常脆弱，体现于无法支撑高分辨率的、大范围的切片需要；并且由于仅限于一部分现代浏览器。

对于在线制图这个新项目而言，Mapbox提供的[静态API](https://docs.mapbox.com/help/how-mapbox-works/static-maps/)在一定程度上能够满足我们的需求。

![Mapbox制图](https://docs.mapbox.com/help/img/interactive-tools/static-api-playground.png)


>与leaflet一致，我们仍然需要的是一种在**后端使用和渲染这些图块并输出地图图像的简便方法，具体的需要体现为根据传入的动态样式，以及提供的制图参数进行制图。**

# Mapbox GL Native NodeJS API

`[Mapbox GL Native NodeJS API](https://github.com/mapbox/mapbox-gl-native/tree/master/platform/node)`是Mapbox提供的nodejs端的渲染呈现引擎，其依赖于linux或mac操作系统，对外提供了用于生成地图图像的轻量级NodeJS API。

## 编译

依赖包括：

- 64 bit macOS or 64 bit Linux
- Node.js version 10.x
- C++ 14

linux环境下的软件依赖相当简单，自行百度安装即可。

运行：

```
mkdir myMapbox
npm init
npm install @mapbox/mapbox-gl-native --save
npm install sharp --save
```

在安装完`@mapbox/mapbox-gl-native`和`sharp`之后即可使用相关js库。这里需要注意mapbox-gl-native依赖于亚马逊云上的相关依赖库，如果直接使用`npm install`，可能存在下载速度极慢的可能；即使使用了`cnpm install`的淘宝镜像仍然是同样的效果，同时`cnpm install`在安装时总是会出现奇奇怪怪的错误，因此推荐使用`npm`或`[yarn](https://yarn.bootcss.com/docs/)`进行依赖库的安装。

## Linux配置v2ray翻墙

针对下载速度极慢的情况，我们需要相关翻墙软件，以加快下载速度。这里使用到的工具包括

- [v2ray] (https://github.com/v2ray/v2ray-core)
- [Qv2ray](https://qv2ray.net/)

step1:下载v2ray与Qv2ray，其中v2ray为压缩包的形式，对其进行解压。

step2:为Qv2ray可执行程序赋予执行权限 `sudo chmod +x Qv2ray.v2.6.3.linux-x64.AppImage`。

step3：配置v2ray环境：

在Qv2ray的**内核设置**里面添加v2ray的路径

![v2ray环境配置](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/peizhihuanjing.png)

step4：添加订阅

将自己的订阅地址添加到对应位置中

![添加订阅](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/tianjiadingyue.png)

此时选择对应的代理服务器，即可翻墙访问外网。

## 使用

在官网的demo案例核心代码如下：

```javascript
var fs = require('fs');
var path = require('path');
var mbgl = require('@mapbox/mapbox-gl-native');
var sharp = require('sharp');

var options = {
  request: function(req, callback) {
    fs.readFile(path.join(__dirname, 'test', req.url), function(err, data) {
      callback(err, { data: data });
    });
  },
  ratio: 1
};

var map = new mbgl.Map(options);

map.load(require('./test/fixtures/style.json'));

map.render({zoom: 0}, function(err, buffer) {
   ......
});
```

其中style文件如下：

```json
{
  "version": 8,
  "name": "Empty",
  "sources": {
    "mapbox": {
      "type": "vector",
      "maxzoom": 15,
      "tiles": [
        "./fixtures/tiles/{z}-{x}-{y}.vector.pbf"
      ]
    }
  },
  "layers": [
     ......
    }
  ]
}
```

从`options`的定义中不难看出其使用的情景更多是为了满足私有应用程序的，及将style中定义的本地资源加载到Mapbox引擎中。


## 转化

作为在线制图平台的后端地图渲染支撑，其将相关数据存储在本地，然后进行加载、渲染，是不可行的，因此其相关数据更多的是以服务在私人服务端或mapbox服务端存在。远程服务包括两类：

- 托管在Mapbox的服务。
- 出于隐私和成本的考虑，托管在个人服务器上的服务。

为了满足当前的应用需要，需要将**远程服务**映射为**本地资源**，进行代码的改进:

首先对`options`中的req判断是否是mapbox的请求，

```javascript
const isMapboxURL = (url) => url.startsWith('mapbox://')
```

如果是`mapbox://`URL请求，需要根据其不同的类型(source,tile,glyph,sprite image,sprite json,image source)将其标准化为对应的可访问的Web请求。如`mapbox://mapbox.mapbox-streets-v7`转化为`https://api.mapbox.com/v4/mapbox.mapbox-streets-v7.json?secure=true&access_token= <您的令牌>`，其转换代码如下：

```javascript
const normalizeMapboxSourceURL = (url, token) => {
  const urlObject = URL.parse(url)
  urlObject.query = urlObject.query || {}
  urlObject.pathname = `/v4/${url.split('mapbox://')[1]}.json`
  urlObject.protocol = 'https'
  urlObject.host = 'api.mapbox.com'
  urlObject.query.secure = true
  urlObject.query.access_token = token
  return URL.format(urlObject)
}
```

如果是非mapbox请求，则直接进行http请求，并将其数据流导出即可。

```
const getRemoteAsset = (url, callback) => {
  webRequest(
    {
      url,
      encoding: null,
      gzip: true,
    },
    (err, res, data) => {
      if (err) {
        return callback(err)
      }
      return callback(null,{data})
    }
  )
}
```

## 总结

在此项目时遇到的最大挑战是，每次都需要从源代码重建Mapbox GL Native Node库。请严格参考上文的编译内容，那么请尽情享用吧！


## 附录index.js

```javascript
var sharp = require('sharp');
var mbgl = require('@mapbox/mapbox-gl-native');
var URL = require('url');
var webRequest =require('request');

const token = "pk.eyJ1IjoibWFwYng2MjUyIiwiYSI6ImNrZGR0dzA1ODExb3IzMXBldG5lNng2ODYifQ.IJ6-8xmQQhV-lriC_QDPzA"

const isMapboxURL = (url) => url.startsWith('mapbox://')

const normalizeMapboxSourceURL = (url, token) => {
  const urlObject = URL.parse(url)
  urlObject.query = urlObject.query || {}
  urlObject.pathname = `/v4/${url.split('mapbox://')[1]}.json`
  urlObject.protocol = 'https'
  urlObject.host = 'api.mapbox.com'
  urlObject.query.secure = true
  urlObject.query.access_token = token
  return URL.format(urlObject)
}

const normalizeMapboxTileURL = (url, token) => {
  const urlObject = URL.parse(url)
  urlObject.query = urlObject.query || {}
  urlObject.pathname = `/v4${urlObject.path}`
  urlObject.protocol = 'https'
  urlObject.host = 'a.tiles.mapbox.com'
  urlObject.query.access_token = token
  return URL.format(urlObject)
}

const normalizeMapboxStyleURL = (url, token) => {
  const urlObject = URL.parse(url)
  urlObject.query = {
    access_token: token,
    secure: true,
  }
  urlObject.pathname = `styles/v1${urlObject.path}`
  urlObject.protocol = 'https'
  urlObject.host = 'api.mapbox.com'
  return URL.format(urlObject)
}

const normalizeMapboxSpriteURL = (url, token) => {
  const extMatch = /(\.png|\.json)$/g.exec(url)
  const ratioMatch = /(@\d+x)\./g.exec(url)
  const trimIndex = Math.min(
    ratioMatch != null ? ratioMatch.index : Infinity,
    extMatch.index
  )
  const urlObject = URL.parse(url.substring(0, trimIndex))

  const extPart = extMatch[1]
  const ratioPart = ratioMatch != null ? ratioMatch[1] : ''
  urlObject.query = urlObject.query || {}
  urlObject.query.access_token = token
  urlObject.pathname = `/styles/v1${urlObject.path}/sprite${ratioPart}${extPart}`
  urlObject.protocol = 'https'
  urlObject.host = 'api.mapbox.com'
  return URL.format(urlObject)
}

const normalizeMapboxGlyphURL = (url, token) => {
  const urlObject = URL.parse(url)
  urlObject.query = urlObject.query || {}
  urlObject.query.access_token = token
  urlObject.pathname = `/fonts/v1${urlObject.path}`
  urlObject.protocol = 'https'
  urlObject.host = 'api.mapbox.com'
  return URL.format(urlObject)
}

const getRemoteTile = (url, callback) => {
  webRequest(
    {
      url,
      encoding: null,
      gzip: true,
    },
    (err, res, data) => {
      if (err) {
        return callback(err)
      }

      switch (res.statusCode) {
        case 200: {
          return callback(null, { data })
        }
        case 204: {
          // No data for this url
          return callback(null, {})
        }
        case 404: {
          // Tile not found
          // this may be valid for some tilesets that have partial coverage
          // on servers that do not return blank tiles in these areas.
          console.warn(`Missing tile at: ${url}`)
          return callback(null, {})
        }
        default: {
          // assume error
          console.error(
            `Error with request for: ${url}\nstatus: ${res.statusCode}`
          )
          return callback(
            new Error(
              `Error with request for: ${url}\nstatus: ${res.statusCode}`
            )
          )
        }
      }
    }
  )
}

const getRemoteAsset = (url, callback) => {
  webRequest(
    {
      url,
      encoding: null,
      gzip: true,
    },
    (err, res, data) => {
      if (err) {
        return callback(err)
      }

      switch (res.statusCode) {
        case 200: {
          return callback(null, { data })
        }
        default: {
          // assume error
          console.error(
            `Error with request for: ${url}\nstatus: ${res.statusCode}`
          )
          return callback(
            new Error(
              `Error with request for: ${url}\nstatus: ${res.statusCode}`
            )
          )
        }
      }
    }
  )
}

const options = {
  request: (req, callback) => {
    const { url, kind } = req;

    const isMapBox = isMapboxURL(url)

    try {
      switch (kind) {
        case 2: {//source
          if (isMapBox) {
            getRemoteAsset(
              normalizeMapboxSourceURL(url,token),
              callback
            )
          } else {
            getRemoteAsset(url, callback)
          }
          break;
        }
        case 3: {//tile

          if (isMapBox) {
            getRemoteTile(
              normalizeMapboxTileURL(url,token),
              callback
            )
          } else {
            getRemoteTile(url, callback)
          }
          break;
        }
        case 4: {//glyph
          getRemoteAsset(
            isMapbox
              ? normalizeMapboxGlyphURL(url, token)
              : URL.parse(url),
            callback
          )
          break
        }
        case 5: {//sprite image
          getRemoteAsset(
            isMapbox
              ? normalizeMapboxSpriteURL(url, token)
              : URL.parse(url),
            callback
          )

          break
        }
        case 6: {//sprite json
          getRemoteAsset(
            isMapbox
              ? normalizeMapboxSpriteURL(url, token)
              : URL.parse(url),
            callback
          )
          break;
        }
        case 7: {
          // image source
          getRemoteAsset(URL.parse(url), callback)
          break
        }
        default: {
          throw new Error("no handler!!!")
        }
      }
    } catch (err) {
      console.error(err);
      callback(err);
    }
  },
  ratio: 1
};

var map = new mbgl.Map(options);

map.load(require('./style.json'));

map.render({
  zoom: 0,
  center: [0, 0],
}, (err, buffer) => {
  if (err) {
    console.log(err);
    throw err;
  }

  map.release();

  var image = sharp(buffer, {
    raw: {
      width: 512,
      height: 512,
      channels: 4
    }
  });

  // Convert raw image buffer to PNG
  image.toFile('image.png', function (err) {
    if (err) {
      console.log(err);
      throw err;
    }
  });
});
```
