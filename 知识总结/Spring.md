# Spring 八股大全

---

layout: post
title: "Spring 八股大全"
date: 2021-08-18 11:17
comments: true
tags: 

 - Spring
   - 知识总结

---

## 1. IOC

IOC(Inversion of Control)，控制反转的核心思想在于，**资源（bean）的使用不由使用各自管理，而是交给不使用资源的第三方进行管理（容器）**。这样的好处是资源是集中管理的，可配置、易维护，同时也降低了双方的依赖度做到了低耦合。



## 2. AOP

AOP（Aspect-Oriented Programming：面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于減少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

Spring AOP 就是**基于动态代理**的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 Cglib，这时候 Spring AOP 会使用 Cgib 生成一个被代理对象的子类来作为代理。

### 说一说代理模式

|                          | 接口实现     | InvocationHandler | CGLIB             |
| ------------------------ | ------------ | ----------------- | ----------------- |
| 种类                     | 静态代理     | 动态代理          | 动态代理          |
| 被代理类是否需要实现接口 | 是           | 是                | 否                |
| 代理类实现接口           | 抽象角色接口 | InvocationHandler | MethodInterceptor |

*动态代理中被代理类不需要实现抽象角色接口。

## 3. Spring

### 3.1 Bean 的创建过程

<img src="https://img2020.cnblogs.com/blog/1443749/202005/1443749-20200510013054067-1092537747.png" alt="img" style="zoom:90%;" />

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpwpib2icSAItVibAibWmFCLpLDUoO6veJkDmuNfzRHE6ZibibZiblichgFufY8C7oh5o4Xfu2oZicPVyIuD4Ww/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:82%;" />

### 3.2 Bean 的生命周期

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

实例化 -> 属性赋值 -> 初始化 -> 销毁

主要逻辑都在doCreate()方法中，逻辑很清晰，就是顺序调用以下三个方法。

1. createBeanInstance() -> 实例化
2. populateBean() -> 属性赋值
3. initializeBean() -> 初始化

详细步骤：

1. Spring 启动，查找并加载需要被 Spring 管理的 bean，进行 Bean 的实例化
2. Bean 实例化后对将 Bean 的引入和值注入到 Bean 的属性中
3. 如果 Bean 实现了 BeanNameAware 接口的话，Spring 将 Bean 的 Id 传递给 setBeanName() 方法
4. 如果 Bean 实现了 BeanFactoryAware 接口的话，Spring 将调用 setBeanFactory() 方法，将 BeanFactory 容器实例传入
5. 如果 Bean 实现了 ApplicationContextAware 接口的话，Spring 将调用 Bean 的 setApplicationContext() 方法，将 bean 所在应用上下文引用传入进来。
6. 如果 Bean 实现了 BeanPostProcessor 接口，Spring 就将调用他们的 postProcessBeforeInitialization() 方法。
7. 如果 Bean 实现了 InitializingBean 接口，Spring 将调用他们的 afterPropertiesSet() 方法。类似的，如果 bean 使用 init-method 声明了初始化方法，该方法也会被调用
8. 如果 Bean 实现了 BeanPostProcessor 接口，Spring 就将调用他们的 postProcessAfterInitialization() 方法。
9. 此时，Bean 已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
10. 如果 Bean 实现了 DisposableBean 接口，Spring 将调用它的 destory() 接口方法，同样，如果 bean 使用了 destory-method 声明销毁方法，该方法也会被调用。

### 3.3 Spring MVC

<img src="https://gitee.com/wtychn/ImageBed/raw/master/image-20210621211028527.png" alt="image-20210621211028527" style="zoom:80%;" />

1. 客户端(浏览器)发送请求，直接请求到`DispatcherServlet`；
2. `DispatcherServlet`根据请求信息调用`HandlerMapping`，解析请求对应的`Handler`；
3. 解析到对应的`Handler`(也就是我们平常说的`Controller`控制器)后，开始由`HandlerAdapter`适配器处理；
4. `HandlerAdapter`会根据`Handler`来调用真正的处理器开处理请求,并处理相应的业务逻辑；
5. 处理器处理完业务后，会返回`ModelAndView`对象，`Model`是返回的数据对象，`View`是个逻辑上的 view；
6. `ViewResolver`会根据逻辑`View`查找实际的`View`；
7. `DispaterServlet`把返回的`Model`传给`View`(视图渲染)；
8. 把`View`返回给请求者(浏览器)。

### 3.4 常用注解

#### 1）Spring Bean 相关

##### 3.4.1 `@Autowired`

自动导入对象到类中，被注入进的类同样要被 Spring 容器管理比如：Service 类注入到 Controller 类中。

