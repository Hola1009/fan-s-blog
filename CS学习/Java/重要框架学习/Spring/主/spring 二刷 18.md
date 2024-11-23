## xml方式 AOP 原理解析

```xml
<aop:config>  
    <aop:pointcut id="myPointcut2" expression="execution(* com.itheima.service.impl.*.*(..))"/>  
    <aop:advisor advice-ref="myAdvice3" pointcut-ref="myPointcut2"/>  
</aop:config>
```

要弄懂它的原理首先我们要找到这这标签命名空间的命名空间处理器, 去对应包下 的 MATE-INF 文件夹下的 spring.handlers 文件可以找到 处理器的全限定类名(然后 ctrl + n 去搜索)

```java
public class AopNamespaceHandler extends NamespaceHandlerSupport {  
  
    /**  
     * Register the {@link BeanDefinitionParser BeanDefinitionParsers} for the  
     * '{@code config}', '{@code spring-configured}', '{@code aspectj-autoproxy}'  
     * and '{@code scoped-proxy}' tags.  
     */    @Override  
    public void init() {  
       // In 2.0 XSD as well as in 2.5+ XSDs  
       registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());  
       ... ...   
    }  
  
}
```

看到了 config 这个标签对应的 解析器, 点进去看看吧, 找到 parse 方法

```java
@Override  
@Nullable  
public BeanDefinition parse(Element element, ParserContext parserContext) {  
	... ...
	// 自动代理创建器
    configureAutoProxyCreator(parserContext, element);  
  
    ... ... 
}
```

点进这个自动代理创建器的方法, 里面就调用了一个方法, 直接点进去, 再进第一个方法

```java
public static void registerAspectJAutoProxyCreatorIfNecessary(  
       ParserContext parserContext, Element sourceElement) {  
  
    BeanDefinition beanDefinition = AopConfigUtils.registerAspectJAutoProxyCreatorIfNecessary(  
          parserContext.getRegistry(), parserContext.extractSource(sourceElement));  
    ... ...
}
```

点进去后, 发现 里面也只调用了一个方法, 我们再点进去

```java
@Nullable  
private static BeanDefinition registerOrEscalateApcAsRequired(  
       Class<?> cls, BeanDefinitionRegistry registry, @Nullable Object source) {  
  
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");  
  
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {  
       BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);  
       if (!cls.getName().equals(apcDefinition.getBeanClassName())) {  
          int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());  
          int requiredPriority = findPriorityForClass(cls);  
          if (currentPriority < requiredPriority) {  
             apcDefinition.setBeanClassName(cls.getName());  
          }  
       }  
       return null;  
    }  
  
    RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);  
    beanDefinition.setSource(source);  
    beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);  
    beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);  
    // 重点部分
    registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);  
    return beanDefinition;  
}
```


自己调试, 第一次进这段代码话, 是进不了第一个逻辑的, 所以只会执行后面的代码, 对于后面的代码我们只看 重点部分
这是向 容器中注入 `org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator`
这个类的对象

去搜一下这个类吧, 搜到后先看看它的族谱

![[Pasted image 20240425142009.png]]

发现它实现了 Bean 的后处理器, 那么 增强的 奥义 肯定就在它实现的方法里面了

再它的祖爷爷 `AbstractAutoProxyCreator` 这个类里面找到了 这个方法的实现
```java
@Override  
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {  
    if (bean != null) {  
       Object cacheKey = getCacheKey(bean.getClass(), beanName);  
       if (this.earlyProxyReferences.remove(cacheKey) != bean) {  
	       // 重点方法
          return wrapIfNecessary(bean, beanName, cacheKey);  
       }  
    }  
    return bean;  
}
```

点进重点方法看看

```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {  
    if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {  
       return bean;  
    }  
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {  
       return bean;  
    }  
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {  
       this.advisedBeans.put(cacheKey, Boolean.FALSE);  
       return bean;  
    }  
  
    // Create proxy if we have advice.  
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);  
    if (specificInterceptors != DO_NOT_PROXY) {  
       this.advisedBeans.put(cacheKey, Boolean.TRUE);  
       // 重点方法
       Object proxy = createProxy(  
             bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));  
       this.proxyTypes.put(cacheKey, proxy.getClass());  
       return proxy;  
    }  
  
    this.advisedBeans.put(cacheKey, Boolean.FALSE);  
    return bean;  
}
```

