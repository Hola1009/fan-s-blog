### 通过自定义工厂实例化Bean
- 静态工厂方法实例化Bean //不许要现有对象
- 实例工厂方法实例化Bean
- 实现FactoryBean规范延迟实例化Bean

> 同样也可以使用 \<constructor-arg\> 标签给工厂方法传参数
#### 静态工厂方法
先静态创建工厂类, 再到配置文件中这样写

```xml
<bean id="dao1" class="com.example.test1.beanFactory.BeanFactory1" factory-method="dao"></bean>
```
##### 用处
- 可以在创建之前完成一些业务操作
- 引入第三方jar包中, 需要通过static方法才能得到的Bean
#### 实例工厂方法
先将实例工厂方法的工厂类注入 Spring 中, 再将实例方法的bean注入spirng
```xml
<bean id="factory2" class="com.example.test1.beanFactory.BeanFactory2"></bean>  
  
<bean id = "dao2" factory-bean="factory2" factory-method="dao"></bean> 
```

`如果是工厂接口的实例化 不许要标注方法名`
##### 用处
引入第三方包中的实例化工厂方法的Bean

### 实现FactoryBean接口延迟实例化Bean
实现一个FB类, 

```xml
<bean id = "dao3" class="com.example.test1.beanFactory.BeanFactory3"></bean>
```
当 getBean("dao3") 时实际调用了工厂类的 getBean() 方法


>在单例池中 k dao 对应的v 是 工厂的地址
  只有getObject() 时会在 factoryBeanObjectCache 中加入 k dao v bean地址 的键值对

### Bean的依赖注入方式
Bean的依赖注入有两种方式:

| 注入方式               | 配置方法                                                                                            |
| ------------------ | ----------------------------------------------------------------------------------------------- |
| 通过 Bean 属性的set方法注入 | \<property name="userDao" ref="userDao"/>  <br>\<property name="userDao" value="haohao"/>       |
| 通过构造Bean的方法进行注入    | \<constructor-arg name="name" ref="userDao"/><br>\<constructor-arg name="name" value="haohao"/> |
ref 表示 引用数据类型
value 表示基本数据类型
#### 依赖注入的三种类型
- 普通数据
- 引用数据类型
- 集合数据类型

### xml文件的写法
存储基本数据类型的集合
```xml
<bean id="dao" class="com.example.test1.Dao.Dao">  
	<property name="list">  
		<list>  
			<value></value>  
			<value></value>                        
			<value></value>
		</list>   
	</property>  
</bean>
```
存储引用数据类型的集合就用ref 代替 value 
如果集合容器时 set 则用set标签
如果容器时map 用map标签 (键值对的标签为 ,  entry key value)
### 自动装配

写xml注入为手动装配
再 bean 标签里写 autowire 属性实现配置自动专配, 其属性有两个
- byName: 通过属性名去容器中匹配
- byType: 通过类型去容器中匹配

```xml
<bean id="beanDemo" class="com.example.test1.Beans.BeanDemo" autowire="byName"></bean>
```