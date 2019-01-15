---
layout:     post
title:      Data Container
date:       2018-07-10
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:
    - OpenGMS
    - 地理模型集成和共享
    - UDX
---

现代计算机技术依赖于冯·诺依曼架构，数据和算法分离存储，互相依赖。这也决定了任何地理模型的集成和运行最终都是模型程序和模型数据的耦合和连接。

具体来说就是*流程控制逻辑*和*数据配置逻辑*。
*流程控制逻辑*主要是负责对建模场景中模型之间的关系进行调配，利用顺序、循环、嵌套等环节来加以控制。
*数据配置逻辑*主要是负责对模型运行所需的数据资源进行调配，在模型与数据的对接、模型与模型之间的数据交换等环节来进行配置。

**数据容器**的存在就是为了在地理模型的集成领域中，将“模型服务集成的流程控制逻辑”和“模型服务运行的数据配置逻辑”进行有机解耦，使得两者在分布式地理模型集成应用过程中协调工作的同时，能够在各自内部独立深化发展。

>简化地理模型服务集成的数据配置、处理和调度的工作。

# UDX

配置模型服务和数据资源的连接，目前采取的都是个案(固定的数据格式)，缺乏一个通用的数据集成基础，从而使所构建的集成建模场景严格依赖于初始设定的数据资源，难以支撑在开放网络环境中进行探索式、可配置的模型-数据资源对接，也制约了建模个案研究成果的累积式集成，使得耗费大量精力构建的集成建模情景难以作为可重用的资源进行共享。

- **首先**，地理模型所涉及到的输入/输出/控制数据的异构特征增加了数据资源组织与管理的不确定性；
- **其次**，数据在地理模型中往往代表了物质流、能量流和信息流的交换，传统的以特点数据格式来代表的方法，难以在支撑模型合理运行方面保证数据内容的无损传输；

UDX旨在为面向地理模型服务与异构数据资源对接的数据规格描述方法，地理模型服务化集成应用提供一致的数据视图，一致的数据视图是连接异构资源和模型服务的通信基础。

UDX(统一数据表达-交换模型)主要有两个实现：一个是统一数据表达的UDX Schema，还有一个是统一数据交换的UDX Data，两者共同构成了统一数据表达-交换模型的内容。这两者的结构层次是同构的。UDX Schema负责对模型数据内容进行详尽的描述，UDX Data则是严格匹配于UDX Schema的具体数据内容。对于地理模型数据，UDX Data是具体数据内容的存储，UDX Schema则对地理模型数据规格在形式和结构层面的描述。

在Node-Kernel结构的基础上，UDX统一数据表达交换模型中还包含了相应的**语义概念信息**、**空间参考信息**、**单位量纲信息**和**数据表达通用模板信息**。通过将这些信息附加到特定的节点Node上，可以对地理模型数据进行完备的描述，提供既支持计算机识别的结构化信息，也易于研究者理解数据内含的语义信息。

## 基于UDX的统一数据交换

我们往往会遇到这样一个情况，在模型程序实现的源代码实现方面，不同的模型开发者往往采用不同的底层数据结构来支撑上层的逻辑计算。而由于地理模型在问题领域、时空尺度、建模方法、实现方式、运行环境等方面的诸多差异，利用某个确定的数据结构在适用于某些地理模型的同时，也面临难以支撑其他地理模型的困难；尤其是面对大量已有的地理模型资源方面（通常称作为Legacy Models），更是难以利用一个固定的数据结构来实现其数据接口。

直接的数据交换策略：

![直接数据交换策略](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/1.png)

虽然地理模型的数据接口在外观上表现出来的是某种格式的数据文件、某种类型的变量，但在内容上则是对特定信息的依赖。

统一数据表达-交换模型UDX是一种构造式的数据表达元模型，通过对信息内容的灵活组装，可以配置出与不同数据结构相对应的表达模型。UDX本身并不直接绑定到一个固定的数据结构之上，利用Node-Kernel结构配置出来的结果才是与某个数据结构相一致的。

无论是WKT形式的几何数据，还是Shapefile形式的数据，都可以通过数据信息的抽取向Node-Kernel结构中填充数据内容（也就是生成UDX Data）；更广泛的自定义格式的数据也可以通过此种方式实现统一的数据交换。

基于UDX的数据交换策略：

![UDX交换策略](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/2.png)

