1. Spring 是什么? 特性? 有哪些框架?
**Spring 是一个轻量级、非入侵式的控制反转 (IoC) 和面向切面 (AOP) 的框架。**
2. Spring 有哪些特性
Spring的优点有很多:
IoC 和 DI 的支持,  AOP 编程的支持, 声明式事务的支持, 快捷测试的支持, 快速集成功能, 复杂 API 模板封装. 

- IoC 和DI的支持
Spring 的核心就是一个大的工厂容器，可以维护所有对象的创建和依赖关系，Spring 工厂用于生成 Bean，并且管理 Bean 的生命周期，实现**高内聚低耦合**的设计理念。

- AOP 编程的支持
Spring 提供了**面向切面编程**，可以方便的实现对程序进行权限拦截、运行监控等切面功能。

- 声明式事务的支持
支持通过配置就来完成对事务的管理，而不需要通过硬编码的方式，以前重复的一些事务提交、回滚的 JDBC 代码，都可以不用自己写了。

- 快捷测试的支持
Spring 对 Junit 提供支持，可以通过**注解**快捷地测试 Spring 程序。

- 快速集成功能
方便集成各种优秀框架，Spring 不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Struts、Hibernate、MyBatis、Quartz 等）的直接支持。

- 复杂API 模板的封装
Spring 对 JavaEE 开发中非常难用的一些 API（JDBC、JavaMail、远程调用等）都提供了模板化的封装，这些封装 API 的提供使得应用难度大大降低

3. Spring 有哪些模块呢?
Spring ，除了最核心的`Spring Core Container`是必要模块之外，其他模块大约有 20 多个模块都是`可选` 的

最最主要的7大模块:
1. **Spring Core**：Spring 核心，它是框架最基础的部分，提供 IoC 和依赖注入 DI 特性。
2. **Spring Context**：Spring 上下文容器，它是 BeanFactory 功能加强的一个子接口。
3. **Spring Web**：它提供 Web 应用开发的支持。
4. **Spring MVC**：它针对 Web 应用中 MVC 思想的实现。
5. **Spring DAO**：提供对 JDBC 抽象层，简化了 JDBC 编码，同时，编码更具有健壮性。
6. **Spring ORM**：它支持用于流行的 ORM 框架的整合，比如：Spring + Hibernate、Spring + iBatis、Spring + JDO 的整合等。
7. **Spring AOP**：即面向切面编程，它提供了与 AOP 联盟兼容的编程实现。


### part 15

1. Spring 有哪些常用的注解呢?
Spring 提供了大量的注解来简化 Java 应用的开发和配置，主要有 Web 开发相关、容器相关、AOP相关、事务控制相关等
2. web开发方面有哪些注解
`@Controller`：用于标注控制层组件。

`@RestController`：是`@Controller` 和 `@ResponseBody` 的结合体，返回 JSON 数据时使用。

`@ResponseBody`：直接将返回的数据放入 HTTP 响应正文中，一般用于返回 JSON 数据。

`@RequestBody`：接收请求体中的json数据的


`@RequestMapping`：用于映射请求 URL 到具体的方法上，还可以细分为：

- `@GetMapping`：只能用于处理 GET 请求
- `@PostMapping`：只能用于处理 POST 请求
- `@DeleteMapping`：只能用于处理 DELETE 请求

`@PathVariable`：用于接收路径参数，比如 `@RequestMapping(“/hello/{name}”)`，这里的 name 就是路径参数。

`@RequestParam`：用于接收请求参数。比如 `@RequestParam(name = "key") String key`，这里的 key 就是请求参数。

3. 容器类注解有哪些?
- `@Component`：标识一个类为 Spring 组件，使其能够被 Spring 容器自动扫描和管理。
- `@Service`：标识一个业务逻辑组件（服务层）。比如 `@Service("userService")`，这里的 userService 就是 Bean 的名称。
- `@Repository`：标识一个数据访问组件（持久层）。
- `@Autowired`：按类型自动注入依赖。
- `@Configuration`：用于定义配置类，可替换 XML 配置文件。
- `@Value`：用于将 Spring Boot 中 application.properties 配置的属性值赋值给变量。

