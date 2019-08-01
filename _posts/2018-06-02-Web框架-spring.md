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

# Spring

## 核心理念

### IOC、DI

IOC(控制反转)和DI(依赖注入)是Spring的的创建维护对象的关键。

IOC(控制反转)，寻求上层控制下层，这样当下层修改时我们不需要更改太多的代码，举例就是 🚗和 轮胎的关系。
我们可以DI(依赖注入),来实现IOC。所谓的DI，就是把底层类作为参数传入上层类，实现上层类对下层类的“控制”。

实现了DI之后，我们在创建对象的时候仍然需要大量的构造代码，因此我们需要IOC容器，我们只需要维护一个Configuration（可以是xml可以是一段代码），而不用每次初始化一辆车都要亲手去写那一大段初始化的代码。这是引入IoC Container的第一个好处。IoC Container的第二个好处是：我们在创建实例的时候不需要了解其中的细节,我们只需要提供Config，然后利用IOC容器，自顶而下寻找依赖关系，到达底层后一步步向上创建。

Spring IOC容器(ApplicationContext)利用Java反射，负责创建对象，用户只需要声明对象。

#### 声明

- @Component(无明确的角色)
- @Service
- @Repository
- @Controller

这四种方式是等效的，但是通过不同的注解名，为Bean设置了不同的使用逻辑。

我们还可以通过`@Configuration`声明当前类为一个配置类,`@Bean`注解在方法上，声明当前方法返回值为一个Bean，且方法名为Bean的名字。

#### Bean的声明周期

- Singleton：**默认**的单列模式
- Prototype：每次调用创建一个
- Request：为每个HTTP request创建一个Bean实例
- Session：为每个HTTP session创建一个Bean实例

#### SpringBean

在 Spring 中，那些组成应用程序的主体及由 Spring IOC 容器所管理的对象，被称之为 bean。简单地讲，bean 就是由 IOC 容器初始化、装配及管理的对象，除此之外，bean 就与应用程序中的其他对象没有什么区别了。

Spring bean默认是单例的,这些单例Bean在多线程程序下如何保证线程安全呢？ 例如对于Web应用来说，Web容器对于每个用户请求都创建一个单独的Sevlet线程来处理请求，引入Spring框架之后，每个Action都是单例的，那么对于Spring托管的单例Service Bean，如何保证其安全呢？ Spring的单例是基于BeanFactory也就是Spring容器的，单例Bean在此容器内只有一个，Java的单例是基于 JVM，每个 JVM 内只有一个实例。

#### 注入

注入Bean有三种方式：

@Autowired
@Inject
@Resource


IOC: 控制反转也叫依赖注入。IOC利用java反射机制，AOP利用代理模式。IOC 概念看似很抽象，但是很容易理解。说简单点就是将对象交给容器管理，你只需要在spring配置文件中配置对应的bean以及设置相关的属性，让spring容器来生成类的实例对象以及管理对象。在spring容器启动的时候，spring会把你在配置文件中配置的bean都初始化好，然后在你需要调用的时候，就把它已经初始化好的那些bean分配给你需要调用这些bean的类。

AOP： 面向切面编程。（Aspect-Oriented Programming） 。AOP可以说是对OOP的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码，属于静态代理。

### AOP

AOP(面向切面编程)，其目的在于去重、解耦。AOP可以将一些公共的代码抽离出来，然后附加到一些类上，从而使这些类共享相同的行为。在传统的OOP(面向对象编程)中，实现这样的功能我们需要通过继承或者实现接口，这样导致了代码的耦合程度增强，随着更多的行为产生，这些类会越来越臃肿。

Spirng 提供支持 `AspectJ(静态植入，在编译时将定义的切点方法进行重构，在指定的位置插入特定的代码)` 的注解式切面编程：
（1） 使用@Aspect声明一个切面，使用@Component注入到容器中。
（2） @Pointcut定义切点(可以定义为注解`@Pointcut("@annotation(*)")`，也可以定义为方法`@Pointcut("@execution(*)")`)
（3） 围绕切点使用@Before、@After、@Around与@AfterReturning来定义advice

