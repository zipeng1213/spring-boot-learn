```
@Overridepublic void run() {    SpringApplication.run(BackendApplication.class, args);}
```

spring-boot的启动执行SpringApplication.run方法，BackendApplication类名作为参数

```
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {    return run(new Class[]{primarySource}, args);}
```

（String... java新加入的功能表示一个可变长度的参数列表 可以使用0个或多个参数）

```
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {    return (new SpringApplication(primarySources)).run(args);}
```

构造方法:

```java
public SpringApplication(Class<?>... primarySources) {    this((ResourceLoader)null, primarySources);}

public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {    this.sources = new LinkedHashSet();    this.bannerMode = Mode.CONSOLE;    this.logStartupInfo = true;    this.addCommandLineProperties = true;    this.addConversionService = true;    this.headless = true;    this.registerShutdownHook = true;    this.additionalProfiles = new HashSet();    this.isCustomEnvironment = false;    this.lazyInitialization = false;    this.resourceLoader = resourceLoader;    Assert.notNull(primarySources, "PrimarySources must not be null");    this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));    this.webApplicationType = WebApplicationType.deduceFromClasspath();    this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));    this.mainApplicationClass = this.deduceMainApplicationClass();}
```



初始化一个SpringApplication调用run方法

```java
public ConfigurableApplicationContext run(String... args) {    StopWatch stopWatch = new StopWatch();    stopWatch.start();    ConfigurableApplicationContext context = null;    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();    this.configureHeadlessProperty();    SpringApplicationRunListeners listeners = this.getRunListeners(args);    listeners.starting();    Collection exceptionReporters;    try {        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);        this.configureIgnoreBeanInfo(environment);        Banner printedBanner = this.printBanner(environment);        context = this.createApplicationContext();        exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);        this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);        this.refreshContext(context);        this.afterRefresh(context, applicationArguments);        stopWatch.stop();        if (this.logStartupInfo) {            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);        }        listeners.started(context);        this.callRunners(context, applicationArguments);    } catch (Throwable var10) {        this.handleRunFailure(context, var10, exceptionReporters, listeners);        throw new IllegalStateException(var10);    }    try {        listeners.running(context);        return context;    } catch (Throwable var9) {        this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null);        throw new IllegalStateException(var9);    }}
```

##### @Retention

注解用于指明修饰的注解的生存周期，即会保留到哪个阶段

- SOURCE：源码级别保留，编译后即丢弃。
- CLASS：编译级别保留，编译后的class文件中存在，在jvm运行时丢弃，这是默认值。
- RUNTIME：运行级别保留，编译后的class文件中存在，在jvm运行时保留，可以被反射调用。

##### @Documented

指明修饰的注解，可以被例如javadoc此类的工具文档化，只负责标记，没有成员取值

##### @Inherited

表示父类的注解是否可以被子类继承

##### @Target

注解用于定义注解的使用位置

- TYPE：类，接口或者枚举
- FIELD：域，包含枚举常量
- METHOD：方法
- PARAMETER：参数
- CONSTRUCTOR：构造方法
- LOCAL_VARIABLE：局部变量
- ANNOTATION_TYPE：注解类型
- PACKAGE：包

##### @SpringBootApplication

等价于@SpringBootConfiguration@EnableAutoConfiguration@ComponentScan

##### SpringBootConfiguration

对@Configuration进行一个包装

@configure注解对component注解进行包装

（@interface表示自定义注解）

##### @EnableAutoConfiguration

包含了两个Import的类 一个是EnableAutoConfigurationImportSelector.class还有一个是AutoConfigurationPackages.Registrar.class

##### @Configuration









##### 控制反转Inversion of Control

Class A使用到了Class B，但Class B不是被Class A创建，Class A只需声明要使用Class B，由**容器**创建Class B并注入A，创建Class B的控制权不在Class B的调用者Class A上，而是**容器**，这就是**控制反转**,classA不需要来创建这个classB只需要声明使用这个类就好了，由容器来创建并注入这个类（依赖注入），

##### 依赖注入Dependency Injection

解决对象之间的依赖关系由容器自动注入对象到申明的地方

查询对象的依赖关系出入依赖对象

IoC容器

