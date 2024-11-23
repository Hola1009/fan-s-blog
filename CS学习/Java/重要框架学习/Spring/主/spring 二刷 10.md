## Spring 整合 MyBatis 的原理分析
整合包提供了一个 `SqlSessionFactoryBean` 和一个扫描 Mapper 的配置对象 SqlSessionFactoryBean 一旦被实例化, 就开始扫描 Mapper 并通过动态代理产生 Mapper 的实现类存储到 Spring 容器中. 相关的有如下四各类:

- **SqlSessionFactoryBean**: 需要进行配置，用于提供SqlSessionFactory;
- **MapperScannerConfigurer**: 需要进行配置，用于扫描指定mapper注册BeanDefinition;
- **MapperFactoryBean**: Mapper的FactoryBean，获得指定Mapper时调用getObject方法
- **ClassPathMapperScanner**: definition.setAutowireMode(2)修改了自动注入状态，所以MapperFactoryBean中的setSqlSessionFactory会自动注入进去。

### SqlSessionFactoryBean

它实现了 `FactoryBean<SqlSessionFactory>`  看看它的getObject 方法

```java
@Override  
public SqlSessionFactory getObject() throws Exception {  
  if (this.sqlSessionFactory == null) {  
    afterPropertiesSet();  
  }  
  
  return this.sqlSessionFactory;  
}
```

调用了 `afterPropertiesSet()` 方法,  这个方法是该类对 `InitializingBean` 的实现, 因为 属性注入 在 `InitializingBean` 接口初始化之前执行 所以在执行 这个方法之前 已经把 `dataSource` 给注进容器了, 看看代码

```java
public void setDataSource(DataSource dataSource) {  
  if (dataSource instanceof TransactionAwareDataSourceProxy) {  
    // If we got a TransactionAwareDataSourceProxy, we need to perform  
    // transactions for its underlying target DataSource, else data    // access code won't see properly exposed transactions (i.e.    // transactions for the target DataSource).    this.dataSource = ((TransactionAwareDataSourceProxy) dataSource).getTargetDataSource();  
  } else {  
    this.dataSource = dataSource;  
  }  
}
```

好了, 这样 `dataSource` 就有值了, 我们再回过头来看看 `afterPropertiesSet()` 方法

```java
@Override  
public void afterPropertiesSet() throws Exception {  
  notNull(dataSource, "Property 'dataSource' is required");  
  notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");  
  state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),  
      "Property 'configuration' and 'configLocation' can not specified with together");  
  //重点
  this.sqlSessionFactory = buildSqlSessionFactory();  
}
```

它里面调用了 `buildSqlSessionFactory()` 方法 点过来看看; 这个方法方法体很长, 我们主要看它的最后一句

```java
protected SqlSessionFactory buildSqlSessionFactory() throws Exception {  

  ... ...  
  
  return this.sqlSessionFactoryBuilder.build(targetConfiguration);  
}
```

这代码是不是很说熟悉 我们来看看 **MyBatis** 连接数据库代码是咋写的

```java
InputStream in = Resources.getResourceAsStream("mybatis-config.xml");

//=========== 重点嗷 ============
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();  
SqlSessionFactory sqlSessionFactory = builder.build(in);  
//=========== 重点嗷 ============

SqlSession sqlSession = sqlSessionFactory.openSession();
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
```

是不是明白了这是咋回事了捏 _(๑￫ܫ￩)_

谁要用到 `SqlSessionFactory` 就调用 `getObject` 方法  就把成员的 `SqlSessionFactory` 对象给返回去  
#### 画个图梳理一下

![[Pasted image 20240418214137.png|left|500]]

### MapperScannerConfigurer 

它叫啥呢? 叫 `mapper扫描配置器` ٩(๑òωó๑)۶

它实现了哪些接口捏? 我们来康康
```java
BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware
```

这其中我们要中点研究的是 `BeanDefinitionRegistryPostProcessor` 这个接口的实现; 来看看这个方法的实现

```java
@Override  
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {

ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
  ... ...  
  scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS));  
  
}
```

重点看这段代码即可, 点进 scan 方法瞅瞅 (发现它是 `ClassPathBeanDefinitionScanner` 的方法, 而不是 `ClassPathMapperScanner` 的方法. 为啥, 因为继承. 而且这个方法没有被重写)

```java
public int scan(String... basePackages) {  
    int beanCountAtScanStart = this.registry.getBeanDefinitionCount();  
    this.doScan(basePackages);  
    if (this.includeAnnotationConfig) {  
        AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);  
    }  
  
    return this.registry.getBeanDefinitionCount() - beanCountAtScanStart;  
}

```
> `basePackages` 存的是 mapper 包的加载路径

它里头还有个 doScan() 方法, 点进去发现是父类的 doScan() 方法, 但因为子类重写了 doScan() 方法所以实际回调用子类的 doScan() 方法, 也就是 `ClassPathMapperScanner` 的 doScan() 方法

```java
@Override  
public Set<BeanDefinitionHolder> doScan(String... basePackages) {  
  Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);  
  
  if (beanDefinitions.isEmpty()) {  
    LOGGER.warn(() -> "No MyBatis mapper was found in '" + Arrays.toString(basePackages)  
        + "' package. Please check your configuration.");  
  } else {  
    processBeanDefinitions(beanDefinitions);  
  }  
  
  return beanDefinitions;  
}
```
发现 它又调用了 父类 的 doScan() 方法... (눈‸눈)

