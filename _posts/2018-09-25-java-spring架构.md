---
layout:     post
title:      java-SpringMVC
date:       2018-07-10
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:
    - 面试之路
    - SpringMVC
---

Spring 是块 :meat_on_bone:。

# Spring的设计理念

Spring的核心就是Core、Context和Beans，它们构建了整个Spring的骨架结构，没有它们就不存在AOP、Web等上层的特性功能。
Spring可以认为是一种BOP(Bean Oriented Programming),Spring解决了一个关键的问题，就是
它让 `对象之间的依赖关系转用配置文件(注解)来进行管理`，也就是他的依赖注入机制。而这个注入关系在一个叫 Ioc 容器中管理，那 Ioc 容器就是被 Bean 包裹的对象。Spring 正是通过把对象包装在 Bean 中而达到对这些对象的管理以及一些列额外操作的目的。

## IOC

IOC就是控制反转，利用JAVA的反射机制，将实例的初始化交给Spring容器来完成。Spring可以通过配置文件来管理实例，当然目前配置文件的方式慢慢被注解的方式就行了取代。

## IOC 和工厂模式的区别

工厂模式也可以完成对象实例的创建，那么为什么我们一定要使用Spring的IOC呢？
工厂模式，在需求变化的时候，需要对工厂进行代码层次上的改变， 而IOC是利用反射实现的，允许我们不重新编译代码，因为它的对象都是动态生成的。

# MVC模式

MVC的原理图如下
![MVC模式原理图](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181016211718.jpg)

## SpringMVC

SpringMVC 框架是以请求为驱动，围绕 Servlet 设计，将请求发给控制器，然后通过模型对象，分派器来展示请求结果视图。其中核心类是 DispatcherServlet，它是一个 Servlet，顶层是实现的Servlet接口。

## 工作原理

- （1）客户端（浏览器）发送请求，利用DispatcherServlet(extends HttpServlet)调用this.do直接请求到 DispatcherServlet，调用this.doDispatch(request,response)

- （2）DispatcherServlet 根据请求信息 调用this.getHandler(请求) 根据this.handlerMapping，解析请求 获取请求对应的Handler。

- （3）解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，创建 HandlerAdapter 适配器。 在适配器工作之前，我们需要进行applyPreHandle, 对拦截器interceptors 进行处理。

- （4）HandlerAdapter 会根据 Handler 来调用真正的处理器来处理请求ha.handle()，处理相应的业务逻辑。通过反射invoke，获取returnValue。

```java
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
        Object[] args = this.getMethodArgumentValues(request, mavContainer, providedArgs);
        ......
        Object returnValue = this.doInvoke(args);//Invoke方法就是我们写的@RestController中的方法 利用反射从handler中获取bean
        ......
        return returnValue;
  }
```

对参数进行处理的包括两个部分，创建和验证

创建参数

```java
protected <T> Object readWithMessageConverters(HttpInputMessage inputMessage, MethodParameter parameter, Type targetType) throws IOException, HttpMediaTypeNotSupportedException, HttpMessageNotReadableException {
        boolean noContentType = false;

        MediaType contentType;

        ......获取contentType

        Class<?> contextClass = parameter.getContainingClass();
        Class<T> targetClass = targetType instanceof Class ? (Class)targetType : null;
        if (targetClass == null) {
            ResolvableType resolvableType = ResolvableType.forMethodParameter(parameter);
            targetClass = resolvableType.resolve();
        }

        HttpMethod httpMethod = inputMessage instanceof HttpRequest ? ((HttpRequest)inputMessage).getMethod() : null;
        Object body = NO_VALUE;

        AbstractMessageConverterMethodArgumentResolver.EmptyBodyCheckingHttpInputMessage message;
        try {
            label98: {
                ......获取转换器
                HttpMessageConverter converter;
                Class converterType;

                ......转换器对body进行处理
            }
        } catch (IOException var17) {
            throw new HttpMessageNotReadableException("I/O error while reading input message", var17);
        }

        if (body != NO_VALUE) {
            return body;
        } else if (httpMethod != null && SUPPORTED_METHODS.contains(httpMethod) && (!noContentType || message.hasBody())) {
            throw new HttpMediaTypeNotSupportedException(contentType, this.allSupportedMediaTypes);
        } else {
            return null;
        }
    }
```

