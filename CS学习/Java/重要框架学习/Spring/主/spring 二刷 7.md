### SpringBean 的生命周期
Spring Bean的生命周期是从 Bean 实例化之后，即通过反射创建出对象之后，到Bean成为一个完整对象，最终存储到单例池中，这个过程被称为Spring Bean的生命周期。Spring Bean的生命周期大体上分为三个阶段:
- **Bean 的实例化阶段**: Spring 框架会取出 BeanDefinition 的信息进行判断当前Bean的范围是否是singleton的, 是否不是延迟加载的, 是否不是FactoryBean等，最终将一个普通的singleton的Bean通过反射进行实例化;
- **Bean的初始化阶段**: Bean创建之后还仅仅是个"半成品"，还需要对Bean实例的属性进行填充、执行一些 Aware 接口方法、执行BeanPostProcessor方法、执行InitializingBean接口的初始化方法、执行自定义初始化init方法等。该阶段是Spring最具技术含量和复杂度的阶段，Aop增强功能，后面要学习的Spring的注解功能等spring高频面试题Bean的循环引用问题都是在这个阶段体现的;
- **Bean的完成阶段**: 经过初始化阶段，Bean就成为了一个完整的SpringBean，被存储到单例池singletonObjects中去了，即完成了Spring Bean的整个生命周期。

#### Bean 的初始化
由于Bean的初始化阶段的步骤比较复杂，所以着重研究Bean的初始化阶段Spring Bean的初始化过程涉及如下几个过程:
- Bean实例的属性填充
- Aware接口属性注入
- BeanPostProcessor的before()方法回调 
- InitializingBean接口的初始化方法回调
- 自定义初始化方法init回调
- BeanPostProcessor的after()方法回调
##### Bean实例的属性填充
BeanDefinition 中有对当前Bean实体的注入信息通过属性propertyValues(在BeanDefinition的value里)进行了存储

Spring在进行属性注入时，会分为如下几种情况:
- 注入普通属性，String、int或存储基本类型的集合时，直接通过set方法的反射设置注入进去
- 单向对象引用属性时，从容器中getBean获取后通过set方法反射设置进去，如果容器中没有，则先创建被注入对象Bean实例后(完成整个生命周期)，在进行注入操作;
- 涉及了循环引用(循环依赖)问题，下面会详细阐述解决方案。注入双向对象引用属性时，就比较复杂了

###### 循环依赖
![[Pasted image 20240413222411.png|left]]


##### 三级缓存
Spring提供了三级缓存存储 完整Bean实例 和 半成品Bean实例 ，用于解决循环引用问题在DefaultListableBeanFactory的上四级父类DefaultSingletonBeanRegistry中提供如下三个Map:

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	//单例池 最终存储单例Bean成品的容器
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); 
    //早期单例池 缓存半成品对象 且当前对象已经备其他对象引用过了
    private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
    //单例Bean工厂池 缓存未被引用的对象
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);   
}
```

假如 A 和 B

先实例化A时将其放入后三级缓存, 然后到了初始化阶段, 为其注入属性, 到容器中去找B对象, 发现没有,再到缓存中去找,发现也没有, 就创建实例化B对象将其放入三级缓存, 之后初始化B对象, 在三级缓存中发现A对象, 完成了B对象的属性注入后将A对象放入二级缓存, 之后再回头去完成A对象的初始化  