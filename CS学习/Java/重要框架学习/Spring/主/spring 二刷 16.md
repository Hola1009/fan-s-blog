# @Import 注解

Spring与MyBatis注解方式整合有个重要的技术点就是@lmport，第三方框架与Spring整合xml方式很多是凭借自定义标签完成的，而第三方框架与Spring整合注解方式很多是靠@Import注解完成的。

@lmport可以导入如下三种类:

- 普通的配置类
- 实现lmportSelector接口的类
- 实现lmportBeanDefinitionRegistrar接囗的类

```java
@Import(OtherBean.class)  
@Import({MyImportSelector.class})  
@Import(MyImportBeanDefinitionRegistrar.class)
```

## 普通配置类

```java
public class OtherBean {  
  
    @Bean("dataSource")  
    public DataSource dataSource(  
            @Value("${jdbc.driver}") String driverClassName,  
            @Qualifier("userDao2") UserDao userDao,  
            UserService userService  
    ){  
  
        DruidDataSource dataSource = new DruidDataSource();  
  
        return dataSource;  
    }  
  
}
```

## 实现 ImportSelector

```java
public class MyImportSelector implements ImportSelector {  
    @Override  
    public String[] selectImports(AnnotationMetadata annotationMetadata) {  
        //参数annotationMetadata叫做注解媒体数组，该对象内部封装是当前使用了@Import注解的类上的其他注解的元信息  
        Map<String, Object> annotationAttributes = annotationMetadata.getAnnotationAttributes(ComponentScan.class.getName());  
        String[] basePackages = (String[]) annotationAttributes.get("basePackages");  
        System.out.println(basePackages[0]);  
        //返回的数组封装是需要被注册到Spring容器中的Bean的全限定名  
        return new String[]{OtherBean2.class.getName()};  
    }  
}
```


## 实现 ImportBeanDefinitionRegistrar

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {  
  
    @Override  
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry, BeanNameGenerator importBeanNameGenerator) {  
        //注册BeanDefinition  
        BeanDefinition beanDefinition = new RootBeanDefinition();  
        beanDefinition.setBeanClassName(OtherBean2.class.getName());  
        registry.registerBeanDefinition("otherBean2",beanDefinition);  
    }  
}
```
# AOP

## 概念

AOP，Aspect Oriented Programming，面向切面编程，是对面向对象编程OOP的升华。OOP是纵向对一个事物的抽象，一个对象包括 `静态的属性` 信息，包括 `动态的方法` 信息等。而AOP是横向的对不同事物的抽象，**属性与属性、方法与方法、对象与对象都可以组成一个切面**，而用这种思维去设计编程的方式叫做面向切面编程

## AOP 思想的实现方案

动态代理技术，在运行期间，对目标对象的方法进行增强，代理对象同名方法内可以执行原有逻辑的同时嵌入执行其他增强逻辑或其他对象的方法

![[Pasted image 20240424102037.png|left|400]]

## 模拟实现方案

怎么做呢, 了解过 Bean 的生命周期的话 我们很容易想到用 `Bean后处理器` 来做, 注入一个 Bean后处理器 在它的 after方法里生成一个 ProxyBean 返回给单例池

- 目的:对  UserserviceImpl  中的 show1 和 show2 方法进行增强, 增强方法存在与 MyAdvice 中
- 问题1: 筛选  service.impl  包下的所有的类的所有方法(目标方法)都可以进行增强, 解决方案 if-else
- 问题2: MyAdvice怎么获取到?  解决方案: 从Spring容器中获得 MyAdvice(B对象)

### Here we go

实现的 bean 处理器 代码如下

```java
@Component
public class MockAopBeanPostProcessor implements BeanPostProcessor, ApplicationContextAware {  
  
    private ApplicationContext applicationContext;  
  
    @Override  
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
  
        if (bean.getClass().getPackage().getName().equals("com.fancier.service.impl")) {  
            bean = Proxy.newProxyInstance(  
                    bean.getClass().getClassLoader(),  
                    bean.getClass().getInterfaces(),  
                    (proxy, method, args) -> {  
                        MyAdvice myAdvice = (MyAdvice) applicationContext.getBean("myAdvice");  
                        myAdvice.beforeAdvice();  
                        Object result = method.invoke(args);  
                        myAdvice.afterAdvice();  
                        return result;  
                    }  
            );  
        }  
  
        return bean;  
    }  
  
    @Override  
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {  
        this.applicationContext = applicationContext;  
    }  
}
```

#### 代码的缺陷
这代码有啥缺陷呢
- 切点是写死再代码里的
- 只能用一种方式增强


## 相关概念

![[Pasted image 20240424120459.png]]



