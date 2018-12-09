---
layout:     post
title:      Web服务器
date:       2018-07-10
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:
    - Web服务器
---

todo

# 转发(forward)和重定向(Redirect)的区别

转发是服务器行为:

```java
request.getRequestDispatcher("login_success.jsp").forward(request, response);
```

重定向是客户端行为，是利用服务器返回的状态码来实现的。客户端浏览器请求服务器的时候，服务器会返回一个状态码(301或者302)。如果服务器返回301或者302，则浏览器会到新的网址重新请求该资源。

具体的区别如下：
1.从地址栏显示来说：forward时，浏览器并不知道服务器做了什么，因此它的地址不会改变。而redirect时，浏览器获得状态码之后，会重新去请求新的网址，从而显示的是新的网址
2.从数据共享来说：forward转发的页面可以共享request里面的数据，而redirect不能共享数据。
3.从运用场景来说：forward一般用于用户登陆时，进行角色转发到相应的模块。redirect一般用于用户注销登陆时返回主页面和跳转到其它的网站等。

# Tomcat

Tomcat是一款开源的应用服务器，凭借技术先进、性能稳定、体积小巧、扩展性好，深受开发人员和软件开发商的认可。

![Tomcat架构](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181107184142.jpg)

Tomcat中最顶层的容器是server，代表整个服务器。然后是Service，用于提供具体服务，一个Server包含多种Service。

Service主要包括两个部分：Connector和Container。

>Connector用于处理连接相关的事情，并提供Socekt与Request、Response相关的转化。
>Container用于封装和管理Servlet，并完成具体的Request请求。

注意Service可以拥有多个Connector，因为一个服务可以有多个连接，如同时提供Http和Https链接。

## connector

![Connector结构图](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181107185532.jpg)

Connector使用ProtocolHandler来处理请求的，不同的ProtocolHandler代表不同的连接类型，比如：Http11Protocol使用的是普通Socket来连接的，Http11NioProtocol使用的是NioSocket来连接的。

其中ProtocolHandler由包含了三个部件：Endpoint、Processor、Adapter。

1）Endpoint用来处理底层Socket的网络连接，Processor用于将Endpoint接收到的Socket封装成Request，Adapter用于将Request交给Container进行具体的处理。
2）Endpoint由于是处理底层的Socket网络连接，因此Endpoint是用来实现TCP/IP协议的，而Processor用来实现HTTP协议的，Adapter将请求适配到Servlet容器进行具体的处理。
3）Endpoint的抽象实现AbstractEndpoint里面定义的Acceptor和AsyncTimeout两个内部类和一个Handler接口。Acceptor用于监听请求，AsyncTimeout用于检查异步Request的超时，Handler用于处理接收到的Socket，在内部调用Processor进行处理。

# nginx

## 负载均衡

`nginx`的负载均衡属于`服务器端均衡`，在服务器端拥有服务实例的清单，在服务器进行负载均衡算法分配。

![服务器端负载均衡](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/20181126102548.png)

负载均衡用于从“upstream”模块定义的后端服务器列表中选取一台服务器接受用户的请求。一个最基本的upstream模块是这样的，模块内的server是服务器列表。

```text
#动态服务器组
upstream dynamic_zuoyu {
    server localhost:8080;  #tomcat 7.0
    server localhost:8081;  #tomcat 8.0
    server localhost:8082;  #tomcat 8.5
    server localhost:8083;  #tomcat 9.0
}
```

当然Nginx拥有丰富的负载均衡策略：

| 轮询               | 默认方式        |
|--------------------|-----------------|
| weight             | 权重方式        |
| ip_hash            | 依据ip分配方式  |
| least_conn         | 最少连接方式    |
| fair（第三方）     | 响应时间方式    |
| url_hash（第三方） | 依据URL分配方式 |