```java
@Service
public class UserService {
  ......
}

@RestController
@RequestMapping("/users")
public class UserController {
   @Autowired
   private UserService userService;
   ......
}
```

当自动装配时，从容器中如果发现有多个同类型的属性时，`@Autowired`注解会先根据类型判断，然后根据`@Primary`、`@Priority`注解判断，最后根据属性名与 beanName 是否相等来判断，如果还是不能决定注入哪一个 bean 时，就会抛出`NoUniqueBeanDefinitionException`异常

**@Autowired 和 @Resource:** 

- @Autowired 默认按照 byType 方式进行 bean 匹配，@Resource 默认按照 byName 方式进行 bean 匹配
- @Autowired 是 Spring 的注解，@Resource 是 J2EE 的注解，这个看一下导入注解的时候这两个注解的包名就一清二楚了

##### 3.4.2 `@Component`,`@Repository`,`@Service`, `@Controller`

我们一般使用 `@Autowired` 注解让 Spring 容器帮我们自动装配 bean。要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,可以采用以下注解实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。

##### 3.4.3 `@RestController`

`@RestController`注解是`@Controller和`@`ResponseBody`的合集,表示这是个控制器 bean，并且是将函数的返回值直 接填入 HTTP 响应体中，是 REST 风格的控制器。

单独使用 `@Controller` 不加 `@ResponseBody`的话一般使用在要返回一个视图的情况，这种情况属于比较传统的 Spring MVC 的应用，对应于前后端不分离的情况。`@Controller` +`@ResponseBody` 返回 JSON 或 XML 形式数据

##### 3.4.4 `@Scope`

声明 Spring Bean 的作用域，使用方法:

```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

**四种常见的 Spring Bean 的作用域：**

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- session : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。

##### 3.4.5 `@Configuration`

一般用来声明配置类，可以使用 `@Component`注解替代，不过使用`Configuration`注解声明配置类更加语义化。

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

#### 2）前后端传值

##### 3.4.6 `@PathVariable` 和 `@Reque5stParam`

`@PathVariable`用于获取路径参数，`@RequestParam`用于获取查询参数。

举个简单的例子：

```java
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
         @PathVariable("klassId") Long klassId,
         @RequestParam(value = "type", required = false) String type ) {
...
}
```

如果我们请求的 url 是：`/klasses/{123456}/teachers?type=web`

那么我们服务获取到的数据就是：`klassId=123456,type=web`。

##### 3.4.7 `@RequestBody`

用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且 **Content-Type 为 application/json** 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用`HttpMessageConverter`或者自定义的`HttpMessageConverter`将请求的 body 中的 json 字符串转换为 java 对象。

#### 3）配置读取

我们的数据源`application.yml`内容如下：：

```yaml
wuhan2020: 2020年初武汉爆发了新型冠状病毒，疫情严重，但是，我相信一切都会过去！武汉加油！中国加油！

my-profile:
  name: Guide哥
  email: koushuangbwcx@163.com

library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```

##### 3.4.8 `@value`(常用)

使用 `@Value("${property}")` 读取比较简单的配置信息：

```java
@Value("${wuhan2020}")
String wuhan2020;
```

##### 3.4.9 `@ConfigurationProperties`(常用)

通过`@ConfigurationProperties`读取配置信息并与 bean 绑定。

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    @NotEmpty
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
  省略getter/setter
  ......
}
```

你可以像使用普通的 Spring bean 一样，将其注入到类中使用。

##### 3.4.10 `PropertySource`（不常用）

`@PropertySource`读取指定 properties 文件

```java
@Component
@PropertySource("classpath:website.properties")

class WebSite {
    @Value("${url}")
    private String url;

  省略getter/setter
  ......
}
```

## 4. SpringBoot

### 4.1 SpringBoot 启动流程

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

####  @SpringBootApplication

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

### 4.2 自动装配原理

1. 整合JavaEE、解决方案、和自动配置所涉及的都在`spring-boot-autoconfigure-2.x.x.RELEASE.jar`包下；
2. `SpringBoot`在启动时，从类路径`/META-INF/spring.factories`下获取指定的类信息，JVM 就可以通过类加载器加载这些类；
3. 这样容器中就会导入很多命名为`xxxAutoConfigure`的类（`@Bean`），就是这些类给容器中导入了这个场景所需要的所有组件；
4. 给容器中自动配置类添加组件的时候，会从`xxxproperties`类中获取某些属性。我们只需要在配置文件中指定这些属性的值即可。

<img src="https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190708145522504-1677532764.png" alt="img" style="zoom:100%;" />

>`xxxAutoConfigurartion`: 自动配置类; 给容器中添加组件  
>`xxxProperties`: 封装配置文件中相关属性

