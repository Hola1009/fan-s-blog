### SpringBoot 原理分析
@SpringBootApplication注解
```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@SpringBootConfiguration  
@EnableAutoConfiguration  
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),  
       @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })  
public @interface SpringBootApplication {...}
```

核心注解
```java
@EnableAutoConfiguration 
```

#### @EnableAutoConfiguration 
```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@AutoConfigurationPackage  
@Import(AutoConfigurationImportSelector.class)  
public @interface EnableAutoConfiguration {
```

两个核心注解
```java
@AutoConfigurationPackage  
@Import(AutoConfigurationImportSelector.class)  
```

##### @Import(AutoConfigurationImportSelector.class) 
AutoConfigurationImportSelector 实现了 `ImportSelector` 接⼝
底层 调用  getAutoConfigurationEntry(annotationMetadata); 方法获取自动配置的配置类信息

```java
@Override  
public String[] selectImports(AnnotationMetadata annotationMetadata) {  
    if (!isEnabled(annotationMetadata)) {  
       return NO_IMPORTS;  
    }  
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);  
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());  
}
```

点进去看看
```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {  
    if (!isEnabled(annotationMetadata)) {  
       return EMPTY_ENTRY;  
    }  
    AnnotationAttributes attributes = getAttributes(annotationMetadata);  
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);  
    configurations = removeDuplicates(configurations);  
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);  
    checkExcludedClasses(configurations, exclusions);  
    configurations.removeAll(exclusions);  
    configurations = getConfigurationClassFilter().filter(configurations);  
    fireAutoConfigurationImportEvents(configurations, exclusions);  
    return new AutoConfigurationEntry(configurations, exclusions);  
}
```

这是关键一步, 这个方法是从自动配置文件中读取自动配置类集合
```java
List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
```



```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {  
    List<String> configurations = new ArrayList<>(  
          SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader()));  
    ImportCandidates.load(AutoConfiguration.class, getBeanClassLoader()).forEach(configurations::add);  
    Assert.notEmpty(configurations,  
          "No auto configuration classes found in META-INF/spring.factories nor in META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you "  
                + "are using a custom packaging, make sure that file is correct.");  
    return configurations;  
}
```

这个方法的功能时获取所有 META-INF/spring.factories 或METAINF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. 文件中配置类的集合

再把这些配置类中使用@Bean声明的对象 方剂容器中去

##### @AutoConfigurationPackage 
```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@Import(AutoConfigurationPackages.Registrar.class)  
public @interface AutoConfigurationPackage {
```

@AutoConfigurationPackage 就是将启动类所在的包下⾯所有的组件都扫描注冊到spring容器中

### 总结
SpringBoot 自动配置原理的大概流程
![[Pasted image 20240406105711.png]]

