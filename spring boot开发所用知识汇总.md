# Spring Boot开发所用杂项工具汇总
Spring Boot开发所需知识繁琐复杂，知识学的差不多在想要进行实际项目练手时，写篇文章总结一下近期的一些学习心得，也作为一个备忘录能够及时回顾。
## web服务原理
### Servlet原理
图片源自[狂神的JavaWeb入门到实战](https://www.bilibili.com/video/BV12J411M7Sj?p=9)
![servelet](https://gitee.com/wtychn/ImageBed/raw/master/img/20200803165223.png)
## 服务器
通常来讲，只要运行在服务器系统之上，绑定了服务器IP地址并且在某一个端口监听用户请求并提供服务的软件都可以叫服务器软件，其更像是一个容器的概念，包含了请求所需的资源容器。
![web服务原理](https://gitee.com/wtychn/ImageBed/raw/master/img/20200803165636.png)
图片源自[狂神的JavaWeb入门到实战](https://www.bilibili.com/video/BV12J411M7Sj?p=9)

目前能使用到的Web服务器就是`Nginx`和`Tomcat`，接下来就这两个软件分开说一下。
### Tomcat
`Tomcat`是`Sping Boot`内嵌的的默认应用服务器或者说应用容器，应用服务器可以理解为特定应用的承载容器，运行时需要环境支持，`Tomcat`就是需要java环境支持。
### Nginx
`Nginx`是一个典型的**HTTP服务器**，它原本的本职工作就是将服务端的某一个静态内容或资源通过`HTTP`协议传到客户端，所以也就是典型的静态服务器。

现实应用部署场景中，`Nginx`一般是与后面真正的动态应用服务器打配合，比如`Tomcat`，把用户请求转发给后面的应用服务器，从而提供灵活稳定的Web服务，而这种转发就是所谓的**反向代理**。因为`Nginx`服务器性能好，稳定性也高，能扛得住冲击，把它放在前面去直面用户。
![Nginx](https://gitee.com/wtychn/ImageBed/raw/master/img/Nginx.jpg)
那么为什么需要反向代理呢？因为`Nginx`在处理静态文件的吞吐量上面比`Tomcat`好很多，所以通常他们俩配合，不会把所有的请求都如本例所示的交给`Tomcat`, 而是把静态请求交给`Nginx`，动态请求，如`jsp`,`servlet`,`ssm`,`struts`等请求交给`Tomcat`. 从而达到**动静分离**的效果。

当访问量很大的时候，一个`Tomcat`吃不消了，这时候就准备多个`Tomcat`，由`Nginx`按照权重来对请求进行分配，从而缓解单独一个`Tomcat`受到的压力。这就叫做**负载均衡**。

通过负载均衡课程，我们可以把请求分发到不同的`Tomcat`来缓解服务器的压力，但是这里存在一个问题： 当同一个用户第一次访问`tomcat_8111`并且登录成功， 而第二次访问却被分配到了`tomcat_8222`， 这里并没有记录他的登陆状态，那么就会呈现未登录状态了，严重伤害了用户体验。

这里用`Redis`来存取`session`.

这样当`tomcat1`需要保存`session`值的时候，就可以把它放在`Redis`上，需要取的时候，也从`Redis`上取，这样就实现了**session共享**。
那么：
1. 用户提交账号密码的行为被分配在了`tomcat_8111`上，登陆信息被存放在`Redis`里。
2. 当用户第二次访问的时候，被分配到了`tomcat_8222`上
3. 那么此时`tomcat_8222`就会从`Redis`去获取相关信息，一看有对应信息，那么就会呈现登陆状态。
![session共享](https://gitee.com/wtychn/ImageBed/raw/master/img/6660.png)
## 安全框架
安全框架主要用于登录、权限等操作。
### Shiro
在使用Shiro 之前，大家做登录，权限什么的都是五花八门，各种花里胡哨的代码，不同系统的做法很有可能千差万别。

但是使用 Shiro 这个安全框架之后，大家做权限的方式都一致化了，这样的好处就是你的代码我看起来容易，我的代码你也好理解。

Shiro 也比较成熟，基本上能满足大部分的权限需要。

>引用文章：
- [服务器软件大盘点！ - CodeSheep - 掘金专栏](https://juejin.im/post/5e8409f9f265da47f144a07a#heading-12)
- [how2j教程](https://how2j.cn/)