在创建完参数后，需要进行validate

```java
protected void validateIfApplicable(WebDataBinder binder, MethodParameter parameter) {
        Annotation[] annotations = parameter.getParameterAnnotations();
        Annotation[] var4 = annotations;
        int var5 = annotations.length;

        for(int var6 = 0; var6 < var5; ++var6) {
            Annotation ann = var4[var6];
            Validated validatedAnn = (Validated)AnnotationUtils.getAnnotation(ann, Validated.class);
            if (validatedAnn != null || ann.annotationType().getSimpleName().startsWith("Valid")) {
                Object hints = validatedAnn != null ? validatedAnn.value() : AnnotationUtils.getValue(ann);
                Object[] validationHints = hints instanceof Object[] ? (Object[])((Object[])hints) : new Object[]{hints};
                binder.validate(validationHints);
                break;
            }
        }

    }
```

- （5） 获取returnValue后对其进行处理handleReturnValue，设置HTTP的输入，将retrunValue放置输出的Body之中.针对json的返回和ModelAndView的返回会有不同的方法

RequestResponseBodyMethodProcessor

```java
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {
        mavContainer.setRequestHandled(true);
        ServletServerHttpRequest inputMessage = this.createInputMessage(webRequest);//输入
        ServletServerHttpResponse outputMessage = this.createOutputMessage(webRequest);//输出
        this.writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
    }
```

ModelAndViewMethodReturnValueHandler:

```java
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
        if (returnValue == null) {
            mavContainer.setRequestHandled(true);
        } else {
            ModelAndView mav = (ModelAndView)returnValue;
            if (mav.isReference()) {
                String viewName = mav.getViewName();
                mavContainer.setViewName(viewName);
                if (viewName != null && this.isRedirectViewName(viewName)) {
                    mavContainer.setRedirectModelScenario(true);
                }
            } else {
                View view = mav.getView();
                mavContainer.setView(view);
                if (view instanceof SmartView && ((SmartView)view).isRedirectView()) {
                    mavContainer.setRedirectModelScenario(true);
                }
            }

            mavContainer.setStatus(mav.getStatus());
            mavContainer.addAllAttributes(mav.getModel());
        }
    }
```

- （6）处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。

```java
  private ModelAndView getModelAndView(ModelAndViewContainer mavContainer, ModelFactory modelFactory, NativeWebRequest webRequest) throws Exception {
        modelFactory.updateModel(webRequest, mavContainer);
        if (mavContainer.isRequestHandled()) {//已经处理了
            return null;
        } else {//未处理  会根据逻辑 View 查找实际的 View。
            ModelMap model = mavContainer.getModel();
            ModelAndView mav = new ModelAndView(mavContainer.getViewName(), model, mavContainer.getStatus());
            if (!mavContainer.isViewReference()) {
                mav.setView((View)mavContainer.getView());
            }

            if (model instanceof RedirectAttributes) {
                Map<String, ?> flashAttributes = ((RedirectAttributes)model).getFlashAttributes();
                HttpServletRequest request = (HttpServletRequest)webRequest.getNativeRequest(HttpServletRequest.class);
                if (request != null) {
                    RequestContextUtils.getOutputFlashMap(request).putAll(flashAttributes);
                }
            }

            return mav;
        }
```

- （7）DispaterServlet 把返回的 Model 传给 View（视图渲染）。或者直接将json对象传递给浏览器

- （8）把 View 返回给请求者（浏览器）

# 架构

## servlet开发存在的问题

映射问题、参数获取问题、格式化转换问题、返回值处理问题、视图渲染问题

## SpringMVC为解决上述问题开发的几大组件及接口

HandlerMapping、HandlerAdapter、HandlerMethodArgumentResolver、HttpMessageConverter、Converter、GenericConverter、HandlerMethodReturnValueHandler、ViewResolver、MultipartResolver

## DispatcherServlet、容器、组件三者之间的关系