```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {  
    Assert.notEmpty(basePackages, "At least one base package must be specified");  
    Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet();  
    String[] var3 = basePackages;  
    int var4 = basePackages.length;  
  
    for(int var5 = 0; var5 < var4; ++var5) {  
        String basePackage = var3[var5];  
        Set<BeanDefinition> candidates = this.findCandidateComponents(basePackage);  
        Iterator var8 = candidates.iterator();  
  
        while(var8.hasNext()) {  
            BeanDefinition candidate = (BeanDefinition)var8.next();  
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);  
            candidate.setScope(scopeMetadata.getScopeName());  
            String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);  
            if (candidate instanceof AbstractBeanDefinition) {  
                this.postProcessBeanDefinition((AbstractBeanDefinition)candidate, beanName);  
            }  
  
            if (candidate instanceof AnnotatedBeanDefinition) {  
                AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition)candidate);  
            }  
  
            if (this.checkCandidate(beanName, candidate)) {  
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);  
                definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);  
                beanDefinitions.add(definitionHolder);  
                
                //=========== 重点嗷 ============
                this.registerBeanDefinition(definitionHolder, this.registry);  
                //=========== 重点嗷 ============
                
            }  
        }  
    }  
  
    return beanDefinitions;  
}
```

看重点部分的代码就是在注入 BeanDefinition , 这个 definitionHolder 就是包了一层的 BeanDefinition, 可以看到这个 BD 里面的 beanClass 是 接口的全限定名, 可接口没法产生对象 \_(:3 」∠ )\_

![[Pasted image 20240419074234.png|left|400]]

好了 这些接口的 BD 被 注册到 BDMap中后 就把包含了刚刚被注册 BD 的 beanDefinitions 放回给子类的 doScan 去, 再执行子类的 processBeanDefinitions 方法
```java
private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {  
  GenericBeanDefinition definition;  
  for (BeanDefinitionHolder holder : beanDefinitions) {  
    definition = (GenericBeanDefinition) holder.getBeanDefinition();  
    String beanClassName = definition.getBeanClassName();  
    LOGGER.debug(() -> "Creating MapperFactoryBean with name '" + holder.getBeanName() + "' and '" + beanClassName  
        + "' mapperInterface");  
  
    // the mapper interface is the original class of the bean  
    // but, the actual class of the bean is MapperFactoryBean    definition.getConstructorArgumentValues().addGenericArgumentValue(beanClassName); // issue #59

	//=========== 重点嗷 ============
    definition.setBeanClass(this.mapperFactoryBeanClass);  
	//=========== 重点嗷 ============

    definition.getPropertyValues().add("addToConfig", this.addToConfig);  
  
    boolean explicitFactoryUsed = false;  
    if (StringUtils.hasText(this.sqlSessionFactoryBeanName)) {  
      definition.getPropertyValues().add("sqlSessionFactory",  
          new RuntimeBeanReference(this.sqlSessionFactoryBeanName));  
      explicitFactoryUsed = true;  
    } else if (this.sqlSessionFactory != null) {  
      definition.getPropertyValues().add("sqlSessionFactory", this.sqlSessionFactory);  
      explicitFactoryUsed = true;  
    }  
  
    if (StringUtils.hasText(this.sqlSessionTemplateBeanName)) {  
      if (explicitFactoryUsed) {  
        LOGGER.warn(  
            () -> "Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");  
      }  
      definition.getPropertyValues().add("sqlSessionTemplate",  
          new RuntimeBeanReference(this.sqlSessionTemplateBeanName));  
      explicitFactoryUsed = true;  
    } else if (this.sqlSessionTemplate != null) {  
      if (explicitFactoryUsed) {  
        LOGGER.warn(  
            () -> "Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");  
      }  
      definition.getPropertyValues().add("sqlSessionTemplate", this.sqlSessionTemplate);  
      explicitFactoryUsed = true;  
    }  
  
    if (!explicitFactoryUsed) {  
      LOGGER.debug(() -> "Enabling autowire by type for MapperFactoryBean with name '" + holder.getBeanName() + "'.");  
      definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);  
    }  
    definition.setLazyInit(lazyInitialization);  
  }  
}
```


重点看 这段代码 `definition.setBeanClass(this.mapperFactoryBeanClass);` 它要将BD中的 beanClass 给覆盖掉
而入参 this.mapperFactoryBeanClass 这个工场类就是用来获取 mapper接口 的实现类的

它实现了 FactoryBean, 我们来看看它的 getObject() 方法
```java
@Override  
public T getObject() throws Exception {  
  return getSqlSession().getMapper(this.mapperInterface);  
}
```
很眼熟, 很眼熟我们看看 MyBatis 的代码是怎么使用的

```java
InputStream in = Resources.getResourceAsStream("mybatis-config.xml");  
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();  
SqlSessionFactory sqlSessionFactory = builder.build(in);  
SqlSession sqlSession = sqlSessionFactory.openSession();

//========== 看这 ==========
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
//========== 看这 ==========

```

好了, 再回过头来看看  processBeanDefinitions() 方法中的这一段 

```java
if (!explicitFactoryUsed) {  
	... ...
	definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);  
}
```

它是用来干啥的呢?   代表设置自动注入 即 `根据类型自动注入` 因为 SqlSessionFactory 已经注进 容器了, 

![[Pasted image 20240419080847.png|left]]

而 现在这个 DB 里面 beanClass 实际是 MapperFactoryBean 所以在属性注入时要为其注入 SqlSessionFactory 的属性

#### 画个图梳理一下

![[Pasted image 20240419090454.png|left|600]]















###### 小芝士

在注释里这样写可以跳转到对应的方法
```java
@see #methodName
```


