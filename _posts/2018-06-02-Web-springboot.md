---
layout:     post
title:      SpringBoot
date:       2018-07-10
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:
    - Java
    - SpringBoot
---

# Spring的设计理念与设计模式分析
Spring的核心就是Core、Context和Beans，它们构建了整个Spring的骨架结构，没有它们就不存在AOP、Web等上层的特性功能。

Spring可以认为是一种BOP(Bean Oriented Programming),Spring解决了一个关键的问题，就是
它让 `对象之间的依赖关系转用配置文件(注解)来进行管理`，也就是他的依赖注入机制。而这个注入关系在一个叫 Ioc 容器中管理，那 Ioc 容器就是被 Bean 包裹的对象。Spring 正是通过把对象包装在 Bean 中而达到对这些对象的管理以及一些列额外操作的目的。


## 设计模式
Spring 中使用的设计模式也很多，比如工厂模式、单例模式、模版模式等。。这里主要介绍代理模式和策略模式。

### 代理模式
代理模式就是给某一个对象创建一个代理对象，而由这个代理对象控制对原对象的引用，而创建这个代理对象就是可以在调用原对象时增加一些额外的操作。

### 策略模式
策略模式顾名思义就是做某事的策略，这在编程上通常是指完成某个操作可能有多种方法，这些方法各有千秋，可能有不同的适应的场合，然而这些操作方法都有可能用到。各一个操作方法都当作一个实现策略，使用者可能根据需要选择合适的策略。

# 使用
## 注解详解
1. Http请求的request和response
  - request
  @RequestHeader 能够获取Http请求的header
  @RequestBody 能够获取Http请求的整个数据体 application/json
  @RequestParam  能够获取query参数、post请求的key-value、post请求的文件(利用MultipartFile)
  HttpServletRequest 获取未包装的HttpRequest

  -----------------

  - response
  @Reponsebody 能够将返回的内容直接映射为JSON对象，如果不添加该注解(且只用了@Controller注解，如果使用了@RestController就相当于@Controller+@ResponseBody，且无法返回页面)则返回的是对应字符串指定的视图处理器
  @RespnseEntity 使用该注解作为返回值，不仅可以返回json结果，还可以定义返回的HttpHeaders和HttpStatus



# 源码解析
