# 基于注解方式配置 AOP

## 基本使用



```java
@Component  
@Aspect  
public class MyAdvice {  
  
    //切点表达式的抽取  
    @Pointcut("execution(* com.fancier.service.impl.*.*(..))")  
    public void myPointcut(){}  
  
    @Before("myPointcut()")  
    public void beforeAdvice(JoinPoint joinPoint){   
        System.out.println("前置的增强....");  
    }
}
```

```xml
<!--组件扫描-->  
<context:component-scan base-package="com.fancier"/>  
  
<!--使用注解配置AOP，需要开启AOP自动代理-->  
<aop:aspectj-autoproxy/>
```

其他几个通知的使用方式也是大同小异

## 原理刨析

### 半注解方式

就是刚刚前面那种, 话不多说, 咱们直接来看它的标签解析器 (去命名空间处理器的init方法里面找) 干了啥

```java
@Override  
public void init() {  
    // In 2.0 XSD as well as in 2.5+ XSDs  
    registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser()); 
    
    // 看这里 
    registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
	// 看这里


    registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());  
  
    // Only in 2.0 XSD: moved to context namespace in 2.5+  
    registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());  
}
```

点击去  `new AspectJAutoProxyBeanDefinitionParser()` 找到它的 perse 方法

```java
@Override  
@Nullable  
public BeanDefinition parse(Element element, ParserContext parserContext) { 
	// 这段很眼熟是不是
    AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(parserContext, element);  
    
    extendBeanDefinition(element, parserContext);  
    return null;  
}
```

那段很眼熟的代码 跟xml 方式配置的代码是一样的, 这里就不再深挖了; 自行回顾: [[spring 二刷 18#xml方式 AOP 原理解析|xml 方式原理解析]]



### 全注解方式

核心配置类
```java
@Configuration  
@ComponentScan("com.fancier")   //<context:component-scan base-package="com.itheima"/>  
@EnableAspectJAutoProxy //<aop:aspectj-autoproxy/>  
public class SpringConfig {  
}
```

`@EnableAspectJAutoProxy` 这个注解就是代替 `<aop:aspectj-autoproxy/>` 的

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Import(AspectJAutoProxyRegistrar.class)  
public @interface EnableAspectJAutoProxy {  
  
    /**  
     * Indicate whether subclass-based (CGLIB) proxies are to be created as opposed     * to standard Java interface-based proxies. The default is {@code false}.  
     */    boolean proxyTargetClass() default false;  
  
    /**  
     * Indicate that the proxy should be exposed by the AOP framework as a {@code ThreadLocal}  
     * for retrieval via the {@link org.springframework.aop.framework.AopContext} class.  
     * Off by default, i.e. no guarantees that {@code AopContext} access will work.  
     * @since 4.3.1  
     */    boolean exposeProxy() default false;  
  
}
```

它里面导入了这样一个配置类 `@Import(AspectJAutoProxyRegistrar.class) ` , 深入进去发现它需要向 spring 中注入这样一个类 `AnnotationAwareAspectJAutoProxyCreator.class` 它自动代理构建器 是用来增强 Bean 的, 这就和前面的两种方式联通起来的, 大同!!!

```java
@Nullable  
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(  
       BeanDefinitionRegistry registry, @Nullable Object source) {  
  
    return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);  
}
```


## 帖个图
![[Pasted image 20240426093925.png]]

无论那种方式最终都是在 容器中注入个 `自动代理构建器` 