4. 事务注解有哪些
主要就是 @Transactional

### part 16

1. Spring 中应用了哪些设计模式呢?
![[Pasted image 20240910075526.png]]
单模策适代工观
Spring 框架中用了蛮多设计模式的：
1. 工厂模式: IoC 容器本身可以看作是一个巨大的工厂，负责创建和管理 Bean 的生命周期和依赖关系。
2. 代理模式: AOP 的实现就是基于代理模式的，我在项目中自定义了一个用于自动填充字段的切面, 用于填充createTime 和 updateTime, Spring 会在我写的原有的类的基础上创建一个代理对象放入容器中
3. 单例模式: Spring 容器中的 Bean 默认作用域都是单例的, 所有对该bean的请求都返回同一个实例, 这样可以保证 Bean 的唯一性，减少开销。
4. 模板模式: Spring 中的RedisTamplate 这种 Template 结尾的类，都使用了模板方法模式。它封装了不变部分，扩展可变部分。比如, 我在项目中自定义的 RedisTamplate 它支持 反序列化 和 序列化 key 和 value 的值, 我只需要关注拓展部分, 不许要关注它底层是怎么根redis打交道的. 
5. 观察者模式: spring 中的监听器就用到了观察者模式, 它用于在对象之间建立 一对多 的 依赖关系. 在该模式中, 当主题对象发布事件时, 它会自动通知其他依赖对象 (称为观察者),  比如 我们可以实现 ApplicationListener , 所以监听到 ApplicationEvent 事件的发布了
#todo
6. 适配器模式: Spring MVC 中的 HandlerAdapter 就用了适配器模式。它允许 DispatcherServlet 通过统一的适配器接口与多种类型的请求处理器进行交互。
7. 策略模式: Spring 中有一个 Resource 接口，它的不同实现类，会根据不同的策略去访问资源。



2. spring 容器, web 容器, mvc 容器的区别
Spring 容器是 Spring 框架的核心部分，负责管理应用程序中的对象生命周期和依赖注入。

Web 容器（也称 Servlet 容器），是用于运行 Java Web 应用程序的服务器环境，支持 Servlet、JSP 等 Web 组件。常见的 Web 容器包括 Apache Tomcat、Jetty等。

Spring MVC 是 Spring 框架的一部分，专门用于处理 Web 请求，基于 MVC（Model-View-Controller）设计模式。


3. 说说什么时IoC? 什么时DI
IoC 即反转控制, 将 bean 的控制权由引用对象交给 spring 管理
DI 即是 依赖注入, 假如我们不用spring, 一个任务要两个对象才能完成的话, 我们可能会在A对象里newB对象才能完成任务, 这样耦合度比较高, 但是有了spring之后, 我们在A对象里就可以直接用注解声明一下, 然后spring就会为我们注入B对象了, 这样就降低了对象之间的耦合度. 

4. 能简单说下一 Spirng IoC 的实现机制吗?
它的实现机制是 工厂 + 反射
Spring 的 IoC 本质是一个大工厂, 一个大工厂需要干的活都有
- 生产产品:  就是利用反射创建Bean 
- 库存产品: 把创建出来的Bean放入容器
- 订单处理: 处理Bean之间的依赖关系

5. 说说 BeanFactory 和 ApplicationContext
可以这么比喻，BeanFactory 是 Spring 的“心脏”，而 ApplicantContext 是 Spring 的完整“身躯”。



### part 17
1. 详细说说 BeanFactory
BeanFactory 位于整个 Spring IoC 容器的顶端, ApplicationContext 算是 BeanFactory 的子接口

它最主要的方法就是 `getBean()`，这个方法负责从容器中返回特定名称或者类型的 Bean 实例 

2. 详细说说 ApplicationContext
ApplicationContext 继承了 HierachicalBeanFactory 和 ListableBeanFactory 接口，算是 BeanFactory 的自动挡版本，是 Spring 应用的默认创建容器方式。

