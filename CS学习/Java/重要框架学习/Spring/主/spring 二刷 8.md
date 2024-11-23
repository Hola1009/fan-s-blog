## 常用的Aware
Aware 接口是一种框架辅助属性注入的思想, 其框架中也可以看到类似的接口; 框架具备 `高度封装性` , 我们接触到一般都是业务代码, 一个 `底层功能 API 不能轻易的获取` 到, 但是这不意味着永远用不到这些对象, `如果用到了`, 就可以`使用`框架提供类似 `Aware的接口`  , **让框架给我们注入该对象**

| Aware接口                 | 回调方法                                                         | 作用                                      |
| ----------------------- | ------------------------------------------------------------ | --------------------------------------- |
| ServletContextAware     | setServletContext(ServletContextcontext)                     | Spring框架回调方法注入ServletContext对象web环境下才生效 |
| BeanFactoryAware        | setBeanFactory(BeanFactory factory)                          | Spring框架回调方法注入beanFactory对象             |
| BeanNameAware           | setBeanName(String beanName)                                 | Spring框架回调方法注入当前Bean在容器中的beanName       |
| ApplicationContextAware | setApplicationContext(ApplicationContext applicationContext) | Spring框架回调方法注入applicationContext对象      |
如果要用到 BeanFactory 就 实现 BeanFactoryAware 的 setBeanFactory() 方法, 让 Spring 为我们注入


## Spring IoC 整体流程图总结
1. 把 xml 文件中的Bean标签 抽取成 Bean 定义对象 (保存Bean标签的信息) 存取到 BeanDefinitionMap 中去, 再遍历 BeanDefinitionMap 去创建Bean对象, 并将其存到单例池中去
2. 在 BeanDefinitionMap 填充完毕要实例化Bean之前会经过Bean工厂后处理器, 创建完Bean对象后有, 然后再经过Bean处理器的 前置 后置 方法后 再放入单例池

![[Pasted image 20240416212000.png|left]]

