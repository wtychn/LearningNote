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
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200803165223.png" alt="servelet" style="zoom: 67%;" />

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
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200803165636.png" alt="web服务原理" style="zoom: 70%;" />
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

## 4. 安全框架

安全框架主要用于登录、权限等操作。
### 4.1 Shiro

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

## 5. 容器化
### 5.1 Docker 
常用基础命令
<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/20200929100309.png" alt="docker命令" style="zoom:50%;" />

## 7. Restful 风格架构

1. 每一个URI代表一种资源；
2. 客户端和服务器之间，传递这种资源的某种表现层；
3. 客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

注意：

- URI 中不包含动词，动词通过 HTTP 方法表达；
- 不同的版本，可以理解成同一种资源的不同表现形式，所以应该采用同一个 URI。版本号可以在 HTTP 请求头信息的 Accept 字段中进行区分。

