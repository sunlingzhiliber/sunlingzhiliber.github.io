---
layout:     post
title:      矢量切片地图
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:

    - 矢量切片
    - GIS

---

> 面对大规模矢量空间数据的高效可视化需求，在地图显示目前最大的、最关键的矛盾在于：地理空间上非匀质实体与屏幕空间上匀质像素之间的非对称映射矛盾。针对此，当前最为认可的解决方案是利用瓦片化的组织方式来对矢量数据进行组织和管理。


# 切片总体思路

瓦片化组织的基本思路是将地理空间按照不同的级别、规则的格网进行分割，每个瓦片都对应到确定的区域范围并拥有唯一的级别-行-列编码；在屏幕空间中将需要呈现的区域转换为相应的瓦片编码，利用该编码信息直接从矢量数据源中抽取所需呈现的数据。

## 预处理

1.根据要素的最大外包矩形。
2.对每个层级，将最大外包矩形转化为瓦片，再利用ST_Intersects判断每个瓦片是否和要素相交，判定为相关的瓦片。

## 切图

1.在每个层级对相关瓦片集合进行化简，针对不同的要素类型采用不同的化简策略。
  - 点要素，不做任何操作（后期可参考PostGIS的ST_Dump函数对其进行点位聚合操作）
  - 线要素与面要素，在兼顾视觉无误差认知的理解下，以当然的分辨率进行ST_SimplifyPreserveTopology化简，当然的分辨率计算方式为`2 * Math.PI * R / TILE_SIZE /Math.pow(2, level)`，对于mapbox而言，其切片大小tileSize为512，地球半径R为6378137.0。

```java
    private static final int[] RESULUTION = {
            78271,
            39135,
            19567,
            9783,
            4891,
            2445,
            1222
            ...
    };
```

2.以化简完的矢量数据，利用ST_AsMVT、ST_AsMVTGeom、ST_TileEnvelope、ST_Intersects进行切片，并将得到的数据以z/x/y.pbf的形势存储在本地。
  - 首先以transport_line中与切片Envelope相交的数据以基础，利用ST_AsMVTGeom将原始的geom数据转换为Mapbox vector tiles支持的MVTgeom，最终将其转为ST_AsMVT的二进制流。

```sql
select ST_AsMVT(p,'transport_line') as mvt from 
(
	select 
	st_asmvtgeom(geom,ST_TileEnvelope(3,6,3)) as geom 
	from transport_line  
	WHERE ST_Intersects(geom, ST_TileEnvelope(3,6,3))
)as P where p.geom is not null
```

# 切图效果评价

以国情全省大数据中的交通线为基础，进行切图效果评价。共计900W+条线数据，包括gid,geom,objectid,lane,width,elevt,areacode,changetype,shape_leng,name,drct,names,分类,rnp,rteg,ec,featid,elemstime,elemetime,ecp,type,主区,道路编号,cc,gb,changeatt,rn,sdtf,matrl等共计20个属性字段，其原始Shapfile大小为11.6GB。

以z=3,x=6,y=3位置的矢量切片为列，其pbf切片比较如表所示。

|          | 要素化简 | 要素不化简 |
|----------|----------|------------|
| 包含属性 | 70.12MB  | 72.51MB    |
| 去除属性 | 7.0622   | 7.27MB     |

发现简化似乎没有取得特别大的成果，这可能跟化简的阈值设定相关；同时发现在考虑属性数据的情况下，数据量陡增，因此需要对数据源的属性数据进行处理。

# 切图相关服务调用过程说明

其所有的访问api已通过Swagger发布，可通过`http://IP:PORT/swagger-ui/index.html`进行访问，其参考接口的具体访问方式为`http://IP:PORT/swagger-ui/index.html#/A/B`。

## 上传数据源

上传Geojson或Shp压缩包文件，并将其发布为数据源。对于Mapbox渲染引擎而言，其本身提供了[geojson-vt](https://github.com/mapbox/geojson-vt),可直接加载geojson数据并在前端发布为矢量切片；对于Shp压缩包而言，利用PostGIS的相关能力进行处理。当Shp数据大于1GB时，其网络传输和数据库入库将花费大量的时间，系统系统了管理员手动导入的功能。

- 上传数据源的接口，参考data-item-controller-impl/uploadFileAndGenerateDataItemUsingPOST
- 对于大数据量的Shp压缩包上传，参考data-item-controller-impl/addUsingPOST

### 大数据量Shp压缩文件的手动上传与入库

1.将shp压缩文件拷贝到后台的静态文件存储路径中的origin_data/shp/{username}/{uuid}/shp压缩包.zip路径下。
2.利用QGIS或者shp2postgresql将shp文件上传到后台的postgresql数据库中，注意其中的主键将设置为gid。

## 创建切片

首先定义切片，切片可发布为单图层切片也可以发布为多图层切片,其中shpName字段为数组，对应值为上传数据源所访问的name。

- 创建切片，参考tile-controller/addUsingPOST_2

## 切片进行切图

将创建号的切片进行切图，其调用参数包括tileName和maxZ，其中maxZ最少为6。在一次切片后还可以对maxZ进行二次赋值，继续切片。

- 发布切片的接口，参考tile-controller/generateTileUsingPOST

## 访问

首先查看已有切片的接口，然后利用切片的名称以及包含的矢量数据名称，即可进行访问。这里以交通线为例子，其切片查询返回的信息如下JSON所示：

```json
{
      "id": 29,
      "createTime": 1597974231954,
      "updateTime": 1597993062565,
      "name": "交通线",
      "tileLevel": 7,
      "cutTileStatus": "FINISH",
      "shpInfoList": [
        {
          "name": "transport_line",
          "type": "ST_MultiLineString",
          "props": "gid,geom,objectid,lane,width,elevt,areacode,changetype,shape_leng,name,drct,names,分类,rnp,rteg,ec,featid,elemstime,elemetime,ecp,type,主区,道路编号,cc,gb,changeatt,rn,sdtf,matrl",
          "ymin": 2393403.294585328,
          "xmax": 13227334.76584799,
          "xmin": 8684292.38869585,
          "ymax": 5063320.550888327
        }
      ]
    }
```

利用其中的name与shpInfo的name即可访问对应的切片，如图所示。

![访问方式](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/json1231.png)

- 查看已有切片的接口，参考tile-controller/listUsingGET_2

# 未来

利用[Citus](https://www.citusdata.com/)进行Postgresql的集群搭建，Citus是postgresql的集群解决方案，并支撑所有的PostGIS相关空间函数。

