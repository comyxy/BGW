# web 基础

## Spring

### Spring 有哪些模块？

* Core Container：核心容器，是 IoC 和 DI 的基础
  * Beans
  * Core
  * Context
  * SpEL
* AOP：提供了面向切面的编程实现
* Data Access/Integration：提供持久层支持
  * JDBC：提供 Java 数据库连接
  * ORM：提供 Hibernate 等 ORM 框架支持
  * ...
* Web：提供 Web 应用支持
  * Spring MVC
  * Servlet
  * WebSocket
  * ...
* ...
  
### Spring 生命周期

* 单例bean：整个生命周期的管理
* 实例bean：只负责创建
  
![bean-lifecycle](pic\bean-lifecycle.webp)
  
### 说一下AOP？

Aspect Oriented Programming，面向切片编程，将一些与主业务逻辑无关的业务（如日志管理、信息统计）从主业务中拿出来做成切面，在需要这些业务逻辑的时候通过切片织入的方式插入主业务逻辑中。代码层面上解除了两者的耦合关系，可以充分复用切片代码。  
实现方式上，通过JDK动态代理或 cglib 的字节码继承技术，将原业务替换为代理对象，执行相应业务。
  
### 说一下 IoC 和 DI？

Inverse of Control，控制反转，是一种设计思想，即将创建对象的操作由**使用对象者**交给 **Spring 框架**来管理，同时，对象间复杂的依赖关系也交给了 Spring 框架管理。当需要某个对象时，使用者只需向 Spring 框架请求这个对象，就可以获得一个已经创建好的对象，这个过程就是 Dependency Injection，即 Spring 框架将创建好的对象按一定匹配规则注入给使用者的过程。这样一来，对象使用者就不需要知道对象是如何被创建出来的，当对象修改时，只需要关心对象本身的修改，而不必担心使用对象的类会被牵一发而动全身。这种低侵入的特定，提高了项目的可维护性，降低了开发难度。
  
### bean 的作用域有哪些？

* singleton：单例，默认
* prototype：每次 getBean 请求创建新实例
* request：每次 HTTP 请求创建新实例，web 下有效
* session：每次 HTTP 会话创建新实例，web 下有效
* global-session：portlet 容器中可使用
  
### @Component 和 @Bean 的区别是什么？

* @Component 注解用于类，说明这个类需要作为一个 bean 注册到容器中
* @Bean 注解用于方法，说明这个方法会返回一个对象，这个对象需要作为一个 bean 注册到容器中
* @Bean 自定义了生成 bean 的方法，可通过传给同一个方法不同的参数获取不同的 bean，这是 @Component 所做不到的
  
### @Autowired 和 @Resource 的区别是什么？

* @Autowired 是 Spring 的注解，按类型匹配。若类型重复，可添加 @Qualifier 注解来指定具体**类名**
* @Resource 是 J2EE 的注解，优先按 name 匹配，可指定 name 和 type
  
### @Mapper 和 @Repository 的区别是什么？

* @Mapper 是 Mybatis 的注解，用于生成动态代理。可以通过 @MapperScan 注解扫描包，省去 @Mapper 注解。可以将 MapperScan 类交由 Spring 托管，将生成的 Mapper 作为 bean 注册到容器中
* @Repository 是 Spring 注解，用于注册 DAO 层 bean
  
### Spring MVC

1. 客户端发送 HTTP 请求到 DispatcherServlet。
2. DispatcherServlet 根据请求信息调用 HandlerMapping 来找到 Handler（Controller）。
   * SimpleUrlHandlerMapping 解析配置文件中定义的映射关系，返回 Controller 类
   * RequestMappingHandlerMapping 解析注解中定义的映射关系，返回具体方法
3. DispatcherServlet 调用 HandlerAdapter 处理上一步返回的 Handler，执行业务逻辑，并返回 ModelAndView。
4. DispatcherServlet 调用 ViewResolver 解析逻辑 View，返回 View 实例。
5. DispatcherServlet 将 Model 传给 View 并渲染，返回给客户端。
  
### SpringBoot 如何实现自动装配？

SpringBoot 定义了一套接口规范：启动时会扫描外部引用包中 `META-INF/spring.factories`文件，将其配置信息加载到 Spring 容器中。用户通过注解和一些简单的配置就可以实现功能。  
`@SpringBootApplication` 注解是 `@Configuration`（允许注册额外 bean）、`@EnableAutoConfiguration`（开启自动配置）、`@ComponentScan`（扫描 @Component）注解的集合  
开启自动配置后，`AutoConfigurationImportSelector` 类会通过注解的方式启动，来加载自动装配类。  
自动装配类的过程如下：

1. 判断是否开启自动装配
2. 获取 `@EnableAutoConfiguration` 中配置的 exclude 和 excludeName
3. 读取 `META-INF/spring.factories` 并获取需要自动装配的配置类全限定名
4. 根据 `@ConditionalOnXXX` 注解来筛选类是否能够加载，如：当容器中有指定 bean 时才加载，当项目为 web 项目时才加载等等
5. 加载
  
### Spring 同类未加 @Transactional 注解的方法调用了加 @Transactional 注解的方法，事务有效吗？

被调用方法的事务会失效。由于 Spring AOP 代理的原因，动态代理未改变被代理类的字节码，访问本类方法时不是通过代理实例来访问的，而是直接访问的。可通过避免同个对象自调用或织入代理实例再调用或使用 AspectJ 代理来解决，AspectJ 通过编译期静态代理修改字节码，实现自调用时的增强。
  
### Spring 中有哪些设计模式？

1. 工厂模式。BeanFactory、ApplicationContext。
2. 单例模式。Bean 的生命周期。
3. 代理模式。AOP。
4. 模板方法模式。JdbcTemplate 等。
5. 观察者模式。事件驱动模型，ApplicationContextEvent、ApplicationListener、ApplicationEventPublisher。
6. 适配器模式。spring mvc 的 HandlerAdapter。
7. ...
  
### cglib 的优点？

1. 不需要接口，通过字节码技术直接继承类。这也导致了 final 类和方法无法被代理。
2. 使用 FastClass 机制，执行效率高。代理类建立被代理类的各个方法的索引，调用方法时使用 `switch (index) case...` 的方式，避免了反射的低效率。建立索引的过程导致 cglib 代理初始化时时间会长一些。
  
## 参考资料  

<https://github.com/Snailclimb/JavaGuide>
