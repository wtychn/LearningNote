# Spring Boot开发所用知识汇总
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

### 4.3 自动装配原理
1. 整合JavaEE、解决方案、和自动配置所涉及的都在`spring-boot-autoconfigure-2.3.3.RELEASE.jar`包下   
2. `SpringBoot`在启动时，从类路径`/META-INF/spring.factories`下获取指定的值（以`url`形式存储）
3. 将这些自动配置类导入容器，将所有需要导入的组件以类名的方式返回，这些组件就会被添加到容器中
4. 容器中也会存在很多命名为`xxxAutoConfigure`的文件（`@Bean`），就是这些文件给容器中导入了这个场景所需要的所有组件
5. 给容器中自动配置类添加组件的时候,会从`xxxproperties`类中获取某些属性。我们只需要在配置文件中指定这些属性的值即可

>`xxxAutoConfigurartion`: 自动配置类; 给容器中添加组件  
>`xxxProperties`: 封装配置文件中相关属性
### 4.4 Spring MVC

<img src="https://gitee.com/wtychn/ImageBed/raw/master/image-20210621211028527.png" alt="image-20210621211028527" style="zoom:80%;" />

1. 客户端(浏览器)发送请求，直接请求到`DispatcherServlet`；
2. `DispatcherServlet`根据请求信息调用`HandlerMapping`，解析请求对应的`Handler`；
3. 解析到对应的`Handler`(也就是我们平常说的`Controller`控制器)后，开始由`HandlerAdapter`适配器处理；
4. `HandlerAdapter`会根据`Handler`来调用真正的处理器开处理请求,并处理相应的业务逻辑；
5. 处理器处理完业务后，会返回`ModelAndView`对象，`Model`是返回的数据对象，`View`是个逻辑上的 view；
6. `ViewResolver`会根据逻辑`View`查找实际的`View`；
7. `DispaterServlet`把返回的`Model`传给`View`(视图渲染)；
8. 把`View`返回给请求者(浏览器)。

## 5. 安全框架

安全框架主要用于登录、权限等操作。
### 5.1 Shiro
在使用Shiro 之前，大家做登录，权限什么的都是五花八门，各种花里胡哨的代码，不同系统的做法很有可能千差万别。

但是使用 Shiro 这个安全框架之后，大家做权限的方式都一致化了，这样的好处就是你的代码我看起来容易，我的代码你也好理解。

Shiro 也比较成熟，基本上能满足大部分的权限需要。

## 6. 容器化
### 6.1 Docker 
常用基础命令
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200929100309.png" alt="docker命令" style="zoom:50%;" />

>引用文章：
- [服务器软件大盘点！ - CodeSheep - 掘金专栏](https://juejin.im/post/5e8409f9f265da47f144a07a#heading-12)
- [how2j教程](https://how2j.cn/)
- [狂神说Java](https://space.bilibili.com/95256449/channel/detail?cid=146244)