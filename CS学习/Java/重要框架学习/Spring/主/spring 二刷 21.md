## xml事务控制原理分析

它是使用 AOP 来完成事务控制的, 首先 tx:advice 这个标签像容器中注入了 事务连接器的对像, tx:advice 这个标签的id就是属于 这个事务连接器对象的, 这个事务连接器就是用来完成增强的

来看看xml文件是怎么写的

```xml
<!--配置平台事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
    <property name="dataSource" ref="dataSource"/>  
</bean>  
<!--配置Spring提供的Advice-->    
<tx:advice id="txAdvice" transaction-manager="transactionManager">  
    <tx:attributes>  
        <tx:method name="*"/>  
    </tx:attributes>  
</tx:advice>  
  
<aop:config>  
    <aop:pointcut id="txPointcut" expression="execution(* com.fancier.service.impl.*.*(..))"/>  
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>  
</aop:config>
```

来看看事务连接器这个类
```java
public class TransactionInterceptor extends TransactionAspectSupport implements MethodInterceptor, Serializable
```

它继承了 `MethodInterceptor` 这个接口, 是用来做环绕通知的

看看它的 invoke 方法

```java
@Nullable  
public Object invoke(MethodInvocation invocation) throws Throwable {  
	//拿到目标对象 
    Class<?> targetClass = invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null;  
    Method var10001 = invocation.getMethod();  
    invocation.getClass();  
    return this.invokeWithinTransaction(var10001, targetClass, invocation::proceed);  
}
```

点进最后一个方法, 它里面有这样一段, 是用来创建事物的, 返回的是一个事务的对象(这个时候已经开事务了)

```java
TransactionInfo txInfo = this.createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);
```

> [!tip]
ptm 之前注入的`平台事务管理器`


然后再 执行 目标方法
```java
retVal = invocation.proceedWithInvocation();
```

执行完目标方法后没有报错就提交事务

```java
this.commitTransactionAfterReturning(txInfo);
```

## 注解方式配置声明式事务
### 半注解

如果要对一个方法进行事务控制, 在方法上加上这些注解, 每个属性的含义在这里看 [[spring 二刷 20#基于 xml 声明式事务控制详情|属性详情]]

```java
@Transactional(isolation = Isolation.READ_COMMITTED,propagation = Propagation.REQUIRED)
```

如果要对这个类的所有方法生效, 就加在类上

加完了后, 还要再 xml 配置文件中 加事务的自动代理 (注解驱动)

```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```


如果平台事务给平台事务管理器配的就是这个名字, 那么这一段 `transaction-manager="transactionManager"` 可以不写, 但是如果 id 取的是其他的名字 

### 怎么弄成全注解

```java
@Configuration  
@ComponentScan("com.itheima")  
@PropertySource("classpath:jdbc.properties")  
@MapperScan("com.fancier.mapper")  
@EnableTransactionManagement //<tx:annotation-driven/>
public class SpringConfig {  
  
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
  
    @Bean  
    public DataSourceTransactionManager transactionManager(DataSource dataSource){  
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();  
        dataSourceTransactionManager.setDataSource(dataSource);  
        return dataSourceTransactionManager;  
    }  
}
```

## 切点表达式配置方法 与 事务属性配置方法 区别
切点表达式是 筛选对哪些方法进行增强, 而事务实行配置表示为哪些需要增强的事务方法进行属性的配置


![[Pasted image 20240427174053.png|left|600]]