ApplicationContext 会在启动时预先创建和配置所有的单例 bean，并支持如 JDBC、ORM 框架的集成，内置面向切面编程（AOP）的支持，可以配置声明式事务管理等。

```java
@Configuration
@ComponentScan(basePackages = "com.github.paicoding.forum.test.javabetter.spring1") // 替换为你的包名
public class AppConfig {
}
```

3. 你知道Spring 容器会在初始阶段干嘛吗?

	Spring 的 IoC 容器工作的过程，其实可以划分为两个阶段：**容器启动阶段**和**Bean 实例化阶段**。

其中容器启动阶段主要做的工作是
加载和解析配置文件，然后把Bean的信息封装到 BeanDefinition 中, 然后这bd会存入bdMap 中

4. 说说 Spring 的 Bean 实例化方式
Spring 提供了 4 种不同的方式来实例化 Bean，以满足不同场景下的需求
有 通过构造方法实例化, 通过静态工厂实例化, 通过实例化工厂实例化, 还有通过FactoryBean接口


5. 说说 SpringBean 的生命周期
	在 Spring 中，基本容器 BeanFactory 和扩展容器 ApplicationContext 的实例化时机不太一样，BeanFactory 采用的是延迟初始化的方式，也就是说，只有在第一次 `getBean()` 获取 Bean 的时候，才会实例化 Bean。
spring 中 bean 的生命周期可以分为4个阶段, 
实例化: 创建出对象, 
属性赋值: 先是填充属性, 然后是调用 Aware 接口
初始化阶段: 初始化会先执行 bean 后处理器的前置方法, InitializaBean 接口回调, 自定义初始化方法的回调, 最后再是 Bean后处理器的后置方法的回调
存储阶段: 然后放入单例池
销毁:  销毁的时候 DisposableBean 接口的回调, 然后是自定义销毁方法的回调

6. 请在一个已有的 Spring Boot 项目中通过单元测试的形式来展示 Spring Bean 的生命周期
```java
public class LifecycleDemoBean implements InitializingBean, DisposableBean {

    // 使用@Value注解注入属性值，这里演示了如何从配置文件中读取值
    // 如果配置文件中没有定义lifecycle.demo.bean.name，则使用默认值"default name"
    @Value("${lifecycle.demo.bean.name:default name}")
    private String name;

    // 构造方法：在Bean实例化时调用
    public LifecycleDemoBean() {
        System.out.println("LifecycleDemoBean: 实例化");
    }

    // 属性赋值：Spring通过反射调用setter方法为Bean的属性注入值
    public void setName(String name) {
        System.out.println("LifecycleDemoBean: 属性赋值");
        this.name = name;
    }

    // 使用@PostConstruct注解的方法：在Bean的属性赋值完成后调用，用于执行初始化逻辑
    @PostConstruct
    public void postConstruct() {
        System.out.println("LifecycleDemoBean: @PostConstruct（初始化）");
    }

    // 实现InitializingBean接口：afterPropertiesSet方法在@PostConstruct注解的方法之后调用
    // 用于执行更多的初始化逻辑
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("LifecycleDemoBean: afterPropertiesSet（InitializingBean）");
    }

    // 自定义初始化方法：在XML配置或Java配置中指定，执行特定的初始化逻辑
    public void customInit() {
        System.out.println("LifecycleDemoBean: customInit（自定义初始化方法）");
    }

    // 使用@PreDestroy注解的方法：在容器销毁Bean之前调用，用于执行清理工作
    @PreDestroy
    public void preDestroy() {
        System.out.println("LifecycleDemoBean: @PreDestroy（销毁前）");
    }

    // 实现DisposableBean接口：destroy方法在@PreDestroy注解的方法之后调用
    // 用于执行清理资源等销毁逻辑
    @Override
    public void destroy() throws Exception {
        System.out.println("LifecycleDemoBean: destroy（DisposableBean）");
    }

    // 自定义销毁方法：在XML配置或Java配置中指定，执行特定的清理逻辑
    public void customDestroy() {
        System.out.println("LifecycleDemoBean: customDestroy（自定义销毁方法）");
    }
}
```