## 叙述SpringMVC对请求的整体处理流程

## SpringBoot

请参考[SpringBoot学习历程](www.baidu.com)

# SpringAOP

## AOP的实现分类

编译期、字节码加载前、字节码加载后三种时机来实现AOP

## 深刻理解其中的角色

AOP联盟、aspectj、jboss AOP、Spring自身实现的AOP、Spring嵌入aspectj。特别是能用代码区分后两者

## 接口设计：

AOP联盟定义的概念或接口：Pointcut（概念，没有定义对应的接口）、Joinpoint、Advice、MethodInterceptor、MethodInvocation

SpringAOP针对上述Advice接口定义的接口及其实现类：BeforeAdvice、AfterAdvice、MethodBeforeAdvice、AfterReturningAdvice；针对aspectj对上述接口的实现AspectJMethodBeforeAdvice、AspectJAfterReturningAdvice、AspectJAfterThrowingAdvice、AspectJAfterAdvice。

SpringAOP定义的定义的AdvisorAdapter接口：将上述Advise转化为MethodInterceptor

SpringAOP定义的Pointcut接口：含有两个属性ClassFilter（过滤类）、MethodMatcher（过滤方法）

SpringAOP定义的ExpressionPointcut接口：实现中会引入aspectj的pointcut表达式

SpringAOP定义的PointcutAdvisor接口（将上述Advice接口和Pointcut接口结合起来）

## SpringAOP的调用流程

## SpringAOP自己的实现方式（代表人物ProxyFactoryBean）和借助aspectj实现方式区分

# Spring事务体系源码以及分布式事务Jotm Atomikos源码实现
## jdbc事务存在的问题
## Hibernate对事务的改进
## 针对各种各样的事务，Spring如何定义事务体系的接口，以及如何融合jdbc事务和Hibernate事务的
## 三种事务模型包含的角色以及各自的职责
## 事务代码也业务代码分离的实现（AOP+ThreadLocal来实现）
## pring事务拦截器TransactionInterceptor全景
## X/Open DTP模型，两阶段提交，JTA接口定义
## Jotm、Atomikos的实现原理
## 事务的传播属性
## PROPAGATION_REQUIRES_NEW、PROPAGATION_NESTED的实现原理以及区别
## 事物的挂起和恢复的原理


# 数据库隔离级别
## Read uncommitted：读未提交
## Read committed ： 读已提交
## Repeatable read：可重复读
## Serializable ：串行化


# 数据库
## 数据库性能的优化

## 深入理解mysql的Record Locks、Gap Locks、Next-Key Locks
例如下面的在什么情况下会出现死锁：
start transaction; DELETE FROM t WHERE id =6; INSERT INTO t VALUES(6); commit;

## insert into select语句的加锁情况

## 事务的ACID特性概念

## innodb的MVCC理解

## undo redo binlog

  1 undo redo 都可以实现持久化，他们的流程是什么？为什么选用redo来做持久化？
  2 undo、redo结合起来实现原子性和持久化，为什么undo log要先于redo log持久化？
  3 undo为什么要依赖redo？
  4 日志内容可以是物理日志，也可以是逻辑日志？他们各自的优点和缺点是？
  5 redo log最终采用的是物理日志加逻辑日志，物理到page，page内逻辑。还存在什么问题？怎么解决？Double Write
  6 undo log为什么不采用物理日志而采用逻辑日志？
  7 为什么要引入Checkpoint？
  8 引入Checkpoint后为了保证一致性需要阻塞用户操作一段时间，怎么解决这个问题？（这个问题还是很有普遍性的，redis、ZooKeeper都有类似的情况以及不同的应对策略）又有了同步Checkpoint和异步Checkpoint
  9 开启binlog的情况下，事务内部2PC的一般过程（含有2次持久化，redo log和binlog的持久化）
  10 解释上述过程，为什么binlog的持久化要在redo log之后，在存储引擎commit之前？
  11 为什么要保持事务之间写入binlog和执行存储引擎commit操作的顺序性？（即先写入binlog日志的事务一定先commit）
  12 为了保证上述顺序性，之前的办法是加锁prepare_commit_mutex，但是这极大的降低了事务的效率，怎么来实现binlog的group commit？
  13 怎么将redo log的持久化也实现group commit？至此事务内部2PC的过程，2次持久化的操作都可以group commit了，极大提高了效率

