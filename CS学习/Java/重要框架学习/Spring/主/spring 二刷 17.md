# XML 方式 AOP 快速入门
前面我们编写的 AOP 基础代码还是存在一些问题的, 主要如下
- 被增强的包名在代码写死了
- 通知对象的方法在代码中写死了
配置方式的设计, 配置文件 (注解) 的解析工作, Spring 已经帮我们封装好了

## xml方式配置AOP的步骤

1. 导入AOP相关坐标;
2. 准备目标类、准备增强类，并配置给Spring管理
3. 配置切点表达式(哪些方法被增强)
4. 配置织入(切点被哪些通知方法增强，是前置增强还是后置增强)

坐标:

```xml
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.6</version>  
</dependency>
```

导入命名空间

```xml
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
xmlns:aop="http://www.springframework.org/schema/aop"  
xsi:schemaLocation="http://www.springframework.org/schema/aop       http://www.springframework.org/schema/aop/spring-aop.xsd">
```

在把 目标类 和 通知类 给导进来

```xml
<!--配置目标类-->  
<bean id="userService" class="com.fancier.service.impl.UserServiceImpl"></bean>  
<!--配置的通知类-->  
<bean id="myAdvice" class="com.fancier.advice.MyAdvice"></bean>
```

再写个 aop 配置的标签

```xml
<aop:config proxy-target-class="true">  
    <!--配置切点表达式，目的是要指定哪些方法被增强-->  
    <aop:pointcut id="myPointcut" expression="execution(void com.fancier.service.impl.UserServiceImpl.show1())"/>
    <!--配置织入，目的是要执行哪些切点与那些通知进行结合-->  
    <aop:aspect ref="myAdvice">  
        <!--前置通知-->  
        <aop:before method="beforeAdvice" pointcut-ref="myPointcut"/>  
    </aop:aspect>
</aop:config>
```

# 切点表达式的配合方式和语法

## xml方式 AOP 配置详情

### 切点表达式配置语法

切点表达式是配置要对哪些连接点(哪些类的哪些方法)进行通知的增强，语法如下:
```java
executid ([访问修饰符]返回值类型 包名.类名.方法名(参数))
```

其中,
- 访问修饰符可以省略不写;
- 返回值类型, 某一级包名, 类名, 方法名 可以使用 * 表示任意
- 包名与类名之间使用单点 . 标表示该包下的类, 使用双点 .. 表示该爆及其子包下的类
- 参数列表可以使用两个 .. 表示任意参数

咱来喂几个栗子

```java
//表示访问修饰符为public、无返回值、在com.itheima.aop包下的TargetImpl类的无参方法show
execution(public void com.itheima.aop.TargetImpl.show())

//表述com.itheima.aop包下的TargetImpl类的任意方法
execution(* com.itheima.aop.TargetImpl.*(..))

//表示com.itheima.aop包下的任意类的任意方法
execution(* com.itheima.aop.*.*(..))

//表示com.itheima.aop包及其子包下的任意类的任意方法
execution(* com.itheima.aop..*.*(..))

//表示任意包中的任意类的任意方法
execution(* *..*.*(..))
```


### AspectJ 的通知有一下五种类型

| **通知名称**          | **配置方式**               | **执行时机**                                                 |
| ----------------- | ---------------------- | -------------------------------------------------------- |
| 前置通知         <br> | < aop:before >         | 目标方法执行之前执行                                               |
| 后置通知              | <aop:after-returning > | 目标方法执行之后执行，目标方法异常时，不在执行                                  |
| 环绕通知              | <aop:around >          | 目标方法执行前后执行，目标方法异常时，环绕后方法不在执行目标方法执行前后执行，目标方法异常时，环绕后方法不在执行 |
| 异常通知              | <aop:after-throwing >  | 目标方法抛出异常时执行                                              |
| 最终通知              | <aop:after >           | 不管目标方法是否有异常，最终都会执行                                       |

通知方法在被调用时, Spring 可以为其传递一些必要的参数

| **参数类型**            | **作用**                                      |
| ------------------- | ------------------------------------------- |
| JoinPoint           | 连接点对象，任何通知都可使用，可以获得当前目标对象、目标方法参数等信息         |
| ProceedingJoinPoint | JoinPoint子类对象，主要是在环绕通知中执行proceed()，进而执行目标方法 |
| Throwable           | 异常对象，使用在异常通知中，需要在配置文件中指出异常对象名称              |
### AOP 配置的两种语法形式

- 使用\<advisor\>配置切面
- 使用\<aspect\>配置切面
Spring定义了一个Advice接口，实现了该接口的类都可以作为通知类出现

```java
public interface Advice {
}
```

它是一个 `标志接口` 里面什么也没有, 可以看看它的几个子接口
 
![[Pasted image 20240425110950.png]]

这几个子接口 分别对应环绕通知, 前置通知 和 后置通知

看看怎么用吧


前置和后置通知
```java
public class MyAdvice2 implements MethodBeforeAdvice, AfterReturningAdvice {  
  
    @Override  
    public void before(Method method, Object[] objects, Object o) throws Throwable {  
        System.out.println("前置通知..........");  
    }  
  
    @Override  
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {  
        System.out.println("后置通知...........");  
    }  
}
```

环绕通知

```java
public class MyAdvice3 implements MethodInterceptor {  
  
    @Override  
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {  
        System.out.println("环绕前******");  
        //执行目标方法  
        Object res = methodInvocation.getMethod().invoke(methodInvocation.getThis(), methodInvocation.getArguments());  
        System.out.println("环绕后******");  
        return res;  
    }  
}
```

将咱们自己写的通知类放入 spring 容器后, 咱们可以 在 xml 文件里这么配置

```xml
<aop:config>  
    <aop:pointcut id="myPointcut2" expression="execution(* com.fanicer.service.impl.*.*(..))"/>  
    <aop:advisor advice-ref="myAdvice3" pointcut-ref="myPointcut2"/>  
</aop:config>
```

这样成了

#### 两种配置方法对比一下

##### 语法形式不同
- advisor是通过实现接口来确认通知的类型
- aspect是通过配置确认通知的类型，更加灵活

##### 可配置的切面数量不同
- 一个advisor只能配置一个固定通知和一个切点表达式
- 一个aspect可以配置多个通知和多个切点表达式任意组合

##### 使用场景不同
- 允许随意搭配情况下可以使用aspect进行配置
- 如果通知类型单一、切面单一的情况下可以使用advisor进行配置
- 在通知类型已经固定，不用人为指定通知类型时，可以使用advisor进行配置，例如后面要学习的 Spring 事务控制的配置

