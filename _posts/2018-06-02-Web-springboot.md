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

# IOC和DI

控制反转和依赖注入是Spring的的创建维护对象的关键。SpringIOC容器(ApplicationContext)负责创建容器。
全局配置使用java配置(如数据库、MVC配置)，业务Bean由注解配置(@Component、@Service、@Repository、@Controller)

## 注解声明bean

声明Bean：
@Component(无明确的角色)
@Service
@Repository
@Controller
这四种方式是等效的，但是通过不同的注解名，为Bean设置了不同的使用逻辑。

注入Bean：
@Autowired
@Inject
@Resource

## Java配置Bean

@Configuration 声明当前类为一个配置类
@Bean 注解在方法上，声明当前方法返回值为一个Bean，且方法名为Bean的名字

## Bean的声明周期

Singleton：默认的单列模式
Prototype：每次调用创建一个
Request：为每个HTTP request创建一个Bean实例
Session：为每个HTTP session创建一个Bean实例

# AOP

Spring提供面向切面编程，AOP存在的目的是为了解耦。AOP可以让一组类共享相同的行为。在OOP中只能通过继承和实现接口，这样使代码的耦合度增强。阻碍了更多的行为添加到一组类上。

Spirng提供支持AspectJ的注解式切面编程：
（1） 使用@Aspect声明一个切面，使用@Component注入到容器中。
（2） @Pointcut定义切点(可以定义为注解`@Pointcut("@annotation(*)")`，也可以定义为方法`@Pointcut("@execution(*)")`)
（3） 围绕切点使用@Before、@After、@Around与@AfterReturning来定义advice

# 资源的调用

使用注解@Value，可以调用普通文件、网址、配置文件和环境变量等各种资源。

（1）注入字符 @value("字符串")
（2）注入操作系统属性 @Value("#{systemProperties['os.name']}")
（3）注入表达式结果   @Value("#{T(java.lang.Math).random()}")
（4）注入文件         @Value("文件路径")
（5）注入网址          @Value("网址")
（7）注入属性       @Value("${配置文件属性}")

# 事件(Application Event)

Application Event为Bean之间的消息通信提供了支持，当一个Bean处理完一个任务后，希望另一个Bean知道并进行相应的操作，这个时候我们就可以让另外一个Bean监听当前Bean所发送的时间。

流程如下：
1.自定义事件  extends ApplicationEvent
2.定义事件监听 implements ApplicationListener<自定义事件>
3.定义事件发布类 该类通过注入 @Autowired ApplicationContext 然后ApplicationContext.publishEvent(new 自定义事件())，
也可以 继承接口 ApplicationEventPublisher 重写 publishEvent(ApplicationEvent evnet).

## Spring Aware

在项目开发中，我们不可避免要用到Spring容器本身的功能资源，这时候Bean本身要意识到容器的存在，才能调用其中的资源。

我们可以通过 创建一个@Component 继承 ApplicationContextAware接口 获取ApplicationContext容器 进而获取容器中的资源。

当然这里的ApplicationContextAware接口可以获取所有的服务，但是我们原则上应该使用什么接口，就使用什么接口。

Spring Aware提供的接口包括：
| 接口名                          | 功能                         |
|--------------------------------|----------------------------------------------|
| BeanNameAware                  | 获取容器中Bean的名称                         |
| BeanFactoryAware               | 获取当前Bean Factory，这样可以调用容器的服务 |
| MessageSourceAware             | 获取Message Source，这样可以获取文本信息     |
| ResourceLoaderAware            | 获取资源加载器，可以获取外部资源             |
| ApplicationEventPublisherAware | 获取发布事件，可以发布事件                   |

# 定时任务

@Scheduled定时任务使用方式为定义在方法上，说明该方法定时执行
@Scheduled(fixedRate=5000)
@Scheduled(cron="")//UNIX下的定时任务

# @Conditional

根据不同的条件来动态创建Bean

```java
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        return conditionContext.getEnvironment().getProperty("os.name").contains("Windows");
    }
}

@Configuration
public class BeanConfig {
    @Bean
    @Conditional(WindowsCondition.class)
    public String windowsString(){
        return new String("windows条件");
    }
}
```

# 多线程

首先定义多线程池

```java
@Configuration
@EnableAsync
public class TaskExecutorConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor taskExecutor=new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(5);
        taskExecutor.setMaxPoolSize(10);
        taskExecutor.setQueueCapacity(25);
        taskExecutor.initialize();
        return taskExecutor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return null;
    }
}
```

然后就可以在Bean中使用@Async进行使用

```java
@Component
public class AsyncTaskComponent {
    @Async
    public void executeAsyncTask(Integer integer){
        System.out.println("执行异步任务："+integer);
    }

    @Async
    public void executeAsyncTaskPlus(Integer integer){
        System.out.println("执行异步任务："+(integer+1));
    }
}

```

# Enable**注解

