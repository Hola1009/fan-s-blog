
### Spring 配置非自定义Bean
#### 要考虑的方面
- 被配置的 Bean 的实例化方式是什么? [无参构造器 , 静态工厂方式还是实例工厂方式](http://t.csdnimg.cn/dstzJ)
- 被配置的 Bean 是否需要注入必要属性

#### 以配置 Durid 数据源交给 Spring 管理为例
先导入 Durid 的坐标
```xml
<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>5.1.49</version>  
</dependency>  
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.1.23</version>  
</dependency>
```

正常是如何使用 Druid 数据库连接池的
```java
DruidDataSource druidDataSource = new DruidDataSource();  
druidDataSource.setDriverClassName();  
druidDataSource.setUrl();  
druidDataSource.setUsername();  
druidDataSource.setPassword();
```

所以在注入 Bean 对象的时候也要为这些属性赋值

```java
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">  
	<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
	...  
</bean>
```

#### 配置连接
正常通过代码来获取连接
```java
Class.forName("com.mysql.jdbc.Driver");  
Connection connection = DriverManager.getConnection("", "", "");
```
通过xml注入, 静态工厂方法来加载驱动
```xml
<bean id = "clazz" class="java.lang.Class" factory-method="forName">  
        <constructor-arg name="className" value="com.mysql.jdbc.Driver"></constructor-arg>  
</bean>  
<bean id="connection" class="java.sql.DriverManager" factory-method="getConnection">  
        <constructor-arg name="url" value=""></constructor-arg>  
        <constructor-arg name="user" value="root"></constructor-arg>  
        <constructor-arg name="password" value=""></constructor-arg>  
</bean>
```

### Bean 实例化的基本流程
Spring 容器在初始化时, 会将 xml 配置的\<bean\>的信息封装成一个 BeanDefinition 对象, 所有的BeanDefinition存储到一个名为beanDefinitionMap的Map集合中去，Spring框架在对该Map进行遍历，使用反射创建Bean实例对象，创建好的Bean对象存储在一个名为singletonObjects的Map集合中，当调用getBean方法时则最终从该Map集合中取出Bean实例对象返回。





### Spring 后处理器
`Spring的后处理器`是Spring对外开发的重要扩展点，`允许`我们`介入`到Bean的`整个实例化流程`中来，以达到`动态注册`BeanDefinition，`动态修改`BeanDefinition，以及动态修改Bean的作用。Spring主要有两种后处理器
- BeanFactoryPostProcessor:Bean工厂后处理器，在BeanDefinitionMap填充完毕，Bean实例化之前执行
- BeanPostProcessor:Bean后处理器，一般在Bean实例化之后，填充到单例池singletonObjects之前执行。

###### 实现后处理器
```java
public class MyHostProcess implements BeanFactoryPostProcessor {  
    @Override  
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {  
        
          
    }  
}
```

###### 注入 Spring 容器
```xml
<bean class="com.example.test1.process.MyHostProcess"></bean>
```

###### 后处理修改全类名实验
```java
@Slf4j  
public class MyHostProcess implements BeanFactoryPostProcessor {  
    @Override  
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {  
        log.info("PostProcessBeanFactory");  
        BeanDefinition beanDefinition = beanFactory.getBeanDefinition("dao");  
        beanDefinition.setBeanClassName("com.example.test1.Beans.BeanDemo");  
    }  
}
```

###### 结果
```java
System.out.println(applicationContext.getBean("dao"));
```
```
com.example.test1.Beans.BeanDemo@b7dd107
```



###### 加入新的BeanDefinition
要将参数表过来的工厂对象转成它的子类 才能注册新的 definition
```java
@Slf4j  
public class MyHostProcess implements BeanFactoryPostProcessor {  
    @Override  
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {  
        log.info("PostProcessBeanFactory");  
        RootBeanDefinition beanDefinition = new RootBeanDefinition();  
        beanDefinition.setBeanClass(PersonDao.class);  
        DefaultListableBeanFactory factory = (DefaultListableBeanFactory) beanFactory;  
        factory.registerBeanDefinition("personDao", beanDefinition);  
    }  
}
```