### Spring的声明周期


![Spring的声明周期](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190722095148.jpg)

- 容器启动后，对bean进行初始化，按照bean的定义，注入属性
- 检测该bean是否存在XXXAware接口，并将相关XXXAware实例注入给bean，自此，该bean已正确构造。如：BeanNameAware、BeanFactoryAware、MessageSourceAware、ResourceLoaderAware、ApplicationEventPublisherAware
- 检测是否实现了BeanPostProcessor接口，进行一些自定义**前处理**，如：postProcessBeforeInitialzation。
- 调用@postConstruct、afterPropertiesSet,init-method等自定义方法。
- 检测是否实现了BeanPostProcessor接口，进行一些自定义**后处理**，如：postProcessAfterInitialzation。
- 经过以上步骤，Bean及可用。
- 容器关闭时，如果Bean实现了DisposableBean接口，则会回调该接口的destroy()方法


## SpringBoot

随着Spring的发展，Spring越来越复杂。当我们启动一个新的Spring项目时，我们必须添加大量构建路径或添加Maven依赖关系，配置应用程序服务器，添加spring配置。而SpringBoot就是为了解决复杂的Spring配置出现的。SpringBoot建立在Spring架构之上，但是比Spring更简单更快捷。

SpringBoot 的理念是 `约定大于配置`，这意味着一方面SpringBoot是可以像Spring一样通过配置来控制应用程序的，另一方面也说明了SpringBoot本身是提供了大量的默认配置(这些配置本身是很少改变的，即**约定好的惯例**)。

优点：

- 减少开发，测试时间和努力。
- 使用JavaConfig有助于避免使用XML。
- 避免大量的Maven导入和各种版本冲突。
- 通过提供默认值快速开始开发，及**惯例大于配置**。
- 没有单独的Web服务器需要，内嵌了Tomcat、Jetty和Undertow。
- 需要更少的配置 因为没有web.xml文件。只需添加用@ Configuration注释的类，然后添加用@Bean注释的方法，Spring将自动加载对象并像以前一样对其进行管理。您甚至可以将@Autowired添加到bean方法中，以使Spring自动装入需要的依赖关系中。
- 基于环境的配置，可以将正在使用的环境传递到应用程序：-Dspring.profiles.active = {enviornment}。在加载主应用程序属性文件后，Spring将在（application{environment} .properties）中加载后续的应用程序属性文件。

### 资源的调用

使用注解@Value，可以调用普通文件、网址、配置文件和环境变量等各种资源。

（1）注入字符         @value("字符串")
（2）注入操作系统属性  @Value("#{systemProperties['os.name']}")
（3）注入表达式结果    @Value("#{T(java.lang.Math).random()}")
（4）注入文件          @Value("文件路径")
（5）注入网址          @Value("网址")
（7）注入属性          @Value("${配置文件属性}")

### 事件(Application Event)

Application Event为Bean之间的消息通信提供了支持，当一个Bean处理完一个任务后，希望另一个Bean知道并进行相应的操作，这个时候我们就可以让另外一个Bean监听当前Bean所发送的时间。
这种机制存在的意义就是将Bean与Bean的关系进行解耦。

流程如下：
1.自定义事件  extends ApplicationEvent
2.定义事件监听 implements ApplicationListener<自定义事件>
3.定义事件发布类 该类通过注入 @Autowired ApplicationContext 然后ApplicationContext.publishEvent(new 自定义事件())，

```java
//事件
@Getter
@Setter
public class PayCompletedEvent extends ApplicationEvent {
    private Long payRecordId;

    public PayCompletedEvent(Object source, Long payRecordId) {
        super(source);
        this.payRecordId = payRecordId;
    }
}
//监听器
@Component
public class PayCompletedEventListener {
    private static Map<Long,SseEmitter> sseEmitters = new Hashtable<>();

    public void addSseEmitters(Long payRecordId, SseEmitter sseEmitter) {
        sseEmitters.put(payRecordId, sseEmitter);
    }

    @EventListener
    public void deployEventHandler(PayCompletedEvent payCompletedEvent) throws IOException {
        Long payRecordId = payCompletedEvent.getPayRecordId();
        //检验是否支付成功
        ...
        SseEmitter sseEmitter = sseEmitters.get(payRecordId);
        sseEmitter.send("支付成功");
        sseEmitter.complete();
    }
}
```