这种基于Node-Kernel的地理模型应用和集成运行，对于某个特定的地理模型而言似乎增加了数据处理的过程。但在综合地理模拟中，涉及到多个不确定的地理模型之间的集成与运行，通过直接的数据格式转换虽然能够方便于某一个特定模型的运行，但对于整个集成模拟应用又造成了新的不便：首先体现在广泛的自定义格式难以直接进行转换，模型使用者在不了解数据内容的前提下进行的数据转换往往具有尝试性，增加了模型使用的不便；其次，模型相关的数据格式多种多样，同源的信息可以表现为异构的数据组织，孤立的数据转换难以避免重复的代码编写；还有，异构地理模型的集成不仅仅只是对数据格式的简单转换，其中涉及到的大量信息的抽取和重构，特定数据格式支撑下的数据处理成果难以形成积累，也难以重用。

## 映射简介

[数据映射](/resource/MapAndRefactor.zip)，实现模型需要的原始数据和UDX Schema描述的数据之间的相互转换的方法。利用数据映射，而可以将模型自定义数据，转换为结构化的UDX Data。

### 映射实现原理

Wraper读取配置文件，通过反射加载dll，获取类以及类中的方法。
因此创建一个重构方法，需要[配置文件](配置文件)，以及[重构方法DLL](DLL).
其中dll，必须至少包换两个方法，`Src2Udx(inputPath,outputPath)`与`Udx2Src(inputPath,outputPath)`，以便实现数据之间的相互转换。

### 映射配置文件

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

### 映射DLL

在创建好配置文件后，你需要创建一个dll库，并在里面定义类名和方法名，如下图所示：

![mapDLL](/img/content/mapDLL.png)

### 映射使用方法

在准备好**配置文件**和**DLL**之后，将其拷贝到MappingWrapper.exe 的同级目录下，输入对应的`cmd`命令后即可调用重构方法 :

```cmd
MappingWraper.exe -r -f dd.tif -u rr.xml //src->udx flag=1
MappingWraper.exe -w -u dd.xml -f rr.tif //udx->src flag=0
```

### 映射部署方法

在准备好所有东西后，将其打包为`Zip`包，上传到数据容器即可。 最终的文件参考目录如下：

![mapDir](/img/content/mapDir.png)

## 重构简介

[重构](/resource/MapAndRefactor.zip)实现了多种数据之间的转换。

### 重构实现原理

Wraper读取配置文件，通过反射加载dll，获取类以及类中的方法。
因此创建一个重构方法，需要[配置文件](配置文件)，以及[重构方法DLL](DLL).

### 重构配置文件

- cfg.xml
  提供给模型容器使用的，找到Wraper所在的路径

```xml
  <config>
    <type>exe</type>
    <start>RefactorWraper.exe</start>
    <schema>./schema.xml</schema>
  </config>
```

- cfg_refactor.xml
  声明调用的DLL，以及DLL里面的类与方法

```xml
  <RefactorMethodInfo name="GridTest.dll">
    <Method name="gridInvdist" class="GridTest.Grid" description="gridInvdist">
      <Params datatype="System.String" type="in" name="srcFile" description="输入文件" schema="./udx_schema.xml" />
      <Params datatype="System.String" type="out" name="outputFile" description="输出文件" schema="./udx_schema.xml" />
    </Method>
  </RefactorMethodInfo>
```

### 重构DLL

在创建好配置文件后，你需要创建一个dll库，并在里面定义类名和方法名，如下图所示：

![refactorDLL](/img/content/refactorDLL.png)

## 重构使用方法

在准备好**配置文件**和**DLL**之后，将其拷贝到RefactorWrapper.exe 的同级目录下，输入对应的`cmd`命令后即可调用重构方法 :

```cmd
RefactorWraper.exe methodName params...
```

## 重构部署方法

在准备好所有东西后，将其打包为`Zip`包，上传到数据容器即可。 最终的文件参考目录如下：
![refacotrDir](/img/content/refacotrDir.png)

# 服务总线

地理模型的分布式集成应用过程中，对于支撑模型运行的数据资源配置工作必然涉及到具体网络节点之间的访问。单个模型服务的调用、多个模型服务的集成，以及参数调优和可视化分析等工作，都需要建模者进行频繁且大量交叉的数据资源配置。因而，在**地理模型数据服务总线**中，为集成建模任务提供统一的数据资源管理功能，是降低数据资源请求分发难度的重要内容。

其中服务总线一方面要提供特定的数据资源，另一方面也要提供相关的数据准备工作(抽取，映射，重构)。

负责将分散的模型数据资源进行统一的管理，为集成建模任务中的每个模型服务配置合适的数据资源，根据地理模型数据规格的服务化接口，响应数据请求和进行数据分发。

在参与式建模任务中，必须从模型执行端和数据资源访问端考虑数据配置工作。数据服务控制模块旨在访问数据资源，在数据服务控制模块中，每个相关的数据资源都被分配了一个数据存根Data Stub(Link configuration)；数据存根用于帮助参与建模者处理从模型服务控制模块路由的数据要求。