```java
@Configuration
public class LifecycleDemoConfig {

    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public LifecycleDemoBean lifecycleDemoBean() {
        return new LifecycleDemoBean();
    }
}
```

7. Bean 定义 和 依赖定义有哪些方式
**直接编码方式**、**配置文件方式**、**注解方式**。

### part 17
1. 有哪些依赖注入的方法
Spring 支持构造方法注入, 属性注入, 工厂方法注入, 其中工厂方法注入, 又可以分为静态工厂方法注入和非静态工厂方法注入

2. Spring 有哪些自动装配的方法
- **byName**：根据名称进行自动匹配，假设 Boss 又一个名为 car 的属性，如果容器中刚好有一个名为 car 的 bean，Spring 就会自动将其装配给 Boss 的 car 属性
- **byType**：根据类型进行自动匹配，假设 Boss 有一个 Car 类型的属性，如果容器中刚好有一个 Car 类型的 Bean，Spring 就会自动将其装配给 Boss 这个属性
- **constructor**：与 byType 类似， 只不过它是针对构造函数注入而言的。如果 Boss 有一个构造函数，构造函数包含一个 Car 类型的入参，如果容器中有一个 Car 类型的 Bean，则 Spring 将自动把这个 Bean 作为 Boss 构造函数的入参；如果容器中没有找到和构造函数入参匹配类型的 Bean，则 Spring 将抛出异常。
- **autodetect**：根据 Bean 的自省机制决定采用 byType 还是 constructor 进行自动装配，如果 bean 有多个构造器，并且构造器的参数可以被自动装配，Spring 会优先选择构造器注入, 否则就用 byType

3. Spring 中 Bean 的作用域有naxie
- **singleton** : 在 Spring 容器仅存在一个 Bean 实例，Bean 以单实例的方式存在，是 Bean 默认的作用域。
- **prototype** : 每次从容器重调用 Bean 时，都会返回一个新的实例。

以下三个作用域于只在 Web 应用中适用：

- **request** : 每一次 HTTP 请求都会产生一个新的 Bean，该 Bean 仅在当前 HTTP Request 内有效。
- **session** : 同一个 HTTP Session 共享一个 Bean，不同的 HTTP Session 使用不同的 Bean。
- **globalSession**：同一个全局 Session 共享一个 Bean，只用于基于 Protlet 的 Web 应用，Spring5 中已经不存在了。

4. Spring 中的单例 Bean 会存在线程安全问题吗?
单例 Bean 一般都是无状态的, 也就是说涉及不到对其的修改, 所以说是线程安全的. 比如 Controller, Service, Mapper 基本都是无状态的

5. 单例 Bean 的线程安全问题怎么解决
第一, 尽量不使用 共享成员变量 , 而是 局部变量
第二, 尽量使用无状态的Bean
第三, 如果确实存在可以修改的成员变量, 那就使用 synchronized 关键字 或是 实现 Lock 接口来保障线程安全


6. 说说循环依赖
假如 A, B 两个 Bean 相互依赖, 就会造成循环依赖

循环依赖只发生在 Singleton 作用域的 Bean 之间, 因为如果是 Prototype 作用域的 Bean, Spring 会直接抛出异常

原因很简单，AB 循环依赖，A 实例化的时候，发现依赖 B，创建 B 实例，创建 B 的时候发现需要 A，创建 A1 实例……无限套娃。。。。

**![[Pasted image 20240914070836.png]]

7. Spring 怎么解决循环依赖
Spring 通过 三级缓存机制来解决循环依赖
一级缓存: 用来存放完全初始化好的单例 Bean
二级缓存: 用来存放部分初始化的 Bean, 而且被其它对象引用了的
三级缓存: 用来存放 BeanDefinition

