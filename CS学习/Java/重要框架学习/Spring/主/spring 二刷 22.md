# Spring 整合 Web 环境

在 Java 语言范围内, web 层框架都是基于Javaweb 基础组件完成的, 所以有必要复习一下Javaweb组件的特点
## Javaweb 三大组件及环境特点

| 组件       | 作用                      | 特点                                                                                                                        |
| -------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Servlet  | 服务端小程序，负责接收客户端请求并作出响应的  | 单例对象，默认第一次访问创建，可以通过配置指定服务器启动就创建，Servlet创建完毕会执行初始化init方法。每个Servlet有一个service方法，每次访问都会执行service方法，但是缺点是一个业务功能就需要配置一个Servlet |
| Filter   | 过滤器，负责对客户端请求进行过滤操作的     | 单例对象，服务器启动时就创建，对象创建完毕执行init方法，对客户端的请求进行过滤，符合要求的放行，不符合要求的直接响应客户端，执行过滤的核心方法doFilter                                         |
| Listener | 监听器，负责对域对象的创建和属性变化进行监听的 | 根据类型和作用不同，又可分为监听域对象创建销毁和域对象属性内容变化的根据监听的域不同，又可以分为监听Request域的，监听Session域的，监听ServletContext城的                                |


## Spring 整合web环江的思路及实现
在进行java开发时要遵循三层架构+MVC, Spring操作最核心的就是Spring容器, web层需要注入service, sevice层需要注入Dao(Mapper), web 层使用Servlet技术充当的化, 需要在Servlet中获得Spring容器

![[Pasted image 20240511212622.png]]

web 层代码如果去编写AnnotationConfigApplicationContext的代码,  那么配置类重复加载了, Spring容器也重复被创建了, 不能每次向获得一个bean就创建一个容器

所以, 我们现在的诉求很简单, 
- 只创建一个容器, 配置类只加载一次
- 最好web服务器启动时, 就执行第1不操作, 后续直接从容器中获取 Bean 使用即可
- ApplicationContext的引用需要在web层任何位置都可以获得到

解决方法

- 在 ServletContextListenter 的 contextInitaliezed 方法中执行 配置文件加载和容器创建, 
- 将创建的容器放到 servletContext 域中

在 web.xml 中添加自己编写的 Linstener
实现了 ServletContextListener 接口
在 contextInitialized 方法中创建容器
在调用sce 的getServletContext().setAttribute(..) 方法将容器放进去

这样就可以使用请求方法的入参中的request获取域对象, 从而获得 spring 容器

不足: 配置文件的代码给写死了,  
解决: 写在配置文件里面

![[Pasted image 20240511221801.png|left|600]]

![[Pasted image 20240511222939.png]]

## Spring的web开发组件 spring-web

web.xml中配置的监听器就得时 spring-web 中的了
![[Pasted image 20240511223927.png]]

在请求方法里就可以通过这个方法来获取 容器了
![[Pasted image 20240511224142.png]]

## 核心配置类的方式怎么配置
监听器底层
![[Pasted image 20240511230204.png]]

根据是否存在这个属性来判断是否通过核心配置类的方式来创建容器
![[Pasted image 20240511230412.png]]

![[Pasted image 20240511230451.png]]