### Aware

在项目开发中，我们不可避免要在**普通的类中**用到Spring容器本身的功能资源，这时候Bean本身要意识到容器的存在，才能调用其中的资源。

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


```java
@Component
public class SpringBootBeanUtils implements ApplicationContextAware {
    private static ApplicationContext applicationContext;
    public static Object getBean(String name) {
        return getApplicationContext().getBean(name);
    }
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (SpringBootBeanUtils.getApplicationContext() == null) {
            SpringBootBeanUtils.applicationContext = applicationContext;
        }
    }
    public static <T> T getBean(Class<T> clazz) {
        return getApplicationContext().getBean(clazz);
    }
    public static <T> T getBean(String name, Class<T> clazz) {
        return getApplicationContext().getBean(name, clazz);
    }
}
```

### 定时任务

Spring为用户提供了定时任务的功能。`@Scheduled`定义在指定方法上，可以使该方法定时执行。

```java
@Configuration
@EnableScheduling
public class SchedulerConfig {
    @Scheduled (cron = "0 0 3 * * ?")
    public void Schedule(){
        System.out.println("凌晨三点对你思念是一天又一天");
    }
}
```

### 多线程和异步调用

在传统的应用中，为了实现异步调用，我们需要自己定义当前线程之外的另一个线程，`new Thread(()-> System.out.println("hello world !"))`。
在Spring中我们利用`@EnableAsync和@Async`就可以使用异步调用了。当然为了线程的使用恰当合理，我们需要在Spring中引入多线程池。

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

@Component
public class AsyncTaskComponent {
    @Async
    public void executeAsyncTask(Integer integer){
        System.out.println("执行异步任务："+integer);
    }

    @Async
    public Future<String> executeAsyncTaskPlus(Integer integer){
        System.out.println("执行异步任务："+(integer+1));
      	return new AsyncResult<String>("有返回值的异步任务");
    }
}
```

### Enable***总结

- @EnableAspectJAutoProxy 开启对AspectJ自动代理的支持
- @EnableAsync 开启异步方法的支持
- @EnableScheduling 开启计划任务的支持
- @EnableWebMvc 开启Web MVC的配置支持
- @EnableConfigurationProperties 开启对@ConfigurationProperties注解配置Bean的支持
- @EnableJpaRepositories 开启对Spring Data JPA repository的支持
- @EnableTransactionManagement 开启注解式事务的支持
- @EnableCaching  开启注解式的缓存支持


### RestTemplate

`RestTemplate`简化了发起HTTP请求以及处理响应的过程。

- delete
- exchange
- execute
- getForEntity
- getForObject
- postForEntity
- postForObject
- headForHeaders
- optionsForAllow
- postForLocation
- put

这里我们将请求分为了两类`Entity`和`Object`

#### Entity

返回值类型为`ResponseEntity`,它封装了返回的响应信息，包括响应状态，响应头和响应体。

#### Object

返回值类型仅为你需要的`body`。

## SpringMVC

SpringMVC是对于Model-View-Controller的实现，可以理解为完成了URL和一个类的方法的映射关系，可以返回一个ModelAndView或者一个数据，其核心类是DispatcherServlet，实现了Servlet接口。

![MVC模式原理图](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181016211718.jpg)


### 工作原理

我们要处理的问题包括：URL映射、参数获取、格式转换、返回值处理、视图渲染问题

- （1）客户端（浏览器）发送请求，利用DispatcherServlet(extends HttpServlet)，调用this.doDispatch(request,response)。

- （2）DispatcherServlet 根据请求信息 调用this.getHandler(请求)，获取Handler

- （3）解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，调用 this.getHandlerAdapter(Handler),创建 HandlerAdapter 适配器。 在适配器真正工作之前，我们需要进行 Handler.applyPreHandle, 对拦截器interceptors 进行处理，当然在适配器工作之后，也会有  Handler.applyPostHandle。

- （4）HandlerAdapter 会根据 Handler 来调用真正的处理器来处理请求ha.handle()，处理相应的业务逻辑。
   具体来说就是通过反射invoke，获取returnValue。

```java
    @Nullable
    public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
        Object[] args = this.getMethodArgumentValues(request, mavContainer, providedArgs);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Arguments: " + Arrays.toString(args));
        }

        return this.doInvoke(args);//Invoke方法就是我们写的@RestController中的方法 利用反射从handler中获取bean
    }
