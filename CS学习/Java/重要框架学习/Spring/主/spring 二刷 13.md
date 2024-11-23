## Spring 的注解开发-注解版本
基本 Bean 注解, 主要是使用注解的方式代替原有 xml 的\<Bean\> 标签及其标签属性的配置
```xml
<bean name="" Class=rn scope="" 1azy-init="" init-method="" destroy-method="" autowire="" autowire="" factory-bean="" factory-method=""></bean>
```

使用 @Component 注解代替 \<bean\> 标签

## @Compent 注解的使用
在需要被注入的类上加上 该注解, 然后在配置文件中配置扫描路劲

加上 扫描路径
```xml
<context:component-scan base-package="com.fancier"/>
```

它底层是 bean 工厂后处理 来完成的

## 作用范围使用注解
除了@Compent 代替 \<bean\> 标签, 还有其他的一些注解来代替, 标签中的一些属性

| xml配置                      | 注解             | 描述                                                    |
| -------------------------- | -------------- | ----------------------------------------------------- |
| \<bean scope=""\>          | @Scope         | 在类上或使用了@Bean标注的方法上，标注Bean的作用范围，取值为singleton或prototype |
| \<bean lazy-  init=""\>    | @Lazy          | 在类上或使用了@Bean标注的方法上，标注Bean是否延迟加载，取值为<br>true和false     |
| \<bean init-method=""\>    | @PostConstruct | 在方法上使用，标注Bean的实例化后执行的方法                               |
| \<bean destroy-method=""\> | @PreDestroy    | 在方法上使用，标注Bean的销毁前执行方法                                 |
```java 
@Scope("singleton")  
@Lazy(value = true)

//下面两个是加在方法上的
@PostConstruct
@PreDestroy
```

## @Component 的衍生注解

| @Component衍生注解 | 描述            |
| -------------- | ------------- |
| @Repository    | 在Dao层上使用      |
| @Service       | 在Service层类上使用 |
| @Controller    | 在web层类上使用     |
当 一个类不属于某个层面就使用  @Component 注解


## 依赖注入的注解

| 属性注入注解     | 描述                                |
| ---------- | --------------------------------- |
| @Value     | 使用在字段或方法上, 用于注入普通属性               |
| @Autowired | 使用在字段或方法上, 用于根据类型 (byType) 注入引用数据 |
| @Qualifier | 使用在字段或方法上, 结合@Autowired, 根据名称注入   |
| @Resource  | 使用在字段或方法上, 根据类型或名称注入              |
如果在 spring  xml 配置文件中 还引入了其他配置文件 

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>
```

则 @vaule 注解还可以这么使用

```java
@Value("${jdbc.driver}")
```

@Autowired 还可以写在set方法上 根据参数类型去匹配的

## 非自定义Bean注解开发

非自定义Bean不能像自定义Bean一样使用@Component进行管理，非自定义Bean要通过工厂的方式进行实例化使用@Bean标注方法即可，@Bean的属性为beanName，如不指定则为当前工厂方法名称

@Bean 方法所在的类必须被 Spring 管理

### 如果需要参数注入怎么办
默认会使用 @Automated 注解进行注入

对于基本数据类型可以使用 @Value("${xx.xx}") 加在入参前面 对于引用数据类型也可以直接使用 @Qualifier

## 配置类替代配置文件

```java
@Configuration
@ComponentScan(basePackages = {"com.fancier"})
@PropertySource({"classpath:jdbc.properties"})
@Import(OtherBean.class)
public class SpringConfig {}
```


使用 ` @Configuration` 注解 标识当前类是一个配置类 (代替配置文件) + @Component

使用 `@ComponentScan` 注解 相当于 `<context:component-scan base-package="com.fancier"/> `

使用 `@PropertySource` 注解 相当于 `<context:property-placeholder location="classpath:jdbc.properties"/>`

使用 `@import` 在核心配置类上导入其他配置类 相当于 `<import resource=""></import>` 标签

这些注解需要写在配置类上

## 启动使用注解的 Spring 容器

使用 xml 文件 得 这么启动 Spring 容器

```java
new ClassPathXmlApplicationContext("applicationContext.xml");
```

那么使用 注解的方法我们肯定得换一个启动类来启动了

```java
new AnnotationConfigApplicationContext(核心配置类.class);
```

## Spring 配置其他注解
`@Primary` 注解用于标注相同类型的Bean优先被使用权，@Primary 是Spring3.0引入的，与 @Component 和 @Bean 一起使用，标注该 Bean 的优先级更高，则在通过类型获取Bean或通过 @Autowired 根据类型进行注入时会选用优先级更高的

`@Profile` 注解的作用同于 xml 配置时学习 profile 属性，是进行环境切换使用的
```xml
<beans profile="test">
```
注解 @Profile 标注在类或方法上，标注当前产生的Bean从属于哪个环境，只有激活了当前环境，被标注的Bean才能被注册到Spring容器里，不指定环境的Bean，任何环境下都能注册到Spring容器里

可以使用以下两种方式指定被激活的环境:
- 使用命令行动态参数，虚拟机参数位置加载

```bash
-Dspring.profiles.active=test
```
- 使用代码的方式设置环境变量 
```java
System.setProperty("spring.profiles.active","test”);
```

## Spring 注解得解析原理

使用 @Component 等注解配置完毕后, 要配置组件扫描注解才能生效

xml配置组件扫描

```xml
<context:component-scan base-package="com.fancier"/>
```

配置类配置组件扫描:

```java
@ComponentScan("com.fancier")
```

它是如何生效的呢, 我们来看看
先通过MATE-INF 文件中的 映射文件找到命名空间处理器
```java
public class ContextNamespaceHandler extends NamespaceHandlerSupport {  
  
    @Override  
    public void init() {  
       registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());  
       registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());  
       registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());  
       registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());  
       registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());  
       registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());  
       registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());  
       registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());  
    }  
  
}
```

再找对应标签的解析器, 看它的 `parse()` 方法

```java
@Override  
@Nullable  
public BeanDefinition parse(Element element, ParserContext parserContext) {  
    String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);  
    basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);  
    String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,  
          ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);  
  
    // Actually scan for bean definitions and register them.  
    ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);  
    Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);  
    registerComponents(parserContext.getReaderContext(), beanDefinitions, element);  
  
    return null;  
}
```

点进 `doScan()` 方法看看

```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {  
    Assert.notEmpty(basePackages, "At least one base package must be specified");  
    Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();  
    for (String basePackage : basePackages) {  
       Set<BeanDefinition> candidates = findCandidateComponents(basePackage);  
       for (BeanDefinition candidate : candidates) {  
          ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);  
          candidate.setScope(scopeMetadata.getScopeName());  
          String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);  
          if (candidate instanceof AbstractBeanDefinition) {  
             postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);  
          }  
          if (candidate instanceof AnnotatedBeanDefinition) {  
             AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);  
          }  
          if (checkCandidate(beanName, candidate)) {  
             BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);  
             definitionHolder =  
                   AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);  
             beanDefinitions.add(definitionHolder);  
             // 重点
             registerBeanDefinition(definitionHolder, this.registry); 
             // 重点 
          }  
       }  
    }  
    return beanDefinitions;  
}
```

重点这一步就是在往BDMap中注入 Bean 

> [!summary] 结论
> 只要将 Bean 对应的 BeanDefinition 注入 BeanDDefinitionMap 中, 就可以经历 SpringBean 的生命周期, 最终实例化进入单例池中

