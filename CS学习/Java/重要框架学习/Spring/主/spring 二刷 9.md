## xml 整合第三方框架有两种整合方案:
- 不许要自定义命名空间, 不许要使用 Spring 的配置文件配置第三方框架本身内容, 例如:MyBatis;
- 需要引入第三方框架命名空间, 需要使用 Spring 的配置文件配置第三方框架本身内容, 例如: Dubbo

### Spring xml 方式整合第三方框架
Spring 整合 MyBatis, 之前已经在 Spring 中简单的配置了 SqlSessionFactory, 但是这不是正规的整合方式, MyBatis 提供了
mybatis-spring.jar 专门用于两大框架整合.
Spring 整合 MyBatis 的步骤如下:
- 导入 MyBatis 整合 Spring 的相关坐标;
- 编写 Mapper 和 Mapper.xml
- 配置 `SqlSessionFactoryBean` 和 `MapperScannerConfigurer`;
- 编写代码

这是一种整合方式但是
```xml
<bean id="in" class="org.apache.ibatis.io.Resources" factory-method="getResourceAsStream">
	<constructor-arg name="resource" value="mybatis-config.xml"></constructor-arg>
</bean>

<bean id="builder" class="org.apache.ibatis.session.SqlSessionFactoryBuilder"></bean>

<bean id="sqlSessionFactory" factory-bean="builder" factory-method="build">
	<constructor-arg name="inputStream" ref="in"></constructor-arg>
</bean>
```

这还没完, 还是要写这么些代码, 或将其注入 Spring 也成, 但是也麻烦
```java
SqlSession sqlSession = sqlSessionFactory.openSession();
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
List<User> all = mapper.findAll();
```

Spring 和 MyBatis 整合是为了, 简化上面的这些代码

#### 怎么做呢
##### 先导几个包
```xml
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.49</version>
</dependency>

<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.5.5</version>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>5.2.13.RELEASE</version>
</dependency>

<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>2.0.5</version>
</dependency>
```




### 源码解读

`SqlSessionFactoryBean` 它继承了FactoryBean\<SqlSessionFactory\> 所以有一个重写的方法 是用来 获取 SqlSessionFactory 对象的

```java
@Override  
public SqlSessionFactory getObject() throws Exception {  
  if (this.sqlSessionFactory == null) {  
    afterPropertiesSet();  
  }  
  return this.sqlSessionFactory;  
}
```

`MapperScannerConfigurer`  它继承了这么多的接口 

```java
public class MapperScannerConfigurer  
    implements BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware {
	... ...    
}
```

咱们 主要看 `BeanDefinitionRegistryPostProcessor` 接口的实现; 嚯, 这么一大坨

```java
@Override  
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {  
  if (this.processPropertyPlaceHolders) {  
    processPropertyPlaceHolders();  
  }  
  
  ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);  
  scanner.setAddToConfig(this.addToConfig);  
  scanner.setAnnotationClass(this.annotationClass);  
  scanner.setMarkerInterface(this.markerInterface);  
  scanner.setSqlSessionFactory(this.sqlSessionFactory);  
  scanner.setSqlSessionTemplate(this.sqlSessionTemplate);  
  scanner.setSqlSessionFactoryBeanName(this.sqlSessionFactoryBeanName);  
  scanner.setSqlSessionTemplateBeanName(this.sqlSessionTemplateBeanName);  
  scanner.setResourceLoader(this.applicationContext);  
  scanner.setBeanNameGenerator(this.nameGenerator);  
  scanner.setMapperFactoryBeanClass(this.mapperFactoryBeanClass);  
  if (StringUtils.hasText(lazyInitialization)) {  
    scanner.setLazyInitialization(Boolean.valueOf(lazyInitialization));  
  }  
  scanner.registerFilters();  
  scanner.scan(  
      StringUtils.tokenizeToStringArray(this.basePackage, ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS));  
}
```

但功能很简单, 就是将 `mapper` 的对象存储到容器中

### 使用一下吧

先写一下 Spring 配置文件

```xml
<!--配置数据源信息-->  
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">  
    <property name="driverClassName" value="${jdbc.driver}"></property>  
    <property name="url" value="${jdbc.url}"></property>  
    <property name="username" value="${jdbc.username}"></property>  
    <property name="password" value="${jdbc.password}"></property>  
</bean>  
  
<!--配置SqlSessionFactoryBean，作用将SqlSessionFactory存储到spring容器-->  
<bean class="org.mybatis.spring.SqlSessionFactoryBean">  
    <property name="dataSource" ref="dataSource"></property>  
</bean>  
  
<!--MapperScannerConfigurer,作用扫描指定的包，产生Mapper对象存储到Spring容器-->  
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
    <property name="basePackage" value="com.itheima.mapper"></property>  
</bean>
```

这样, 下面这些代码就都不用写了, 也不用写 MyBatis 配置文件了, 直接在需要 mapper 的地方直接注入即可

```java
InputStream in = Resources.getResourceAsStream("mybatis-config.xml");  
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();  
SqlSessionFactory sqlSessionFactory = builder.build(in);  
SqlSession sqlSession = sqlSessionFactory.openSession();
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
```

需要用 mapper 时直接注入即可
```xml
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl">  
    <property name="userMapper" ref="userMapper"></property>  
</bean>
```