```

这里又必须注意Spring是如何将我们的request请求给转换成了method的参数，其中包括了参数的转换和参数的验证。
参数会有resolver.resolveArgument()进行解析:

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

如果参数被验证了话，还会调用validateIfApplicable(binder, parameter);

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

- （7）DispaterServlet 把返回的 Model 传给 View（视图渲染）, 或者直接将json对象传递给浏览器

- （8）把 View 返回给请求者（浏览器）

### 常用注解说明

1.定义在class上

@RestController 指明所有请求的方法都以JSON的形式返回给前台(前后端分离开发使用)
@RequestMapping (value = "/book") 指定根路由

2.定义在method上

@RequestMapping(value="",method=*,produces={"application/xml;charset=UTF-8"})
value是路径，method指明请求类型，produces表明返回的类型，consumes限定接受的类型

3.定义在parameter上

针对request：

  @RequestHeader 能够获取Http请求的header
  @RequestBody 能够获取Http请求的整个数据体     application/json application/xml application/x-www-form-urlencoded
  @RequestParam  能够获取query参数、post请求的key-value、post请求的文件(利用MultipartFile)
  @PathVariable
  HttpServletRequest 获取未包装的HttpRequest

### HttpMessageConverter

实现该接口的类，根据HTTP请求的mediaType将对应的InputStream转换为对应的类型。

![HttpMessageConverter](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/20181115185957.png)

我们可以通过定义自己的`HttpMessageConverter`，来实现特定的解析。

```java
public class CustomUdxConverter extends AbstractHttpMessageConverter<Object> {

    public CustomUdxConverter() {
        super(Charset.forName("UTF-8"), new MediaType("application", "udx"));
    }

    @Override
    protected boolean supports(Class aClass) {
        return true;
    }

    @Override
    protected Object readInternal(Class aClass, HttpInputMessage httpInputMessage) throws IOException, HttpMessageNotReadableException {
        String s = StreamUtils.copyToString(httpInputMessage.getBody(), Charset.forName("UTF-8"));
        return s;
    }

    @Override
    protected void writeInternal(Object o, HttpOutputMessage httpOutputMessage) throws IOException, HttpMessageNotWritableException {
        httpOutputMessage.getBody().write("hello,udx for text".getBytes());
    }
}

//添加的Converter集合中
@Configuration
public class SpringMvcConfig implements WebMvcConfigurer {
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new CustomUdxConverter());
    }
}

```



### SSE

服务器端推送(Server Send Event),就是在客户端和服务端建立一个长连接，服务器端可以推送数据给客户端。
SSE是建立不断开的长连接，然后客户端会自动请求，比长轮询的效率更高，可以认为是简化版的websocket。

>需要把response的类型 改为 text/event-stream，才是sse的类型

```java
@RequestMapping ("/sse")
@RestController
public class SSEController {
    @RequestMapping (value = "push", produces = "text/event-stream;charset=UTF-8")
    public JsonResult push() {
        return ResultUtils.success(Math.random());
    }
}
```

>前端也需要使用对应的HTML5长连接

```javascript
    let source=new EventSource('sse/push');
    source.onmessage= (event)=> {
        console.log(event);
    }
