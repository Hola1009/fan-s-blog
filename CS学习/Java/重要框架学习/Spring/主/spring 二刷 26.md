
# MVC 全注解开发
### 消除 mvc.xml 

mvc.xml 中
有组件扫描 -> @ComponentScan
![[Pasted image 20240513195019.png]]

非自定义的bean -> @Bean
![[Pasted image 20240513195040.png]]

非bean的配置 -> @EnableWebMvc
![[Pasted image 20240513195110.png]]


## EnableWebMvc

![[Pasted image 20240513195524.png]]

![[Pasted image 20240513200532.png]]

这个类有一个 这个方法 用来执行自定义的配置类


![[Pasted image 20240513200716.png]]


这些自定义配置类都实现了  这个 接口![[Pasted image 20240513200939.png]]
可以用来向注册拦截器 默认静态资源处理器 等等

![[Pasted image 20240513200858.png]]



复写这些方法即可 代替 非自定义的bean的注入了

![[Pasted image 20240513201303.png]]




web 中的 这段可以不用写了, 但怎么让核心配置类起作用呢
![[Pasted image 20240513201553.png]]

看看 DispatcherServlet 中是怎么加载初始化参数的. 找初始化方法

contextClass

![[Pasted image 20240513201839.png]]

先创建一个类去集成它 AnnotationConfigWebApplicationContext


![[Pasted image 20240513202024.png]]

## 消除 web.xml

![[Pasted image 20240514110300.png]]


web.xml 中要写啥
spring 的入口
![[Pasted image 20240514105553.png]]

springmvc 的入口
![[Pasted image 20240514111912.png]]

前端控制器映射路径
![[Pasted image 20240514105625.png]]


![[Pasted image 20240514112125.png]]

Spring 中 也有

![[Pasted image 20240514112057.png]]

它扫描的是这个接口的实现类
![[Pasted image 20240514112216.png]]

所以我们需要实现一个接口

![[Pasted image 20240514112522.png]]

![[Pasted image 20240514112343.png]]

# SpringMVC 的组件原理刨析

![[Pasted image 20240514112641.png]]


前端控制器的继承体系
![[Pasted image 20240514112850.png]]

看看源码

创建时先执行init方法

