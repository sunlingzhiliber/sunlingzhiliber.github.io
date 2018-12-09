---
layout:     post
title:      SpringCloud
date:       2018-07-10
author:     Liber Sun
header-img: img/post-basic.jpg
catalog: true
tags:
    - Java
    - SpringCloud
---

SpringCloud是Spring提供的用于创建"微服务"的架构。

![功能划分](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/20181126103717.png)

# 基础认知

## 集群

>计算机集群简称集群是一种计算机系统，它通过一组松散集成的计算机软件和/或硬件连接起来高度紧密地协作完成计算工作。在某种意义上，他们可以被看作是一台计算机。集群系统中的单个计算机通常称为节点，通常通过局域网连接，但也有其它的可能连接方式。集群计算机通常用来改进单个计算机的计算速度和/或可靠性。一般情况下集群计算机比单个计算机，比如工作站或超级计算机性能价格比要高得多

总结来说：
同一个业务，部署在多个服务器上(不同的服务器运行同样的代码，干同一件事)

## 分布式

>分布式系统是一组计算机，通过网络相互连接传递消息与通信后并协调它们的行为而形成的系统。组件之间彼此进行交互以实现一个共同的目标。

总结来说：
将应用程序按功能进行拆分，模块之间独立的部署到不同的服务器，在使用的时候再将这些独立的模块组合起来构成一个应用。（不同的服务器运行不同的代码，为了同一个目的）

## CAP理论

当你使用分布式架构的时候，你必然会碰到CAP的问题。我们来看看CAP意味着什么？

C：数据一致性(consistency)
所有节点拥有数据的最新版本
A：可用性(availability)
数据具备高可用性
P：分区容错性(partition-tolerance)
容忍网络出现分区，分区之间网络不可达。

这三点中我们要意识到，分区容错性是必须的，是客观世界一定存在的问题。

# SpringClound

## 为什么需要SpringCloud

我们需要将我们的庞大的项目，分解成多个小的模块(注意这里的模块是部署到了不同的服务器)，通过小模块的有机组合，来完成业务功能。

## Spring clound的功能

### 基础功能

客户端负载均衡： Spring Cloud Ribbon
服务容错保护： Spring  Cloud Hystrix  
声明式服务调用： Spring  Cloud Feign
API网关服务：Spring Cloud Zuul
分布式配置中心： Spring Cloud Config

#### 服务治理： Spring  Cloud Eureka<!-- TOC -->

