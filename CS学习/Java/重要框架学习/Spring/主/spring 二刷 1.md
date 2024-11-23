### 传统JavaWeb开发困惑以及解决方案
-  问题一: 层与层之间紧密耦合, 接口也具体实现紧密耦合
- 解决方案: 第三方提供Bean对象
- 问题二: 通用的事物功能耦合再业务代码中, 通用日志功能耦合在业务代码中
- 解决方案: 第三方提供增强的Bean对象

### IoC, DI 和 AOP思想的提出
#### IoC Inversion of Control
控制翻转, 将创建Bean的权利交给第三方
#### DI Dependency Injection
依赖注入,  将Bean之间的关系交给第三方管理
@Autowired XXX xxx;
xxx 具体的实例交给Spring去赋值
#### AOP Aspect Oriented Programming
面向切面的编程, 功能是横向提取, 就是上面提到的增强Bean对象


### 框架的概念
此框架未程序中的框架, 即一些经常要用到的代码, 就像盖房子要用到钢筋水泥, 我不会直接去造钢筋水泥, 而是直接去买, 直接用
#### 特点
- 基础技术上抽取的通用方案
- 半成品
- 使用了大量设计模式, 算法, 底层代码操作技术,
- 拓展性
- 方便我们写代码

#### java中常用的框架
分为两类
- 基础框架: 我们要学习的Spring 之类的
- 服务框架: 特定领域要用的

### Spring框架的诞生
不过多介绍了
### BeanFactory 简单入门
分析下Spring的BeanFactory的开发步骤:
![[BeanFactory工作流程|left|500]]

测试一下
1) 导入meaven坐标
2) 定义Bean类
3) 间Bean类的信息配置到beans.xml里
4) 编写测试代码
```java
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.3.7</version>  
</dependency>
```

```xml
<bean id="beanDemo" class="com.example.test1.Beans.BeanDemo"></bean>
```

```java
public class BeanDemoTest {  
    public static void main(String[] args) {  
        //创建一个工厂对象  
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();  
        //创建一个读取器  
        XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);  
        //加载xml文件  
        xmlBeanDefinitionReader.loadBeanDefinitions("BeanConfig.xml");  
        //根据id获取Bean实例对象  
        Object beanDome = beanFactory.getBean("beanDemo");  
        System.out.println(beanDome);  
    }  
}
```

#### 在Bean工厂内部去完成注入操作
1. 在需要注入的类里写注入要调用的set方法
2. 在xml文件中配置属性注入
```xml
<bean id="beanDemo" class="com.example.test1.Beans.BeanDemo">  
        <property name="dao" ref="dao"></property>  
</bean>
```
name为要配置的属性名setXxx xxx就是name, ref要注入的bean的id


### 使用 ApplicationContext 

```java
ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("BeanConfig.xml");  
Object beanDemo = applicationContext.getBean("beanDemo");  
System.out.println(beanDemo);
```

