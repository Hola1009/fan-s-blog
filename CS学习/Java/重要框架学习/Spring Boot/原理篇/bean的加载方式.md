- 在 xml 里面 用标签声明
- 在xml中指定扫描路径, 再用注解来 声明
- 使用配置类 + 注解来 声明
- 使用BeanFactory


怎么在使用全注解的基础上注入 在 xml 中声明的bean, 弄上这么一个类
![[Pasted image 20240523160907.png]]



加了这个注解后 去调用 被 @Bean 修饰的方式时 默认从容器中去拿, 这个方法的默认值时 true
![[Pasted image 20240523193527.png]]

![[Pasted image 20240523194309.png|left|600]]

也可以拿着上下文对象手动注册 bean



或者 实现 ImportSelector 在 @import 这个实现的类

重写 的 selectImports() 方法的入参 可以 拿到 被 import 那个类的信息: 比如 注解,  类的全限定名;

然后可以根据 这个被 import 的类 的信息, 去控制 它重写方法里的bean的注册



或者实现 ImportBeanDefinitionRegistrar 重写方法

可以注册 beanDefintion 

![[Pasted image 20240523201051.png|left|600]]


或者实现 BeanDefinitionRegistryPostProcessor 接口

实现 bean工厂后处理器 重写方法 往BeanDefinitionMap 中注入 BD

这个方法注册bean的优先级很高, 可以用来替换第三方要注的bean

![[Pasted image 20240523202402.png]]

# Bean 的加载控制
bean的加载控制指根据特定情况对bean进行选择性加载以达到适用于项目的目标

对bean进行选择性的加载

可以 进行控制的 加载方式

- 使用上下文对象来注册
- 实现 ImportSelector
- @import 实现 ImportBeanDefinitionRegistrar 的类
- 实现 BeanDefinitonRegistryPostProcessor 的类 并通过 @Import 导入

## @Conditional

使用@Conditional注解的派生注解设置各种组合条件控制bean的加载

加在 @Bean 得方法上 符合条件才 注入 Bean

当然写在类上也是欧克得

![[Pasted image 20240523212915.png|left|600]]




# ImportSelector
在 被 @Configuratin 标注的配置类上, 可以用 @Import 引入其他配置类, 还可以 引入主人公 ImportSelector

可以用于注册第三方bean

```java
@Override
public String[] selectImports(AnnotationMetadata importingClassMetadata) {
	return new String[] {HelloServiceA.class.getName(), HelloServiceB.class.getName()};
}
```
这个从写的方法返回的就是需要被注册的 bean 的全限定类名 AnnotationMetadata importingClassMetadata 入参可以拿到被@Import 标注的类的信息, 读取一些其他的配置, 从而可以做到 操控bean的注入


# ImportBeanDefinitionRegistrar
实现后重写的方法用于生成 BeanDefinition 并注册到 BeanDefinitionRegistry 中, 为后续实例化 Bean 做准备, 

```java
public class CustomRegistrar implements ImportBeanDefinitionRegistrar { 
	@Override 
	public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) { 
	// 在此处编写动态注册Bean的逻辑 
	// 例如根据条件注册不同的Bean 
	} 
}
```

