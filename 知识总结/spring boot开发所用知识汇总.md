# Spring Boot开发所用知识汇总
---
layout: post
title: "SpringBoot 的各种乱七八糟"
date: 2021-05-28 11:37
comments: true
tags: 
	- SpringBoot
	- 知识总结

---
Spring Boot开发所需知识繁琐复杂，知识学的差不多在想要进行实际项目练手时，写篇文章总结一下近期的一些学习心得，也作为一个备忘录能够及时回顾。
## 1. web服务原理
### 1.1 Servlet原理
图片源自[狂神的JavaWeb入门到实战](https://www.bilibili.com/video/BV12J411M7Sj?p=9)
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200803165223.png" alt="servelet" style="zoom:50%;" />

### 1.2 MVC架构
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200904160121.png" alt="MVC" style="zoom:50%;" />
**M**odle

- 业务处理（Service）
- CRUD持久层  

**V**iew
- 展示数据
- 提供链接发起Servlet请求

**C**ontroller

- 接收用户请求
- 交给业务层对应代码
- 控制视图跳转

## 2. Cookie & Session
二者的区别：
- `session`存储空间比`cookie`大。
- `session`存储在服务器端，而`cookie`则存储在客户端，你可以在C盘中找到本机存储的`cookie`。

所以一般使用`cookie`来存储`session`的`id`，相当于`cookie`是打开`session`这个保险柜的钥匙。
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200825232221.png" alt="cookie" style="zoom: 50%;" />
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200825233220.png" alt="session" style="zoom: 49%;" />

## 3. 服务器
通常来讲，只要运行在服务器系统之上，绑定了服务器IP地址并且在某一个端口监听用户请求并提供服务的软件都可以叫服务器软件，其更像是一个容器的概念，包含了请求所需的资源容器。
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200803165636.png" alt="web服务原理" style="zoom: 50%;" />
图片源自[狂神的JavaWeb入门到实战](https://www.bilibili.com/video/BV12J411M7Sj?p=9)

目前能使用到的Web服务器就是`Nginx`和`Tomcat`，接下来就这两个软件分开说一下。
### 3.1 Tomcat
`Tomcat`是`Sping Boot`内嵌的的默认应用服务器或者说应用容器，应用服务器可以理解为特定应用的承载容器，运行时需要环境支持，`Tomcat`就是需要java环境支持。
### 3.2 Nginx
`Nginx`是一个典型的**HTTP服务器**，它原本的本职工作就是将服务端的某一个静态内容或资源通过`HTTP`协议传到客户端，所以也就是典型的静态服务器。

现实应用部署场景中，`Nginx`一般是与后面真正的动态应用服务器打配合，比如`Tomcat`，把用户请求转发给后面的应用服务器，从而提供灵活稳定的Web服务，而这种转发就是所谓的**反向代理**。因为`Nginx`服务器性能好，稳定性也高，能扛得住冲击，把它放在前面去直面用户。
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/Nginx.jpg" alt="Nginx" style="zoom: 33%;" />
那么为什么需要反向代理呢？因为`Nginx`在处理静态文件的吞吐量上面比`Tomcat`好很多，所以通常他们俩配合，不会把所有的请求都如本例所示的交给`Tomcat`, 而是把静态请求交给`Nginx`，动态请求，如`jsp`,`servlet`,`ssm`,`struts`等请求交给`Tomcat`. 从而达到**动静分离**的效果。

当访问量很大的时候，一个`Tomcat`吃不消了，这时候就准备多个`Tomcat`，由`Nginx`按照权重来对请求进行分配，从而缓解单独一个`Tomcat`受到的压力。这就叫做**负载均衡**。

通过负载均衡课程，我们可以把请求分发到不同的`Tomcat`来缓解服务器的压力，但是这里存在一个问题： 当同一个用户第一次访问`tomcat_8111`并且登录成功， 而第二次访问却被分配到了`tomcat_8222`， 这里并没有记录他的登陆状态，那么就会呈现未登录状态了，严重伤害了用户体验。

这里用`Redis`来存取`session`.

这样当`tomcat1`需要保存`session`值的时候，就可以把它放在`Redis`上，需要取的时候，也从`Redis`上取，这样就实现了**session共享**。
那么：
1. 用户提交账号密码的行为被分配在了`tomcat_8111`上，登陆信息被存放在`Redis`里。
2. 当用户第二次访问的时候，被分配到了`tomcat_8222`上
3. 那么此时`tomcat_8222`就会从`Redis`去获取相关信息，一看有对应信息，那么就会呈现登陆状态。
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/6660.png" alt="session共享" style="zoom:50%;" />

## 4. SpringBoot

### 4.1 IOC

IOC(Inversion of Control)，控制反转的核心思想在于，**资源（bean）的使用不由使用各自管理，而是交给不使用资源的第三方进行管理（容器）**。这样的好处是资源是集中管理的，可配置、易维护，同时也降低了双方的依赖度做到了低耦合。

### 4.2 AOP

AOP（Aspect-Oriented Programming：面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于減少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

Spring AOP 就是**基于动态代理**的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用Cgib，这时候 Spring AOP 会使用 Cgib 生成一个被代理对象的子类来作为代理。

#### 说一说代理模式

|                          | 接口实现     | InvocationHandler | CGLIB             |
| ------------------------ | ------------ | ----------------- | ----------------- |
| 种类                     | 静态代理     | 动态代理          | 动态代理          |
| 被代理类是否需要实现接口 | 是           | 是                | 否                |
| 代理类实现接口           | 抽象角色接口 | InvocationHandler | MethodInterceptor |

*动态代理中被代理类不需要实现抽象角色接口。

### 4.3 Spring MVC

<img src="https://gitee.com/wtychn/ImageBed/raw/master/image-20210621211028527.png" alt="image-20210621211028527" style="zoom:80%;" />

1. 客户端(浏览器)发送请求，直接请求到`DispatcherServlet`；
2. `DispatcherServlet`根据请求信息调用`HandlerMapping`，解析请求对应的`Handler`；
3. 解析到对应的`Handler`(也就是我们平常说的`Controller`控制器)后，开始由`HandlerAdapter`适配器处理；
4. `HandlerAdapter`会根据`Handler`来调用真正的处理器开处理请求,并处理相应的业务逻辑；
5. 处理器处理完业务后，会返回`ModelAndView`对象，`Model`是返回的数据对象，`View`是个逻辑上的 view；
6. `ViewResolver`会根据逻辑`View`查找实际的`View`；
7. `DispaterServlet`把返回的`Model`传给`View`(视图渲染)；
8. 把`View`返回给请求者(浏览器)。

<<<<<<< HEAD
### 4.5 SpringBoot 启动流程

<img src="https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190707171658626-1389392187.png" alt="img" style="zoom:100%;" />
=======
### 4.4 启动流程

<img src="https://gitee.com/wtychn/ImageBed/raw/master/image-20210625153912948.png" alt="image-20210625153912948" style="zoom:100%;" />

从 SpringBoot 的启动类开始分析：

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### 4.4.1 @SpringBootApplication

该注解又用到了如下注解：

```java
@Target(ElementType.TYPE) // 注解的适用范围，其中TYPE用于描述类、接口（包括包注解类型）或enum声明
@Retention(RetentionPolicy.RUNTIME) // 注解的生命周期，保留到class文件中（三个生命周期）
@Documented // 表明这个注解应该被javadoc记录
@Inherited // 子类可以继承该注解
@SpringBootConfiguration // 继承了Configuration，表示当前是注解类
@EnableAutoConfiguration // 开启springboot的注解功能，springboot的四大神器之一，其借助@import的帮助
@ComponentScan(excludeFilters = { // 扫描路径设置
@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
...
}　
```

其中`@SpringBootConfiguration`、`@EnableAutoConfiguration`、`@ComponentScan`三个注解最为重要

<img src="https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190709114128801-171612088.png" alt="img" style="zoom:80%;" />

##### @SpringBootConfiguration

该类继承自`@Configuration`注解，其作用是扫描配置类。SpringBoot 中主要使用 Config 配置类来解决配置问题。

@ComponentScan

1. `@ComponentScan`的功能其实就是自动扫描并加载符合条件的组件（比如`@Component`和`@Repository`等）或者bean定义；
2. 将这些 bean 定义加载到 **IoC** 容器中。

我们可以通过 basePackages 等属性来细粒度的定制`@ComponentScan`自动扫描的范围，如果不指定，则默认Spring框架实现会从声明`@ComponentScan`所在类的 package 进行扫描。

**注**：所以 SpringBoot 的启动类最好是放在 root package 下，因为默认不指定 basePackages。

##### @EnableAutoConfiguration

`@EnableAutoConfiguration`也是借助`@Import`的帮助，将所有符合自动配置条件的 bean 定义加载到 IoC 容器。

```java
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage【重点注解】
@Import(AutoConfigurationImportSelector.class)【重点注解】
public @interface EnableAutoConfiguration {
...
}
```

###### @AutoConfigurationPackage

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
}
```

通过`@Import(AutoConfigurationPackages.Registrar.class)`

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
        @Override
        public void registerBeanDefinitions(AnnotationMetadata metadata,
                BeanDefinitionRegistry registry) {
            register(registry, new PackageImport(metadata).getPackageName());
        }
		...
}
```

注册当前启动类的根 package；

注册`org.springframework.boot.autoconfigure.AutoConfigurationPackages`的 BeanDefinition。

######  @Import(AutoConfigurationImportSelector.class)（关键）

`AutoConfigurationImportSelector`实现了`DeferredImportSelector`从`ImportSelector`继承的方法：`selectImports()`。

1. Spring Boot 在启动时除了扫描与启动类同一包下的组件之外，还会检查各个 jar 包中是否存在 `META-INF/spring.factories` 文件，为自动装配做准备。
2. 第三方的 `spring-boot-starter` 会通过将自己的自动装配类写到 `META-INF/spring.factories` 中让 Spring Boot 加载到容器中，使自动装配类能够生效。
3. 第三方的自动装配类会通过利用 `@Conditional` 系列注释保证自己能在各种环境中成功自动装配。

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
    	return NO_IMPORTS;
    }
    AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
    			.loadMetadata(this.beanClassLoader);
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 从 jar 包下的 /META-INF/spring.factories 路径获取自动配置类列表
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = filter(configurations, autoConfigurationMetadata);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return StringUtils.toStringArray(configurations);
}
```

##### 自动装配原理

1. 整合JavaEE、解决方案、和自动配置所涉及的都在`spring-boot-autoconfigure-2.x.x.RELEASE.jar`包下；
2. `SpringBoot`在启动时，从类路径`/META-INF/spring.factories`下获取指定的类信息，JVM 就可以通过类加载器加载这些类；
3. 这样容器中就会导入很多命名为`xxxAutoConfigure`的类（`@Bean`），就是这些类给容器中导入了这个场景所需要的所有组件；
4. 给容器中自动配置类添加组件的时候，会从`xxxproperties`类中获取某些属性。我们只需要在配置文件中指定这些属性的值即可。

<img src="https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190708145522504-1677532764.png" alt="img" style="zoom:100%;" />

>`xxxAutoConfigurartion`: 自动配置类; 给容器中添加组件  
>`xxxProperties`: 封装配置文件中相关属性

## 5. 安全框架

安全框架主要用于登录、权限等操作。
### 5.1 Shiro

在使用Shiro 之前，大家做登录，权限什么的都是五花八门，各种花里胡哨的代码，不同系统的做法很有可能千差万别。

但是使用 Shiro 这个安全框架之后，大家做权限的方式都一致化了，这样的好处就是你的代码我看起来容易，我的代码你也好理解。

Shiro 也比较成熟，基本上能满足大部分的权限需要。

<img src="https://atts.w3cschool.cn/attachments/image/wk/shiro/2.png" alt="img" style="zoom:110%;" />

- **Subject**：主体，代表了当前 “**用户**”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是 Subject，如网络爬虫，机器人等；即一个抽象概念；所有 Subject 都绑定到 SecurityManager，与 Subject 的所有交互都会委托给 SecurityManager；可以把 Subject 认为是一个门面；SecurityManager 才是实际的执行者；

- **SecurityManager**：安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 Subject；可以看出它是 Shiro 的核心，它负责与后边介绍的其他组件进行交互，如果学习过 SpringMVC，你**可以把它看成 DispatcherServlet 前端控制器**；

- **Realm**：域，Shiro 从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色 / 权限进行验证用户是否能进行操作；**可以把 Realm 看成 DataSource**，即安全数据源。

也就是说对于我们而言，最简单的一个 Shiro 应用：

1. 应用代码通过 Subject 来进行认证和授权，而 Subject 又委托给 SecurityManager；
2. 我们需要给 Shiro 的 SecurityManager 注入 Realm，从而让 SecurityManager 能得到合法的用户及其权限进行判断。

## 6. 容器化
### 6.1 Docker 
常用基础命令
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200929100309.png" alt="docker命令" style="zoom:50%;" />

>引用文章：
- [服务器软件大盘点！ - CodeSheep - 掘金专栏](https://juejin.im/post/5e8409f9f265da47f144a07a#heading-12)
- [how2j教程](https://how2j.cn/)
- [狂神说Java](https://space.bilibili.com/95256449/channel/detail?cid=146244)