管理对象，包括创建对象发布对象销毁对象

1. 声明bean的注解

- @Controller

作用在类上，使用注解配置和类路径扫描会被spring扫描注册为bean，为web中的控制层组件，单例模式

- @Service

业务层组件，没有设置参数时默认为类名表示Bean的名字，单例模式

- @Repository

单例模式，用于标注数据访问组件，Dao组件，它还能将所标注的类中抛出的数据访问异常封装为 Spring 的数据访问异常类型

- @Component

将简单的java对象（Javabean）注册为bean

- @Configuration+@bean

等价于将把类作为xml文件的<beans>，表示一个配置类@Bean作用于方法上表示产生一个bean对象交给spring对象管理放入IoC容器中

- @ComponentScan包扫描

默认扫描在application类目录下所有的类，@ComponentScan 的作用就是根据定义的扫描路径，把符合扫描规则的类装配到spring容器中

​		basePackages与value:  用于指定包的路径，进行扫描

​		basePackageClasses: 用于指定某个类的包的路径进行扫描

​		nameGenerator: bean的名称的生成器

​		useDefaultFilters: 是否开启对@Component，@Repository，@Service，@Controller的类进行检测

​		includeFilters: 包含的过滤条件

​            FilterType.ANNOTATION：按照注解过滤

​            FilterType.ASSIGNABLE_TYPE：按照给定的类型

​            FilterType.ASPECTJ：使用ASPECTJ表达式

​            FilterType.REGEX：正则

​            FilterType.CUSTOM：自定义规则

- @Import

快速给容器导入一个组件，容器中就会自动注册这个组件，id默认是全类名



pring为其生成beanName就是将短类名的首字母小写，当短类名的首字符和第二个字符均大写时，beanName就是短类名



@Scope用于配置bean的作用域

1. singleton单例模式 IoC容器中只有一个Bean实例，一直存活

2. prototype原型模式 每次获取bean都会创建一个新的实例
3. request模式只适用于Web程序，每次http请求都产生一个新的bean，结束后对象生命周期结束
4. session模式
5. application模式

Bean装配注解

- @Autowired

byType方式required属性设置当没有找到时是都报错

- @Qualifier

  当有多个类型的Bean时通过name来指示，和Autowired配合使用

- @Primary优先注入的Bean对象

- @Resource

默认按名称装配，如果不到与名称匹配的bean，会按类型装配。



##### servlet

 Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层

他就是运行在Web服务器中的程序，响应http请求实现用户自己的逻辑，将结果返回给用户，Web服务器收到客户端的http请求，会针对每一次请求，分别创建一个用于代表请求的request对象、和代表响应的response对象。

分别为下面两个

几个重要的结构

- HttpServletRequest

将http请求封装到一个HttpServletRequest对象中，

getHeader 获取请求头的头部信息

getCookies 获取当前请求的cookie信息

getMethod 获取当前请求方法类别  Get Post

getQueryString 获取当前请求参数的kv串  k1=v1=&k2=v2

getRequestURI  获取当前请求路径  /servlet/a

getRequestURL 获取当前请求的总路径，包含域名协议 http://localhost:8080/servlet/a

getSession 获取当前请求的session信息

getParameter 获取请求参数中对应key的valu

- HttpServletResponse

将http返回的信息封装到一个HttpServletResponse对象中

getOutputStream获取输出流，用来输出相应内容



##### @ResponseBody*

它的作用是将controller层中的方法的返回值对象转换成相应的格式，然后将其写入到HttpServletResponse响应体中，等同于将数据写到HttpServletResponse的输出流中。

controller的方法返回的对象通过适当的**转换器**转换为指定的格式之后，写入到response对象的body区，通常用来返回JSON数据或者是XML数据，需要注意的呢，在使用此注解之后不会再走试图处理器，而是直接将数据写入到输入流中，他的效果等同于通过response对象输出指定格式的数据。

##### @RequestBody*

将http请求中的body参数封装到javaBeans中（封装时使用系统默认配置的 **HttpMessageConverter**进行解析）

通常用在Controller参数中使用

##### ResponseEntity

ResponseEntity是通用类型，因此可以使用任意类型作为响应体

可以设置body，header，和相应码