再点进去看看

```java
protected Object createProxy(Class<?> beanClass, @Nullable String beanName,  
       @Nullable Object[] specificInterceptors, TargetSource targetSource) {  
  
    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {  
       AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);  
    }  
  
    ProxyFactory proxyFactory = new ProxyFactory();  
    proxyFactory.copyFrom(this);  
  
    if (!proxyFactory.isProxyTargetClass()) {  
       if (shouldProxyTargetClass(beanClass, beanName)) {  
          proxyFactory.setProxyTargetClass(true);  
       }  
       else {  
          evaluateProxyInterfaces(beanClass, proxyFactory);  
       }  
    }  
  
    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);  
    proxyFactory.addAdvisors(advisors);  
    proxyFactory.setTargetSource(targetSource);  
    customizeProxyFactory(proxyFactory);  
  
    proxyFactory.setFrozen(this.freezeProxy);  
    if (advisorsPreFiltered()) {  
       proxyFactory.setPreFiltered(true);  
    }  
  
    // Use original ClassLoader if bean class not locally loaded in overriding class loader  
    ClassLoader classLoader = getProxyClassLoader();  
    if (classLoader instanceof SmartClassLoader && classLoader != beanClass.getClassLoader()) {  
       classLoader = ((SmartClassLoader) classLoader).getOriginalClassLoader();  
    }  
	//重点
    return proxyFactory.getProxy(classLoader);  
}
```

最终会点进这个方法, 就是动态代理那个方法

```java
@Override  
public Object getProxy(@Nullable ClassLoader classLoader) {  
    if (logger.isTraceEnabled()) {  
       logger.trace("Creating JDK dynamic proxy: " + this.advised.getTargetSource());  
    }  
    return Proxy.newProxyInstance(classLoader, this.proxiedInterfaces, this);  
}
```

### 总结
就是通过 标签的解析器往 Spring 中注入一个 bean 后处理器, 这个后处理器的 after 方法可以通过动态代理生成增强对象, 
再放入到单例池中去


### AOP 底层两种生成 proxy 方式

动态代理的实现的选择，在调用getProxy0 方法时，我们可选用的 AopProxy接口有两个实现类，如上图，这两种都是动态生成代理对象的方式，一种就是基于JDK的，一种是基于Cqlib的

| 代理技术                                                                 | 使用条件                                    | 配置方式                                                                          |
| -------------------------------------------------------------------- | --------------------------------------- | ----------------------------------------------------------------------------- |
| JDK 动态代理技术                                                           | 目标类有接口，是基于接口动态生成实现类的代理对象                | 目标类有接口的情况下，默认方式                                                               |
| <br>Cglib 动态 代理技术                                               <br> | 目标类无接口且不能使用final修饰，是基于被代理对象动态生成子对象为代理对象 | 目标类无接口时，默认使用该方式;目标类有接口时，手动配置<aop:config proxy-target-class=“true”>强制使用Cglib方式 |

```java
//CGlib基于父类（目标类）生成Proxy  
  
//目标对象  
Target target = new Target();  
//通知对象（增强对象）  
MyAdvice4 myAdvice4 = new MyAdvice4();  
  
//编写CGlib的代码  
Enhancer enhancer = new Enhancer();  
//设置父类  
enhancer.setSuperclass(Target.class);//生成的代理对象就是Target的子类  
//设置回调  
enhancer.setCallback(new MethodInterceptor() {  
    @Override  
    //intercept方法相当于JDK的Proxy的invoke方法  
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {  
        myAdvice4.before();  
        Object res = method.invoke(target, objects); //执行目标方法  
        myAdvice4.after();  
        return res;  
    }  
});  
  
//生成代理对象  
Target proxy = (Target) enhancer.create();  
  
//测试  
proxy.show();
```