---
title: "Data Refactor"
categories:
  - OpenGMS
tags:
  - UDX
---
# 简介
[重构](/resource/MapAndRefactor.zip)实现了多种数据之间的转换。


## 实现原理
Wraper读取配置文件，通过反射加载dll，获取类以及类中的方法。
因此创建一个重构方法，需要[配置文件](配置文件)，以及[重构方法DLL](DLL).

## 配置文件
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
## DLL
在创建好配置文件后，你需要创建一个dll库，并在里面定义类名和方法名，如下图所示：
![](/assets/images/something/refactorDLL.png)


## 使用方法
在准备好**配置文件**和**DLL**之后，将其拷贝到RefactorWrapper.exe 的同级目录下，输入对应的`cmd`命令后即可调用重构方法 :
```
RefactorWraper.exe methodName params...
```


## 部署方法
在准备好所有东西后，将其打包为`Zip`包，上传到数据容器即可。 最终的文件参考目录如下：
![](/assets/images/something/refacotrDir.png)