# ORM框架: mybatis、Hibernate
演进之路
## jdbc
## Spring的JdbcTemplate
## hibernate
## JPA

## SpringDataJPA

1. 分页

```java
Page<User> users = repository.findAll(new PageRequest(1, 20));
```

2. jpa

```java
interface UserRepository extends CrudRepository<User, Long> {
  //count
  long countByLastname(String lastname);

  //remove
  long deleteByLastname(String lastname);
  List<User> removeByLastname(String lastname);

//query
List<Person> findByLastname(String lastname);

  List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);

  // Enables the distinct flag for the query
  List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
  List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);

  // Enabling ignoring case for an individual property
  List<Person> findByLastnameIgnoreCase(String lastname);
  // Enabling ignoring case for all suitable properties
  List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);

  // Enabling static ORDER BY for a query
  List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
  List<Person> findByLastnameOrderByFirstnameDesc(String lastname);

  Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);

User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
}
```

3. example

```java
Person person = new Person();
person.setFirstname("Dave");

ExampleMatcher matcher = ExampleMatcher.matching()
  .withIgnorePaths("lastname")
  .withIncludeNullValues()
  .withStringMatcherEnding();

  ExampleMatcher matcher = ExampleMatcher.matching()
  .withMatcher("firstname", endsWith())
  .withMatcher("lastname", startsWith().ignoreCase());

ExampleMatcher matcher = ExampleMatcher.matching()
  .withMatcher("firstname", match -> match.endsWith())
  .withMatcher("firstname", match -> match.startsWith());

Example<Person> example = Example.of(person,matcher);

return personRepository.findAll(example,pageRequest,sort);

```


# 安全
## Session和Cookie的区别和联系以及Session的实现原理
## SpringSecurity的认证过程以及与Session的关系
## CAS实现SSO
[CAS](http://elim.iteye.com/blog/2128728)


# 日志
# jdk自带的logging、log4j、log4j2、logback
# 门面commons-logging、slf4j
# 上述6种混战时的日志转换

# datasource
## c3p0
## druid
## JdbcTemplate执行sql语句的过程中对Connection的使用和管理

# HTTPS的实现原理

# Swagger2 结合SpringBoot 生成API文档

## @Api

修饰整个类，描述Controller的作用

## @ApiOperation

描述一个类的一个方法，或者说一个接口

`@ApiOperation(value="获取用户列表", notes="获取所有用户列表",produces = "application/json")`

## @ApiParam

单个参数描述，需要较少的参数，但是会侵入代码。
针对于复杂对象，可以使用@ModelAttribute注解，该注解是SPringMVC的注解，但是可以被Swagger解析。

## @ApiModel

用对象来接收参数

`@ApiModel(value = "User", description = "用户对象")`

## @ApiProperty

用对象接收参数时，描述对象的一个字段

```java
@ApiModelProperty(value = "ID",example = "1")
private Integer id;
```

## @ApiResponse

HTTP响应其中1个描述

## @ApiResponses

HTTP响应整体描述
`@ApiResponses(value = {@ApiResponse(code = 405,message = "Invalid input",response = Integer.class)})`

## @ApiIgnore

使用该注解忽略这个API

## @ApiError

发生错误返回的信息

## @ApiImplicitParam

一个请求参数
`@ApiImplicitParam(name = "id",value = "用户ID",dataType = "int",paramType = "path")`

paramType  支持 path，query，body，header，form
dataType 支持 "int","date","string","double","float","boolean","byte","object","long","date-time","file","biginteger"，"bigdecimal"，"uuid"
required 支持 true false
allowMultiple=true,

## @ApiImplicitParams

多个请求参数

```java
@ApiImplicitParams({  
        @ApiImplicitParam(name = "id",value = "用户ID",paramType = "path",dataType = "int"),  
        @ApiImplicitParam(name = "userName",value = "用户名称",paramType = "form",dataType = "string")
})
```