8. 三级缓存解决循环依赖的过程是什么样子的
假如有有 A B 这两个 Bean 他们相互依赖
先是实例化 A 的时候将 A 的 BeanDefinition 放入三级缓存, 表示 A 开始实例化了

然后 A 实例化的时候, 发现要注入 B , 就从 1 到 3 级缓存依次去找, 发现没有找到, 就去实例化 B , 然后走一遍 A 之前走过的流程, 在三级缓存中发现了 A , 然后通过 BeanDefinition 引用 A 后将 A 放入2级缓存, 将 B 放入 1 级别缓存, 然后接着 A 的实例化 将单例池中的 B 注入 A 后将 A 从 二级缓存移入一级

9. 为什么要 3 级缓存 2 级不行吗?
不行 考虑到 代理对象问题, 如果只有二级缓存, bean在初始化过程中, 如果被代理对象覆盖了, 可能会导致后续获取的bean对象与预期不一致

### part 18
1. @Autowired 的功能是通过bean后处理器的来完成的, 在后置处理器中会对会判断bean是否需要进行属性填充, 然后再解析@Autowried 注解 来完成属性填充]

2. 说说什么是 AOP
AOP 即是 面向切面编程, 是 Spring 中最重要的核心概念之一, 简单来说, AOP 就是把一些业务逻辑相同的代码抽取到一个独立的模块中, 让业务变得清爽. 比如说, 我在我的项目中开发了一个打印请求接口日志的这样一个功能, 它就是通过AOP实现的, 如果我不用AOP 那我就得写两个方法, 在请求方法开始和返回结果之前调用, 这样就很麻烦, 而且手动调用难免会有纰漏, 所以使用 AOP 来解决得话就让业务逻辑变得更加清爽了, 实现业务逻辑与通用逻辑之间的分离.

3. AOP 有哪些核心概念
- 切面: 类是对物体特征的抽象, 假如把圆柱体看成一个类, 我们AOP关注的就是把它切成一片一片后的切面的特征的抽象
- 连接点:  就是AOP允许你插入的地方(或者说拦截)
- 切点: 通过匹配连接点来定位切面的应用位置
- 通知: 就是对原有代码的增强
- 目标对象: 代理的目标对象
- 引介: 一种特殊的增强, 即时动态的为类去添加一些属性和方法
- 织入: 织入是将增强添加到目标对象的切点上的过程。

织入还可以分为: 
编译器织入: 切面在目标类编译时被织入
类加载期织入: 切面在目标类加载到 JVM 时织入
运行期织入: Spring AOP 就是采用运行期织入

4. AOP 有哪些环绕方式
AOP 有 5 种环绕方式:
- 前置通知: 
- 返回通知 : 只有在目标方法成功执行后才触发
- 异常通知
- 后置通知 : 在目标方法执行后触发, 无论成功与否
- 环绕通知 


5. 说说 JDK 动态代理 和 CGLIB 代理的区别
JDK 动态代理是基于接口的代理, 只能代理实现了接口的类, 如果把接口看作是师傅, 那么创建出的代理对象就是目标对象的同门师弟,
而 CGLIB 则是 继承目标类来创建代理对象, 创建出来的代理对象可以说是目标对象的儿子


6. Spring 事务的种类
在 Spring 中, 事务管理可以分为两大类: 声明式事务 和 编程式事务 管理

7. 介绍下 编程式事务管理 
编程式事务管理可以使用 TransactionTemplate 和 PlatformTransactionManager 来实现，**需要显式执行事务**。**通过编程方式明确指定事务的开始、提交和回滚**。

8. 介绍一下声明式事务管理?
**声明式事务是建立在 AOP 之上的。其本质是通过 AOP ，将事务的功能编织到目标方法中**，也就是在目标方法开始之前启动一个事务，在目标方法执行完之后根据执行情况提交或者回滚事务。

相比较编程式事务，**优点将业务逻辑代码和事务管理的代码分离**， Spring 推荐通过 @Transactional 注解的方式来实现声明式事务管理，也是日常开发中最常用的。

