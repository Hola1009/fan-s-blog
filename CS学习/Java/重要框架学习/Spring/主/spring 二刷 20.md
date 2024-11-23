# 基于 AOP 的声明式事物控制
## Spring 事物编程概述
事务是开发中必不可少的东西，使用JDBC开发时，我们使用connnection对事务进行控制，使用MvBatis时，我们使用SalSession对事务进行控制，缺点显而易见，当我们切换数据库访问技术时，事务控制的方式总会变化Spring 就将这些技术基础上，提供了统一的控制事务的接口。Spring的事务分为:编程式事务控制 和 声明式事务控制

| 事物控制方式                                                                        | 解释                                                                         |
| ----------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 编程式事务控制                                                                       | 操作代码耦合到了一起，开发中不使用 Spring 提供了事务控制的类和方法，使用编码的方式对业务代码进行事务控制，事务控制代码和业务         |
| 声明式事务控制                        <br>                                      <br> | Spring 将事务控制的代码封装，对外提供了Xml和注解配置方式，通过配置的方式完成事务的控制可以达到事务控制与业务操作代码解耦合，开发中推荐使用 |

## Spring 事物编程相关的类

| 事物控制相关类                                   | 解释                                                    |
| ----------------------------------------- | ----------------------------------------------------- |
| 平台事务管理器PlatformTransactionManager<br><br> | 是一个接口标准，实现类都具备事务提交、回滚和获得事务对象的功能，<br>不同持久层框架可能会有不同实现方案 |
| 事务定义TransactioDefinition                  | 封装事务的隔离级别、传播行为、过期时间等属性信息                              |
| 事务状态 TransactionStatus   <br>             | 存储当前事务的状态信息，如果事务是否提交、是否回滚、是否有回滚点等                     |


虽然编程式事务控制我们不学习，但是编程式事务控制对应的这些类我们需要了解一下，因为我们在通过配置的方式进行声明式事务控制时也会看到这些类的影子

# AOP 思想解决事物
## 基于 xml 声明式事务控制
结合上面我们学习的 AOP 技术, 很容易想到, 可以使用 AOP 对 service 的方法进行事务增强.
- 目标类: 自定义的AccountServicelmpl，内部的方法是切点
- 通知类: Spring提供的，通知方法已经定义好，只需要配置即可

我们的分析
- 通知类是Spring提供的， 需要导入Spring事务的相关的坐标，
- 配置目标类AccountServicelmpl;
- 使用 advisor 标签配置切面

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


## 基于 xml 声明式事务控制详情


对于这个标签 \<tx:attributes\> 
```xml
<tx:attributes>  
    <!--  
        配置不同的方法的事务属性  
        name：方法名称  *代表通配符  添加操作addUser、addAccount、addOrders=>add*  
        isolation：事务的隔离级别，解决事务并发问题  
        timeout：超时时间 默认-1 单位是秒  
        read-only：是否只读，查询操作设置为只读  
        propagation：事务的传播行为，解决业务方法调用业务方法（事务嵌套问题）  
    -->  
   <tx:method name="*"/>  
  
</tx:attributes>
```

### isolation 事务的隔离级别

isolation属性:指定事务的隔离级别，事务并发存在三大问题:脏读、不可重复读、幻读/虚读。可以通过设置事务的隔离级别来保证并发问题的出现，常用的是READCOMMITTED 和 REPEATABLE READ

| islation属性       | 解释                                                      |
| ---------------- | ------------------------------------------------------- |
| DEFAULT          | 默认隔离级别，取决于当前数据库隔离级别，例如MySQL默认隔离级别是REPEATABLE READ       |
| READ UNCOMMITTED | A事务可以读取到B事务尚未提交的事务记录，不能解决任何并发问题，安全性最低，性能最高              |
| READ COMMITTED   | A事务只能读取到其他事务已经提交的记录，不能读取到未提交的记录。可以解决脏读问题，但是不能解决不可重复读和幻读 |
| REPEATABLE READ  | A事务多次从数据库读取某条记录结果一致，可以解决不可重复读，不可以解决幻读                   |
| SERIALIZABLE     | 串行化，可以解决任何并发问题，安全性最高，但是性能最低                             |

### propagation 事务的传播行为

propagation属性:设置事务的传播行为，主要解决是A方法调用B方法时，事务的传播方式问题的，例如:使用
单方的事务，还是A和B都使用自己的事务等。事务的传播行为有如下七种属性值可配置

| 事务传播行为        | 解释                                            |
| ------------- | --------------------------------------------- |
| REQUIRED(默认值) | A调用B，B需要事务，如果A有事务B就加入A的事务中，如果A没有事务，B就自己创建一个事务 |
| REQUIRED NEW  | A调用B，B需要新事务，如果A有事务就挂起，B自己创建一个新的事务             |
| SUPPORTS      | A调用B，B有无事务无所谓，A有事务就加入到A事务中，A无事务B就以非事务方式执行     |
| NOT SUPPORTS  | A调用B，B以无事务方式执行，A如有事务则挂起                       |
| NEVER         | A调用B，B以无事务方式执行，A如有事务则抛出异常                     |
| MANDATORY     | A调用B，B要加入A的事务中，如果A无事务就抛出异常                    |
| NESTED        | A调用B，B创建一个新事务，A有事务就作为嵌套事务存在，A没事务就以创建的新事务执行    |