当我们将系统拆分为多个子系统之后，那么我们如何解决子系统之间的`通讯`问题呢？
子系统与子系统不再同一个环境下，那就需要`远程调用`。我们考虑可以使用HttpClient建立请求,在Spring中我们可以利用Spring封装的[RestTemplate](#RestTemplate)。但是我们要使用远程调用可能就必须要知道ip地址。

当IP地址改变的时候，我们就要手动更新IP地址。在服务较多的情况下，我们手动的维护这些静态配置就是噩梦。

>为了解决微服务架构中的服务实例维护问题(ip地址)， 产生了大量的服务治理框架和产品。 这些框架和产品的实现都围绕着服务注册与服务发现机制来完成对微服务应用实例的自动化管理。

Spring  Cloud Eureka就是一种服务注册中心。

![服务注册中心](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181124152013.jpg)

注意这里服务A，可能会注册多个，那么一个服务A，就会有多个服务器可以提供，那么就需要根据用户的情况，根据服务器的使用情况进行[动态负载均衡](#负载均衡ribbon)。
并不是每个服务都需要注册到服务中心，因为它可能只是作为一个`单纯的服务消费者`。

下面是Eureka的治理机制：

- 服务提供者
  - 服务注册：启动的时候会通过发送REST请求的方式将自己注册到Eureka Server上，同时带上了自身服务的一些元数据信息。
  - 服务续约：在注册完服务之后，服务提供者会维护一个心跳用来持续告诉Eureka Server:  "我还活着 ” 、
  - 服务下线：当服务实例进行正常的关闭操作时，它会触发一个服务下线的REST请求给Eureka Server, 告诉服务注册中心：“我要下线了 ”。
- 服务消费者
  - 获取服务：当我们启动服务消费者的时候，它会发送一个REST请求给服务注册中心，来获取上面注册的服务清单
  - 服务调用：服务消费者在获取服务清单后，通过服务名可以获得具体提供服务的实例名和该实例的元数据信息。在进行服务调用的时候，优先访问同处一个Zone中的服务提供方。
- Eureka Server(服务注册中心)：
  - 失效剔除：默认每隔一段时间（默认为60秒） 将当前清单中超时（默认为90秒）没有续约的服务剔除出去。
  - 自我保护：EurekaServer 在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%(通常由于网络不稳定导致)。 Eureka Server会将当前的- 实例注册信息保护起来， 让这些实例不会过期，尽可能保护这些注册信息。

![应用情形](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181124153947.jpg)

#### RestTemplate

通过Eureka服务治理框架，我们可以通过服务名来获取具体的服务实例的位置了(IP)。一般在使用SpringCloud的时候不需要自己手动创建HttpClient来进行远程调用。

```java
    // 传统的方式，直接显示写死IP是不好的！
    //private static final String REST_URL_PREFIX = "http://localhost:8001";

    // 服务实例名
    private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";

    /**
     * 使用 使用restTemplate访问restful接口非常的简单粗暴无脑。 (url, requestMap,
     * ResponseBean.class)这三个参数分别代表 REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。
     */
    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept) {
        return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
    }
```

#### 负载均衡Ribbon

为了实现服务的高可用性，我们可以将服务提供者集群。对同一个服务，利用多台服务器来承载以提高并发访问量。利用多台服务器`合理分摊`用户的请求。

这个时候我们可以使用`nginx`进行负载均衡，这是一种`服务端的负载均衡`。服务实例的清单在服务器端，在服务器进行负载均衡算法分配。

当然SpringCloud也提供了负载均衡的功能，只不过它是一种`客户端的负载均衡`。这个功能的实现就是Ribbon。服务实例的清单在客户端(从Eureka服务注册中心获取)，客户端进行负载均衡算法分配。客户端可以从Eureka Server中得到一份服务清单，在发送请求时通过负载均衡算法，在多个服务器之间选择一个进行访。

Ribbon的默认负载均衡策略是轮询，我们也可以对其进行自定义。另外SpringCloud在CAP理论中，是选着了 可用性和分区容错性。

### 高级功能

到目前为止，我们服务实现了一些功能:能够根据服务名来远程调用其他服务，可以实现服务端的负载均衡。

![已实现的功能](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181124172508.jpg)

但是，如果我们在调用某个功能时(该功能由多个服务嵌套组合形成)，如果其中的某个服务出现延迟，会怎么样？这个服务会卡死，知道某服务完全。

在`高并发`的情况下，由于单个服务的延迟，可能导致所有的请求都出现延迟状态，而服务器同时支持的线程是有限的。这就会在几秒之内使服务器处于负载饱满的状态，直到该服务不可用，导致依赖该服务的关联服务也不可用，最终导致这个分布式系统都不可用，这就是"雪崩"

![雪崩](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181124172938.jpg)

针对以上问题，Spring Cloud 提供了 Hystrix实现了断路器、线程隔离等一系列服务保护功能。

#### Hystrix

- Fallback：但某个服务单元发生故障之后，通过断路器的故障监控，向调用方返回一个错误响应，而不是长时间的等待。这样就不会使得线程因调用故障服务被长时间占用不释放，避免了故障在分布式系统中的蔓延。
- 线程隔离：它会为每一个依赖服务创建一个独立的线程池，这样就算某个依赖服务出现延迟过高的情况，也只是对该依赖服务的调用产生影响， 而不会拖慢其他的依赖服务。

Hystrix提供几个熔断关键参数：滑动窗口大小（20）、 熔断器开关间隔（5s）、错误率（50%）

1.每当20个请求中，有50%失败时，熔断器就会打开，此时再调用此服务，将会直接返回失败，不再调远程服务。
2.直到5s钟之后，重新检测该触发条件，判断是否把熔断器关闭，或者继续打开。
Hystrix还有请求合并、请求缓存这样强大的功能，在此就不具体说明了。

#### Feign

上面已经介绍了Ribbon和Hystrix了，可以发现的是：他俩作为基础工具类框架广泛地应用在各个微服务的实现中。我们会发现对这两个框架的使用几乎是同时出现的。

为了简化我们的开发，`Spring Cloud Feign`出现了！它基于 Netflix Feign 实现，整合了 Spring Cloud Ribbon 与 Spring Cloud Hystrix,  除了整合这两者的强大功能之外，它还提供了`声明式的服务调用`(不再通过RestTemplate)。

>Feign是一种声明式、模板化的HTTP客户端。在Spring Cloud中使用Feign, 我们可以做到使用HTTP请求远程服务时能与调用本地方法一样的编码体验，开发者完全感知不到这是远程方法，更感知不到这是个HTTP请求。

服务绑定：定义接口之后，我们在@Controller这一层，就直接使用DeptClientService

```java
// value --->指定调用哪个服务
// fallbackFactory--->熔断器的降级提示
@FeignClient(value = "MICROSERVICECLOUD-DEPT", fallbackFactory = DeptClientServiceFallbackFactory.class)
public interface DeptClientService {


    // 采用Feign我们可以使用SpringMVC的注解来对服务进行绑定！
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list();

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept);
}
```

熔断器：

```java
/**
 * Feign中使用断路器
 * 这里主要是处理异常出错的情况(降级/熔断时服务不可用，fallback就会找到这里来)
 */
@Component // 不要忘记添加，不要忘记添加
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept get(long id) {
                return new Dept().setDeptno(id).setDname("该ID：" + id + "没有没有对应的信息,Consumer客户端提供的降级信息,此刻服务Provider已经关闭")
                        .setDb_source("no this database in MySQL");
            }

            @Override
            public List<Dept> list() {
                return null;
            }

            @Override
            public boolean add(Dept dept) {
                return false;
            }
        };
    }
}
```

#### Zuul

在使用到上面的技术后，我们现在的架构设计会是这个样子的：
![服务架构](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181126102817.jpg)

但这种架构存在一些问题:

1.`路由规则与服务实例的维护间题`:外层的负载均衡(nginx)需要维护所有的Open服务实例清单。(upstream模块中需要维护每个服务实例的IP)
2.`签名校验、登陆校验冗余问题`：为了保证对外服务的安全性，我们在服务器端实现的微服务接口(非OpenService)，都会进行一定的权限校验机制，但是我们的服务是独立的，我们不得不`在这些应用中都实现这样一套校验逻辑`，这样就形成了代码冗余。

为了解决这些问题，`API网关的概念`应用而生。
在SpringCloud中了提供了基于Netfl ix Zuul实现的API网关组件`Spring Cloud Zuul`。

- SpringCloud Zuul通过与SpringCloud Eureka进行整合，将自身注册为Eureka服务治理下的应用，同时从Eureka中获得了所有其他微服务的实例信息。外层调用都必须通过API网关，使得将维护服务实例的工作交给了服务治理框架自动完成。
- 在API网关服务上进行统一调用来对微服务接口做前置过滤，以实现对微服务接口的拦截和校验。

#### SpringCloud Config

随着业务的扩展，我们的服务会越来越多。每个服务都有自己的配置文件。
在分布式系统中，某一个服务信息的变更，都会引起一系列的更新与重启。因此我们需要一个解决分布式系统的配置管理方案。`Spring Cloud Config`

它包含了Client和Server两个部分，server提供配置文件的存储、以接口的形式将配置文件的内容提供出去，client通过接口获取数据、并依据此数据初始化自己的应用。

- 简单来说，使用Spring Cloud Config就是将配置文件放到统一的位置管理(比如GitHub)，客户端通过接口去获取这些配置文件。
- 在GitHub上修改了某个配置文件，应用加载的就是修改后的配置文件。