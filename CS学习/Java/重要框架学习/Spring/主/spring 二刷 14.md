## 基于注解方式的组件扫描

找到我们的容器入口 
```java
new AnnotationConfigApplicationContext(SpringConfig.class);
```

点进去

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {  
    this();  
    register(componentClasses);  
    refresh();  
}
```

点 this() 方法

```java
public AnnotationConfigApplicationContext() {  
    ... ... 
    // 重点
    this.reader = new AnnotatedBeanDefinitionReader(this);  
    ... ... 
}
```

点进这个 重点 方法

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {  
    ... ... 
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);  
}
```

不多说了, 再点进去吧, 然后再点一次 进到这个方法

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(  
       BeanDefinitionRegistry registry, @Nullable Object source) {  
  
    ... ...   
  
    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);  
	  // 注入 Bean 工厂后处理器用来注入 BD
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {  
       RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);  
       def.setSource(source);  
       beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));  
    }  
	// 注入 Bean 后处理器用来完成属性注入
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {  
       RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);  
       def.setSource(source);  
       beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));  
    }  
  
    ... ... 
  
    return beanDefs;  
}
```

好了 现在来看看 这个方法注册的 bean 工厂后处理器吧 `ConfigurationClassPostProcessor.class`
它实现了 `BeanDefinitionRegistryPostProcessor` 说明它肯定能用来注册BD, 那我们就来看看它的实现方法吧

```java
@Override  
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {  
    int registryId = System.identityHashCode(registry);  
    ... ... 
    // 重点
    processConfigBeanDefinitions(registry);  
}
```

咱就看看这重点代码, 点进去吧

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {  
    List<BeanDefinitionHolder> configCandidates = new ArrayList<>();  
    ... ...
	... ...
    // Detect any custom bean name generation strategy supplied through the enclosing application context  
     
  
    // 解析器 
    ConfigurationClassParser parser = new ConfigurationClassParser(  
          this.metadataReaderFactory, this.problemReporter, this.environment,  
          this.resourceLoader, this.componentScanBeanNameGenerator, registry);  
  
    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);  
    Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());  
    do {  
       StartupStep processConfig = this.applicationStartup.start("spring.context.config-classes.parse");  
       //重点
       parser.parse(candidates);    
       ... ...  
    }  while (!candidates.isEmpty());  
}
```

点进去吧 , 不废话了, 它里面调用了自己重载的 `parse` 方法

```java
if (bd instanceof AnnotatedBeanDefinition) {  
    parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());  
}
```

我再点

```java
processConfigurationClass(new ConfigurationClass(metadata, beanName), DEFAULT_EXCLUSION_FILTER);
```

额... 再点..., 找到里面这个方法, 点进去, 重点看这么一段...

```java
do {  
       sourceClass = doProcessConfigurationClass(configClass, sourceClass, filter);  
} 
while (sourceClass != null);
```

再点吧, 看到它里面有这个一段...

```java
Set<BeanDefinitionHolder> scannedBeanDefinitions =  
       this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
```

再点..., 进去后开头就看到这么一段, 这一段是不是很眼熟, 咱们基于 xml 组件扫描, 标签解析器的解析方法里面会用来这个类和它的doScan方法, 咱们在往后面一瞅, 果然这个方法也调用了 doScan 方法, 于是两种 组件扫描 方式在这里迎来了大同 

```java
ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry,  
       componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);

... ...

return scanner.doScan(StringUtils.toStringArray(basePackages));
```


## 画个图来总结吧

![[Pasted image 20240423204001.png]]


