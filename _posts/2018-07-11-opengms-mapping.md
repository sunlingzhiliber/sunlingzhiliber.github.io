---
layout:     post
title:      Data Refactor
date:       2018-07-10
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:
    - OpenGMS
    - UDX
---

# 简介

[数据映射](/resource/MapAndRefactor.zip)，实现模型需要的原始数据和UDX Schema描述的数据之间的相互转换的方法。利用数据映射，而可以将模型自定义数据，转换为结构化的UDX Data。

## 实现原理

Wraper读取配置文件，通过反射加载dll，获取类以及类中的方法。
因此创建一个重构方法，需要[配置文件](配置文件)，以及[重构方法DLL](DLL).
其中dll，必须至少包换两个方法，`Src2Udx(inputPath,outputPath)`与`Udx2Src(inputPath,outputPath)`，以便实现数据之间的相互转换。

## 配置文件

- cfg.xml
  提供给模型容器使用的，找到Wraper所在的路径

```xml
  <config>
    <type>exe</type>
    <start>MappingWraper.exe</start>
    <schema>./schema.xml</schema>
  </config>
```

- cfg_map.xml
  声明调用的DLL，以及DLL里面的类与方法

```xml
  <MappingMethodInfo name="Mapping.dll">
    <Method action="-w" name="Udx2SrcRun" class="Mapping.csvConverter" description="Udx2SrcRun">
      <Params datatype="System.String" type="in" name="srcFile" description="输入文件" schema="" />
      <Params datatype="System.String" type="out" name="outputFile" description="输出文件" schema="udx_schema.xml" />
    </Method>
    <Method action="-r" name="Src2UdxRun" class="Mapping.csvConverter" description="Src2UdxRun">
      <Params datatype="System.String" type="in" name="srcFile" description="输入文件" schema="udx_schema.xml" />
      <Params datatype="System.String" type="out" name="outputFile" description="输出文件" schema="" />
    </Method>
    <Method action="-h" name="help"  description="帮助信息">
    </Method>
    <Method action="-v" name="version"  description="版本信息">
    </Method>
    <Method action="-s" name="schema"  schema="udx_schema.xml">
    </Method>
    <Method action="-c" name="cability"  description="可访问的节点">
    </Method>
  </MappingMethodInfo>
```

## DLL

在创建好配置文件后，你需要创建一个dll库，并在里面定义类名和方法名，如下图所示：

![mapDLL](/img/content/mapDLL.png)

## 使用方法

在准备好**配置文件**和**DLL**之后，将其拷贝到MappingWrapper.exe 的同级目录下，输入对应的`cmd`命令后即可调用重构方法 :

```cmd
MappingWraper.exe -r -f dd.tif -u rr.xml //src->udx flag=1
MappingWraper.exe -w -u dd.xml -f rr.tif //udx->src flag=0
```

## 部署方法

在准备好所有东西后，将其打包为`Zip`包，上传到数据容器即可。 最终的文件参考目录如下：

![mapDir](/img/content/mapDir.png)