@EnableAspectJAutoProxy 开启对AspectJ自动代理的支持
@EnableAsync 开启异步方法的支持
@EnableScheduling 开启计划任务的支持
@EnableWebMvc 开启Web MVC的配置支持
@EnableConfigurationProperties 开启对@ConfigurationProperties注解配置Bean的支持
@EnableJpaRepositories 开启对Spring Data JPA repository的支持
@EnableTransactionManagement 开启注解式事务的支持
@EnableCaching  开启注解式的缓存支持

# SpringMVC

## 注解详解

1.Http请求的request和response

- request
  @RequestHeader 能够获取Http请求的header
  @RequestBody 能够获取Http请求的整个数据体     application/json application/xml application/x-www-form-urlencoded
  @RequestParam  能够获取query参数、post请求的key-value、post请求的文件(利用MultipartFile)
  HttpServletRequest 获取未包装的HttpRequest

  -----------------

- response
  @Reponsebody 能够将返回的内容直接映射为JSON对象，如果不添加该注解(且只用了@Controller注解，如果使用了@RestController就相当于@Controller+@ResponseBody，且无法返回页面)则返回的是对应字符串指定的视图处理器
  @RespnseEntity 使用该注解作为返回值，不仅可以返回json结果，还可以定义返回的HttpHeaders和HttpStatus

2.@RequestMapping(value="",method=*,produces={"application/xml;charset=UTF-8"})
produces可以设置返回对象的媒体类型。

## HttpMessageConverter

根据Content-Type(入口)、Accept(出口)、controller方法的consumes(入口)/produces(出口)、Converter.mediaType以及Converter的排列顺序这四个属性，来将Inputstream转换为Java对象，或者Java对象转换为InputStream。

已有的转换器包括

| 名称                                   | 作用                                           | 读支持MediaType                   | 写支持MediaType                   |
|----------------------------------------|------------------------------------------------|-----------------------------------|-----------------------------------|
| ByteArrayHttpMessageConverter          | 数据与字节数组的相互转换                       | */*                               | application/octet-stream          |
| StringHttpMessageConverter             | 数据与String类型的相互转换                     | text/*                            | text/plain                        |
| FormHttpMessageConverter               | 表单与MultiValueMap的相互转换                  | application/x-www-form-urlencoded | application/x-www-form-urlencoded |
| SourceHttpMessageConverter             | 数据与javax.xml.transform.Source的相互转换     | text/xml和application/xml         | text/xml和application/xml         |
| MarshallingHttpMessageConverter        | 使用Spring的Marshaller/Unmarshaller转换XML数据 | text/xml和application/xml         | text/xml和application/xml         |
| MappingJackson2HttpMessageConverter    | 使用Jackson的ObjectMapper转换Json数据          | application/json                  | application/json                  |
| MappingJackson2XmlHttpMessageConverter | 使用Jackson的XmlMapper转换XML数据              | application/xml                   | application/xml                   |
| BufferedImageHttpMessageConverter      | 数据与java.awt.image.BufferedImage的相互转换   | Java I/O API支持的所有类型        | Java I/O API支持的所有类型        |

默认是返回json格式的数据，引入jackson-dataformat-xml 后可以将json转换为xml，但是这样就会使默认的json输出变成xml输出
定义了produces和consumes之后就会进行输入和输出的限制。

![HttpMessageConverter](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/20181115185957.png)

## 服务器端推送（SSE）

# SpringBoot

## SSL的设置

## WebSocket的设置

## 模板引擎

## Spring Data

SpringData为我们使用统一的API来对不同的数据存储技术进行数据访问操作提供了支持。无论是关系数据库还是非关系数据库都可以以统一的标准进行访问。

该标准包括CURD、查询、排序和分页。

使用Spring Data JPA访问数据，只需要一个接口，集成JpaRepository。
使用Spirng Data Repository可以极大减少数据访问层的代码。
它还可以根据属性名对数据库中的实体进行操作，不用写实现，只用声明。

1.常规查询。
2.限制结果数量
利用top或者first来实现
3.利用@Query写SQL语句
4.Sort排序、Page分页

## 事务

@Transactional

## 数据缓存

@EnableCaching

## Spring Security

## Spirng Batch

处理大量数据操作的一个框架。主要用来读取大量数据，然后进行一定的处理在输出为指定形式。

## 异步消息

为了系统与系统之间的通信

## Spring Integration

系统集成，解决了不同系统之间的交互问题，通过异步消息驱动来达到系统交互之间的松耦合。

## 应用监控

# Spring Cloud

为服务是最近特别火的概念，它的含义是：使用定义好边界的小的独立组件来做好一件事情。微服务是相对于传统单块式架构而言的。

使用微服务不可避免的需要将功能按照边界拆分为单个服务，体现分布式的特征。这个时候每个微服务之间的通信就是我们需要解决的问题。

Spring Cloud包括

1.配置管理 通过Config Server 我们可以集中存储所有应用的配置文件。
2.服务发现 通过Eureka来实现服务发现
3.路由网关 让所有的微服务对外只有一个接口，我们只需要访问一个网关，由网关将请求代理到不同的服务中。通过Zuul来实现
4.负载均衡 使用Ribbon和Feign
5.断路器  当某个服务不可用时，如何调用后备方法。提高容错性、阻止级联错误
6.代理服务