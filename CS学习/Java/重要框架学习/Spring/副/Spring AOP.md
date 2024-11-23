##### Spring两大核心 IoC AOP
- IoC: 控制翻转
- AOP: 面相切面编程 
	- 面向对象的一种补充, 是一种思想 将一类事情抽象成公共的事情
###### 切面?
某一类公共的事情的集中处理
- AOP的实现
	- 统一功能的处理 (局限性: 只适用于特定场景)
		- 拦截用户请求
		- 统一结果返回
		- 统一异常处理
- AOP的实现步骤
	1. 引入依赖
	   ```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
	2. 实现切面的逻辑
		1. 建包 aspect 建类
		2. 加 @Aspect注解
		    `import org.aspectj.lang.annotation.Aspect;`
	3. 

### 如何使用AOP完成个性化的开发  
#### 使用AOP统一计算方法运行时间

完成准备工作后开始建类
```java
@Slf4j  
@Aspect
@Component//交给Spring进行管理后才能生效
public class TimeRecordAspect {  
    //传入切点
    @Around("execution(* com.example.springbook.controller.*.*(..))")  //切点表达式
    public Object record(ProceedingJoinPoint pj /*连接点*/) throws Throwable {  
        /**  
         * 该切面的逻辑  
         * 1. 记录开始时间  
         * 2. 执行目标方法  
         * 3. 记录结束时间  
         * 4. 记录消耗时间  
         */  
        long start = System.currentTimeMillis();  //目标方法执行前
        Object result = pj.proceed();  //执行目标方法
        long end = System.currentTimeMillis();  //目标方法执行后
        
        log.info(pj.getSignature() + "cost time:{}ms", end - start); 
        return result;  
    }  
}
```

##### 获得作用的方法
```java
pj.getSignature()
```


切点: A班级全体学生
连接点: 具体学生 (目标方法)
@Around:(通知类型 切面具体要做的事情) 环绕通知
通知类型: 哪个时刻执行通知的内容

#### 通知类型
上⾯我们讲了什么是通知, 接下来学习通知的类型.  `@Around ` 就是其中⼀种通知类型, 表⽰环绕通知. 
Spring中AOP的通知类型有以下⼏种:
• @Around: 环绕通知, 此注解标注的通知⽅法在⽬标⽅法前, 后都被执⾏ 
• @Before: 前置通知,?此注解标注的通知⽅法在⽬标⽅法前被执⾏ 
• @After: 后置通知, 此注解标注的通知⽅法在⽬标⽅法后被执⾏, ⽆论是否有异常都会执⾏ 
• @AfterReturning: 返回后通知, 此注解标注的通知⽅法在⽬标⽅法后被执⾏, 有异常不会执⾏ 
• @AfterThrowing: 异常后通知, 此注解标注的通知⽅法发⽣异常后执⾏

#### 定义切点
方便使用 减少代码量
```java
@Pointcut("execution(* com.bite.aop..*(..))")
public void pt(){};

@Before("pt()")
public void doBefore(){
	log.info("执行AspectDemo.before方法...");
}
```

跨类使用这个切点的使用方法
需要把切点所在类的全限定类名给写上

```java
@Before("com.bite.aop.aspect.AspectDemo.pt()")//全限定类名+切点
public void doBefore(){
	log.info("执行AspectDemo2.before方法...");
}
```

#### 多个切面类的执行顺序
默认是按名称排序
![[多个切面类的执行顺序|left|300]]
##### 如何改变优先级
使用 `@Order` 注解 数越大 优先级越低 //@Order也可以用在其他注解 比如拦截器
```java
public @interface Order {  
  
    /**  
     * The order value.     * <p>Default is {@link Ordered#LOWEST_PRECEDENCE}.  
     * @see Ordered#getOrder()  
     */    int value() default Ordered.LOWEST_PRECEDENCE;  
  
}
```

#### 切点表达式的写法
 ![[Pasted image 20240314152925.png|left400]]
切点表达式⽀持通配符表达:
1. * ：匹配任意字符，只匹配⼀个元素(返回类型, 包, 类名, ⽅法或者⽅法参数)
	a. 包名使⽤ * 表⽰任意包(⼀层包使⽤⼀个*)
	b. 类名使⽤ * 表⽰任意类
	c. 返回值使⽤ * 表⽰任意返回值类型
	d. ⽅法名使⽤ * 表⽰任意⽅法
	e. 参数使⽤ * 表⽰⼀个任意类型的参数
1. .. ：匹配多个连续的任意符号, 可以通配任意层级的包, 或任意类型, 任意个数的参数
	a. 使⽤ .. 配置包名，标识此包以及此包下的所有⼦包
	b. 可以使⽤ ..  配置参数，任意个任意类型的参数
	
#### 使用自定义注解完成AOP开发

1. 定义一个注解
```java
@Retention(RetentionPolicy.CLASS)//生存期  
@Target(ElementType.METHOD)//作用范围  
public @interface TimeRecord {  

}
```
2. 实现该注解需要完成的功能
```java
 @Around("@annotation(com.bite.aop.aspect.TimeRecord)")
    public Object timeRecord(ProceedingJoinPoint joinPoint) {
        long start = System.currentTimeMillis();
        Object result = null;
        try {
            result = joinPoint.proceed();
        }catch (Throwable e){
            log.error(joinPoint.toString()+"发生异常, e:",e);
        }
        log.info(joinPoint.toString()+"执行时间: "+ (System.currentTimeMillis()-start) + "ms");
        return result;
    }
}
```
3. 使用该注解

### SpringAOP的原理
代理模式, 也叫委托模式
代理类
目标类(被代理对象) : 目标类来做具体实现

对一个类不太适合直接访问(可能这个类没开放权限), 就可以使用代理模式(类似Getter 和 Setter 方法)
##### 代理模式中的主要角色
1. Subject: 业务接口类, 可以是抽象类或者接口
2. RealSubject: 业务实现类, 具体的业务执行, 也就是被代理对象
3. Proxy: 代理类, RealSubject的代理类

#### 静态代理
对每个目标类都生成一个代理类

#### 动态代理
在运行时, 用反射动态生成代理类

##### 动态代理的实现方式
1. JDK动态代理
2. CGLib
Spring AOP 两个都使用了

##### Spring AOP 使用的是哪种代理
共同点, 都可以使用JDK 和 CGLib来代理
不同点
1. Spring Framework
	- 如果代理的是接口, 使用JDK, 如果代理的是没有实现接口的类, 使用的是CGLib
2. Spring  Boot
	- 默认配置使用CGLib, 代理的无论是否实现的接口, 都使用CGLib, 如果需要使用JDK代理

## 实际应用中
### 怎么解决自调用事务失效问题
需要通过 `@EnableAspectJAutoProxy(exposeProxy = true)` 来暴露代理对象
通过 `AopContext.currentProxy()` 拿到代理对象来调用事务管理方法