```

我们这里结合上面Event 实现一个更复杂的逻辑：以支付宝付款为案列

- 客户端发送请求push，检验是否付款成功，客户端处于监听状态。
- 客户端打开新的支付页面，进行付款。
- 付款成功，发布付款成功事件
- push请求监听付款成功事件，当成功时，返回到客户端。
- 客户端得到响应。
  
```java
@RequestMapping ("/sse")
@RestController
public class SSEController {
    @Autowired
    ApplicationContext applicationContext;
    @Autowired
    PayCompletedEventListener payCompletedEventListener;


     @GetMapping("/push")
    public SseEmitter push(@RequestParam Long payRecordId){
        final SseEmitter emitter = new SseEmitter();
        try {
           payCompletedEventListener.addSseEmitters(payRecordId,emitter);
        }catch (Exception e){
            emitter.completeWithError(e);
        }
        return emitter;
    }

    @GetMapping("/pay-callback")
    public String payCallback(@RequestParam Long payRecordId){
        applicationContext.publishEvent(new PayCompletedEvent(this,payRecordId));
        return "付款事件促发成功";
    }
}
```



### WebSocket

WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。
Spring也提供了对应的策略。

## SpringCloud

微服务是最近特别火的概念：使用定义好边界的小的独立组件来做好一件事情。使用微服务不可避免的需要将功能按照边界拆分为单个服务，体现分布式的特征。
这个时候**每个微服务之间的通信**就是我们需要解决的问题。

Spring Cloud大致分为以下的几个模块：

服务治理和发现：Eureka
客户端负载均衡： Spring Cloud Ribbon
服务容错保护： Spring  Cloud Hystrix  
声明式服务调用： Spring  Cloud Feign
API网关服务：Spring Cloud Zuul
分布式配置中心： Spring Cloud Config

![功能划分](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/20181126103717.png)




### 服务发现和治理：Eureka

当我们将系统拆分为多个子系统之后，那么我们如何解决子系统之间的`通讯`问题呢？
子系统与子系统不再同一个环境下，那就需要`远程调用`。我们考虑可以使用HttpClient建立请求,在Spring中我们可以利用Spring封装的[RestTemplate](#RestTemplate)。
但是我们要使用远程调用可能就必须要知道ip地址。当IP地址改变的时候，我们就要手动更新IP地址。在服务较多的情况下，我们手动的维护这些静态IP配置就是噩梦。

>为了解决微服务架构中的服务实例维护问题(ip地址)， 产生了大量的服务治理框架和产品。 这些框架和产品的实现都围绕着服务注册与服务发现机制来完成对微服务应用实例的自动化管理。

`Eureka`就是一种服务注册、管理中心。

![服务注册中心](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181124152013.jpg)

注意可能这里每个服务可能注册多个，那么一个服务就可以由多个服务器提供服务，当用户使用时，要根据服务器的使用情况，进行动态负载均衡。(Ribbon提供，由用户选择)

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

### 负载均衡：Ribbon

为了实现服务的高可用性，我们可以提供服务集群。对同一个服务，利用多台服务器来承载以提高并发访问量，利用多台服务器`合理分摊`用户的请求。

这个时候我们可以使用`nginx`进行负载均衡，这是一种`服务端的负载均衡`。服务实例的清单在服务器端，在服务器进行负载均衡算法分配。当然SpringCloud也提供了负载均衡的功能，只不过它是一种`客户端的负载均衡`。

这个功能的实现就是Ribbon。服务实例的清单在客户端(从Eureka服务注册中心获取)，客户端进行负载均衡算法分配。客户端可以从Eureka Server中得到一份服务清单，在发送请求时通过负载均衡算法，在多个服务器之间选择一个进行访问。

Ribbon的默认负载均衡策略是轮询，我们也可以对其进行自定义。另外SpringCloud在CAP理论中，是选择 可用性和分区容错性。

### 服务容错保护：Hystrix

通过`Eureka`和`Ribbon`，我们就能够根据服务名来远程调用其他服务，可以实现服务端的负载均衡。

但是如果在调用某个功能时(该功能可能由多个服务组成)，如果某个服务出了问题，整个功能就会卡死。在高并发的情况下，单个服务的延迟，可能导致所有的请求都出现延迟，而服务器同时支持的线程是有限的。这就会在几秒之内使服务器处于负载饱满的状态，直到该服务不可用，导致依赖该服务的关联服务也不可用，最终导致这个分布式系统都不可用，这就是"雪崩"。

`Hystrix`就实现了断路器、线程隔离等一系列服务保护功能。

- Fallback：但某个服务单元发生故障之后，通过断路器的故障监控，向调用方返回一个错误响应，而不是长时间的等待。这样就不会使得线程因调用故障服务被长时间占用不释放，避免了故障在分布式系统中的蔓延。
- 线程隔离：它会为每一个依赖服务创建一个独立的线程池，这样就算某个依赖服务出现延迟过高的情况，也只是对该依赖服务的调用产生影响， 而不会拖慢其他的依赖服务。

Hystrix提供几个熔断关键参数：滑动窗口大小（20）、 熔断器开关间隔（5s）、错误率（50%）

1.每当20个请求中，有50%失败时，熔断器就会打开，此时再调用此服务，将会直接返回失败，不再调远程服务。
2.直到5s钟之后，重新检测该触发条件，判断是否把熔断器关闭，或者继续打开。

Hystrix还有请求合并、请求缓存这样强大的功能。

### 声明式服务调用：Feign

`RestTemplate`是一种很好的远程服务调用方式，但是我们可以很明确的感知到，我们在调用远程的服务。

>而`Feign`是一种声明式、模板化的HTTP客户端。在Spring Cloud中使用Feign, 我们可以做到使用HTTP请求远程服务时能与调用本地方法一样的编码体验，开发者完全感知不到这是远程方法，更感知不到这是个HTTP请求。

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

### Zuul

在使用了以上的技术之后，我们现在的架构设计会是这个样子的：

![服务架构](https://raw.githubusercontent.com/sunlingzhiliber/imgstore/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181126102817.jpg)

但这种架构存在一些问题:

1.`路由规则与服务实例的维护间题`:外层的负载均衡(nginx)需要维护所有的Open服务实例清单。(upstream模块中需要维护每个服务实例的IP)

2.`签名校验、登陆校验冗余问题`：为了保证对外服务的安全性，我们在服务器端实现的微服务接口(非OpenService)，都会进行一定的权限校验机制，但是我们的服务是独立的，我们不得不`在这些应用中都实现这样一套校验逻辑`，这样就形成了代码冗余。

为了解决这些问题，`API网关的概念`应用而生。
在SpringCloud中了提供了基于Netfl ix Zuul实现的API网关组件`Spring Cloud Zuul`。

- SpringCloud Zuul通过与SpringCloud Eureka进行整合，将自身注册为Eureka服务治理下的应用，同时从Eureka中获得了所有其他微服务的实例信息。外层调用都必须通过API网关，使得将维护服务实例的工作交给了服务治理框架自动完成。
- 在API网关服务上进行统一调用来对微服务接口做前置过滤，以实现对微服务接口的拦截和校验。

### Config

随着业务的扩展，我们的服务会越来越多。每个服务都有自己的配置文件。
在分布式系统中，某一个服务信息的变更，都会引起一系列的更新与重启。因此我们需要一个解决分布式系统的配置管理方案。`Spring Cloud Config`

它包含了Client和Server两个部分，server提供配置文件的存储、以接口的形式将配置文件的内容提供出去，client通过接口获取数据、并依据此数据初始化自己的应用。

- 简单来说，使用Spring Cloud Config就是将配置文件放到统一的位置管理(比如GitHub)，客户端通过接口去获取这些配置文件。
- 在GitHub上修改了某个配置文件，应用加载的就是修改后的配置文件。

### Sleuth+Zipkin

将所有的请求数据记录下来，方便我们进行后续分析和获取额外的收益。

## Spring Data JPA

Spring Data JPA是Spring Data家族的一部分，可以轻松实现基于JPA的存储库。 此模块处理对基于JPA的数据访问层的增强支持。 它使构建使用数据访问技术的Spring驱动应用程序变得更加容易。
SpringData为我们使用统一的API来对不同的数据存储技术进行数据访问操作提供了支持。无论是关系数据库还是非关系数据库都可以以统一的标准进行访问。
该标准包括CURD、查询、排序和分页。
另外，它还可以根据**属性名对数据库中的实体进行操作，不用写实现，只用声明**,返回的结果我们可以使用Optional、VO、Page、Slice、Stream包装。
当然也可以使用@Query注解实现原生的查询。

[参考](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)

使用Spring Data JPA，我们只需要一个接口，继承JpaRepository即可。

```java
interface UserRepository extends CrudRepository<User, Long> {
  // Enables the distinct flag for the query
  List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
  List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);
  // Enabling ignoring case for an individual property
  List<Person> findByLastnameIgnoreCase(String lastname);
  // Enabling ignoring case for all suitable properties
  List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);
  // Enabling static ORDER BY for a query
  List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
  Slice<Person> findByLastnameOrderByFirstnameDesc(String lastname);
  Page<User> findByLastname(String lastname, Pageable pageable);

   //mongo
   @Query("{ 'age' : { $gt: ?0, $lt: ?1 } }")
   List<User> findUsersByAgeBetween(int ageGT, int ageLT);
   @Query(value="{ 'firstname' : ?0 }",fields="{ 'firstname' : 1, 'lastname' : 1}")
   //JPQL
   @Query("select u from User u")
   @Query("select u from User u where u.emailAddress = ?1")
   @Query("select u from User u where u.firstname like %?1")
   //SQL 
   @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
}
```

另外Spring Data JPA提供了Example的查询方式

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


## Spring Security




## Spirng Batch

## 注解大全

@SpringBootApplication：包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。其中@ComponentScan让spring Boot扫描到Configuration类并把它加入到程序上下文。

@Configuration 等同于spring的XML配置文件；可结合@Bean使用

@EnableAutoConfiguration 自动配置。

@ComponentScan 组件扫描，可自动发现和装配一些Bean。

@Component可配合CommandLineRunner使用，在程序启动后执行一些基础任务。

@RestController注解是@Controller和@ResponseBody的合集,表示这是个控制器bean,并且是将函数的返回值直 接填入HTTP响应体中,是REST风格的控制器。

@Autowired自动导入。

@PathVariable获取参数。

@RequestMapping：@RequestMapping(“/path”)表示该控制器处理所有“/path”的URL请求。RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。
用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。该注解有六个属性：
   params:指定request中必须包含某些参数值是，才让该方法处理。
   headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。
   value:指定请求的实际地址，指定的地址可以是URI Template 模式
   method:指定请求的method类型， GET、POST、PUT、DELETE等
   consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html;
   produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回

@RequestParam：用在方法的参数前面。
String a =request.getParameter(“a”)。

@JsonBackReference解决嵌套外链问题。

@RepositoryRestResourcepublic配合spring-boot-starter-data-rest使用。


@RequestMapping：提供路由信息，负责URL到Controller中的具体函数的映射。

@Import：用来导入其他配置类。

@ImportResource：用来加载xml配置文件。

@Autowired：自动导入依赖的bean,根据类型自动装配，是Spring提供的注解。

@Qualifier：当有多个同一类型的Bean时，可以用@Qualifier(“name”)来指定。与@Autowired配合使用。@Qualifier限定描述符除了能根据名字进行注入，但能进行更细粒度的控制如何选择候选者。

```java
@Autowired 
@Qualifier(value = “demoInfoService”) 
private DemoInfoService demoInfoService;
```

@Resource(name=”name”,type=”type”)：没有括号内内容的话，默认byName。与@Autowired干类似的事。属于J2EE，由JSR(java 规范 提案)-250提供，减少了和spring的耦合。

@Service：一般用于修饰service层的组件

@Repository：使用@Repository注解可以确保DAO或者repositories提供异常转译，这个注解修饰的DAO或者repositories类会被ComponetScan发现并配置，同时也不需要为它们提供XML配置项。

@Bean：用@Bean标注方法等价于XML中配置的bean。

@Value：注入Spring boot application.properties配置的属性的值。

@Inject：由JSR-330提供，等价于默认的@Autowired，只是没有required属性，功能较少。

@Component：泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

@ControllerAdvice：包含@Component。可以被扫描到。统一处理异常。

@ExceptionHandler（Exception.class）：用在方法上面表示遇到这个异常就执行以下方法。

--------------------JPA-------------------------
@Entity：@Table(name=”“)：表明这是一个实体类。一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略

@MappedSuperClass:用在确定是父类的entity上。父类的属性子类可以继承。

@NoRepositoryBean:一般用作父类的repository，有这个注解，spring不会去实例化该repository。

@Column：如果字段名与列名相同，则可以省略。

@Id：表示该属性为主键。

@GeneratedValue(strategy = GenerationType.SEQUENCE,generator = “repair_seq”)：表示主键生成策略是sequence（可以为Auto、IDENTITY、native等，Auto表示可在多个数据库间切换），指定sequence的名字是repair_seq。

@SequenceGeneretor(name = “repair_seq”, sequenceName = “seq_repair”, allocationSize = 1)：name为sequence的名称，以便使用，sequenceName为数据库的sequence名称，两个名称可以一致。

@Transient：表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic。@Basic(fetch=FetchType.LAZY)：标记可以指定实体属性的加载方式

@JsonIgnore：作用是json序列化时将Java bean中的一些属性忽略掉,序列化和反序列化都受影响。

@JoinColumn（name=”loginId”）:一对一：本表中指向另一个表的外键。一对多：另一个表指向本表的外键。

@OneToOne、@OneToMany、@ManyToOne：对应hibernate配置文件中的一对一，一对多，多对一。

--------------------Mongodb-------------------------

@Document 标明该类映射的集合名；类似@Entity注解

@Id  主键 不可重复，自动维护

@Indexed(Unique=true)  索引，加索引后以该字段为条件检索将大大提高速度。
对数组进行索引，MongoDB会索引这个数组中的每一个元素。
对整个Document进行索引，排序是预定义的按插入BSON数据的先后升序排列。
对关联的对象的字段进行索引，譬如User对关联的address.city进行索引

@CompoundIndex ： 联合索引 加复合索引后通过复合索引字段查询将大大提高速度。

```java
@Document(collection="system_user") 
// userName和age将作为复合索引
// 数字参数指定索引的方向，1为正序，-1为倒序
@CompoundIndexes({
    @CompoundIndex(name = "userName_age_idx", def = "{'userName': 1, 'age': -1}")
})
public class User{
    @Id
    private String id;
    private String userName;
    private String age;
    
}
```

@Field ： 属性 能够设置参数的key值

@Transient： 属性 忽略记录到数据库中

@DBRef :关联另一个document对象。类似于mysql的表关联，但并不一样，mongo不会做级联的操作。 
```java
public class School{
    @Id
    private String id;
    private String name;
    @DBRef
    private List<Student> students;
}
```

比如我们建立三个学生，然后一个学校，将学生赋值给学生list，执行学校的插入操作
**在不加@DBRef的情况下**
学校的list会直接存储学生的全部信息，当然也不会为学生建Document。(不同于sql的级联存储)

**在使用@DBRef的情况下**
学校的list中不会直接存储所有属性，而只是存储了学生的id和namespace,注意这里也不会为学生创建Document。
不会级联保存，会导致exception，因此需要单独处理关联的对象

```java
org.springframework.data.mapping.MappingException: Cannot create a reference to an object with a NULL id.
```


同理在删除学校时，关联的学生并不会删除。同理删除学生，学校也不会更改。

从这里来看，这个@DBRef是比较鸡肋的，甚至会以SQL的思维误导大家。

使用情形大致是为了避免单个document过大，而模拟sql外键的作用。

