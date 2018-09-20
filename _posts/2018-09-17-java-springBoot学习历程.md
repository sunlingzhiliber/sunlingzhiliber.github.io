---
title: "SpringBoot"
categories:
  - Java
tags:
  - SpringBoot
---


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


2. 