不足的地方是，声明式事务管理最细粒度只能作用到方法, 无法像编程式事务那样可以作用到代码块级别。

我在开发项目中就遇到过回滚失败的问题, 后来发现是因为我在声明式事务控制的方法里面调用了其他方法, 导致回滚失败, 所以之后我就采用了编程式事务控制(即使用了 TransactionTemplate 来管理事务)

9. 说说两者的区别
编程式事务管理: 需要我们显示执行事务管理, 事务管理的颗粒度比较细, 使用起来比较灵活, 但是代码的侵入性比较强, 也不优雅
声明式事务管理: 使用AOP来声明事务, 将事务管理代码和业务逻辑代码分分离了, 优点是代码简洁, 方便维护, 缺点是颗粒度度不够细, 不够灵活 

10. 说说 Spring 事务隔离级别?
好，事务的隔离级别定义了一个事务可能受其他并发事务影响的程度。SQL 标准定义了四个隔离级别，Spring 都支持，并且提供了对应的机制来配置它们，定义在 TransactionDefinition 接口中。

①、ISOLATION_DEFAULT：使用数据库默认的隔离级别（你们爱咋咋滴 😁），MySQL 默认的是可重复读，Oracle 默认的读已提交。

②、ISOLATION_READ_UNCOMMITTED：读未提交，允许事务读取未被其他事务提交的更改。这是隔离级别最低的设置，可能会导致“脏读”问题。

③、ISOLATION_READ_COMMITTED：读已提交，确保事务只能读取已经被其他事务提交的更改。这可以防止“脏读”，但仍然可能发生“不可重复读”和“幻读”问题。

④、ISOLATION_REPEATABLE_READ：可重复读，确保事务可以多次从一个字段中读取相同的值，**即在这个事务内，其他事务无法更改这个字段**，从而避免了“不可重复读”，但仍可能发生“幻读”问题。

⑤、ISOLATION_SERIALIZABLE：串行化，这是最高的隔离级别，它把事务完全隔离了，确保事务序列化执行，以此来避免“脏读”、“不可重复读”和“幻读”问题，但性能影响也最大。

11. Spring 的事务传播机制
事务的传播机制定义了在方法被另一个事务方法调用时, 这个方法的事务行为是什么样子的
Spring 提供了一系列事务传播行为, 这些传播行为定义了事务的边界和事务上下文如何在方法调用链中传播

### part 19
1. 声明式事务在哪些情况下会失效
![[Pasted image 20240919054317.png]]

1. @Transactional 应用在非 public 修饰的方法上
   因为在生成代理对象时会调用原来的方法

2. @Transactional 注解属性 隔离级别 设置错误
   比如说在事务环境下执行的操作使用NEVER

3. 同一个类的方法调用
   因为 如果没有声明事务的方法 A 调用了 声明事务的方法 B, 这个 B 方法可能会因为没有被外部调用而不被代理

4. 没有设置 rollbackFor 或者设置错误
rollbackFor 用来指定能够触发事务回滚的异常类型。Spring 默认抛出继承自 RuntimeException 的异常或者 Error 才回滚事务，其他异常不会触发回滚事务。

2. Spring MVC 的核心组件
**DispatcherServlet**: 前置控制器，是整个流程控制的**核心**，控制其他组件的执行
**Handler**：处理器，完成具体的业务逻辑，相当于 Servlet。
**HandlerMapping**：用来将请求映射到对应 Handler。
**HandlerInterceptor**：拦截器，如果需要完成一些拦截处理，可以实现该接口。
**HandlerExecutionChain**：处理器执行链，包括两部分内容：Handler 和 HandlerInterceptor
**HandlerAdapter**: 处理器适配器，Handler 执行业务方法之前，需要进行一系列的操作，包括表单数据的验证、数据类型的转换、将表单数据封装到 实体里 等

#todo 
3. SpringMVC 的工作流程

4. 介绍一下Spring boot 有哪些优点
