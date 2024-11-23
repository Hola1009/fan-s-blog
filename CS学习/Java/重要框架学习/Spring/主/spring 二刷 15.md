# Spring 注解方式整合第三方框架
第三方框架整合，依然使用MvBatis作为整合对象，之前我们已经使用xml方式整合了MyBatis，现在使用注解方式无非就是将xml标签替换为注解，将xml配置文件替换为配置类而已，原有xml方式整合配置如下

咱们之前 用xml方式是下面这样整合的, 现在使用注解的方式无非是用配置类替换xml文件

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>  
  
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
    <property name="basePackage" value="com.fancier.mapper"></property>  
</bean>
```


## 让我们来看看配置类怎么去写

首先呢, 得加上这么几个注解来代替部分标签

```java
@Configuration  //标注当前类是一个配置类（替代配置文件）+@Component  
//<context:component-scan base-package="com.itheima"/>  

@ComponentScan("com.fancier")  
//<context:property-placeholder location="classpath:jdbc.properties"/> 

@PropertySource("classpath:jdbc.properties")  
//<import resource=""></import>  


@MapperScan("com.fancier.mapper")//Mapper的接口扫描 
// <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer"></bean>
```

## 其次我们还要, 用注解的方式再来注入几个Bean

要注入 dataSource 对象(熟属性我们从之前注入的文件中拿), 再就是 sqlSessionFactoryBean 属性设置同样被注入的 dataSource

```java
@Bean  
public DataSource dataSource(  
        @Value("${jdbc.driver}") String driver,  
        @Value("${jdbc.url}") String url,  
        @Value("${jdbc.username}") String username,  
        @Value("${jdbc.password}") String password  
){  
    DruidDataSource dataSource = new DruidDataSource();  
    dataSource.setDriverClassName(driver);  
    dataSource.setUrl(url);  
    dataSource.setUsername(username);  
    dataSource.setPassword(password);  
    return dataSource;  
}  
  
@Bean  
public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){  
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();  
    sqlSessionFactoryBean.setDataSource(dataSource);  
    return sqlSessionFactoryBean;  
}
```

## 原理分析啊 （づ￣3￣）づ╭❤～

```java
@MapperScan("com.fancier.mapper")
```

咱来看看这个注解都干了些啥呢? 

```java
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.TYPE)  
@Documented  
//重点
@Import(MapperScannerRegistrar.class)  
//重点

@Repeatable(MapperScans.class)  
public @interface MapperScan {}
```

点进那个 `MapperScannerRegistrar` 看看, 它就是重点

```java
void registerBeanDefinitions(AnnotationMetadata annoMeta, AnnotationAttributes annoAttrs,  
    BeanDefinitionRegistry registry, String beanName) {    
		BeanDefinitionBuilder builder = 
		BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);
```

它是为了把 xml 方式需要注入的 `MapperScannerConfigurer` 给弄进 BDMap 的 

所以将它注入  Spring 中后就能注册那些 Mapper 了

