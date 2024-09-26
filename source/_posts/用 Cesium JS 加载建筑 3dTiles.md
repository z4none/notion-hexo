---
updated: '2019-08-29 08:00:00'
categories: code
excerpt: "随着现代浏览器的普及，在 WebGIS 应用中，越来越多的场合需要对城市建筑模型进行展示，目前较流行的解决方案有：\n1. Cesium 的 3DTiles\n2. Mapbox-gl 的 vector source 根据 height 拉伸\n3. ArcGIS\n本文讨论 Cesium 的 3DTiles，什么是 3DTiles\_https://github.com/AnalyticalGraphicsInc/3d-tiles/tree/master/specification"
date: '2019-08-29 08:00:00'
tags:
  - Windows
  - C++
urlname: cesium-load-3dtiles
title: 用 Cesium JS 加载建筑 3dTiles
---

随着现代浏览器的普及，在 WebGIS 应用中，越来越多的场合需要对城市建筑模型进行展示，目前较流行的解决方案有：

1. Cesium 的 3DTiles
2. Mapbox-gl 的 vector source 根据 height 拉伸
3. ArcGIS

本文讨论 Cesium 的 3DTiles，什么是 3DTiles [https://github.com/AnalyticalGraphicsInc/3d-tiles/tree/master/specification](https://github.com/AnalyticalGraphicsInc/3d-tiles/tree/master/specification)


## 建筑数据来源


### shapefile


相关行业一般采用 [shapefile](https://www.esri.com/library/whitepapers/pdfs/shapefile.pdf) 记录建筑信息，shape 文件中用多边形表示建筑轮廓，附加信息中一般有楼层字段可以得到建筑高度。shape 文件可以用 QGIS 进行查看和编辑。


### OSM


[openstreetmap](https://wiki.openstreetmap.org/wiki/Simple_3D_buildings) 是一个开放地图平台，托管着公开的 GIS 数据，包括建筑，国内的建筑数据相对不全，它有在线编辑器可以自己对建筑信息进行描绘和录入（公开）。与 shape 类似，建筑数据保存的是轮廓和高度（可以通过属性定义建筑屋顶形状，不过目前的数据中基本没有这个信息），相关数据可导出成 OSM 格式。 [https://wiki.openstreetmap.org/wiki/OSM_XML](https://wiki.openstreetmap.org/wiki/OSM_XML)


## 数据转换


### shape 转 3DTiles 的方法

- 使用 [3dtiles](https://github.com/fanvanzh/3dtiles) 这是个命令行工具，转换时需要指定高度字段，单位为 m，所以需要将 shape 文件预处理，通过楼层得到高度，转换后的数据没有进行 LOD 处理，渲染大量数据性能差。
- 使用 Cesium ion Cesium ion 是 Cesium 提供的在线数 GIS 据平台，用户可以将自己的数据在线托管，平台提供了 3D 模型转 3DTiles 的功能，操作步骤如下
	- 先将 shape 文件转换成 obj
	- 打包 obj 上传到 Cesium ion 平台，转换成 3DTiles
	- 通过脚本将 3DTiles 下载到本地 这种方法得到的 3DTiles 经过了 LOD 处理，渲染性能好，缺点是不能对 LOD 过程参数进行自定义。

## 数据加载


[https://zouyaoji.top/vue-cesium/#/zh/primitives/cesium-3dtileset](https://zouyaoji.top/vue-cesium/#/zh/primitives/cesium-3dtileset) [https://cesiumjs.org/Cesium/Build/Documentation/Cesium3DTileset.html?classFilter=Cesium3DTileset](https://cesiumjs.org/Cesium/Build/Documentation/Cesium3DTileset.html?classFilter=Cesium3DTileset)


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/1c25912c-9737-4bac-962c-ef14d02789bc/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T042920Z&X-Amz-Expires=3600&X-Amz-Signature=48959005856dabc1cc6960f4776593300e153907ea2277b2ef95d2c635b54d0d&X-Amz-SignedHeaders=host&x-id=GetObject)

