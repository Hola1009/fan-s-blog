# web层 MVC 框架思路与设计思路

## MVC 框架思想及设计思路

Java 程序员在开发一般都是 MVC + 三层架构, MVC 是 web 开发模式, 传统的 Javaweb 技术栈实现的 MVC 如下:

![[Pasted image 20240428192734.png]]

原始Javaweb开发中，Servlet充当Controller的角色，Jsp充当View角色，JavaBean充当模型角色，后期Ajax异步流行后，在加上现在前后端分离开发模式成熟后，View就被原始Html+Vue替代。原始Javaweb开发中，Service充当Controller有很多弊端，显而易见的有如下几个:

| Servlet作为Controller的问题                      | 解决思路和方案                                        |
| ------------------------------------------- | ---------------------------------------------- |
| 每个业务功能请求都对应一个Servlet                        | 根据业务模块去划分Controller                            |
| 每个Servlet的业务操作太繁琐                           | 将通用行为, 功能经行抽取封装                                |
| Servlet 获得 Spring 容器的组件只能通过客户端代码去获取，不能优雅的整合 | 通过Spring的扩展点，:去封装一个框架，从原有的Servlet完全接手过来web层的业务 |

负责共有行为的 Servlet 称为前端控制器, 负责业务行为的 JavaBean 称为控制器 Controller
![[Pasted image 20240428193406.png]]

共有行为 +   特有行为 组成 web 层
前端控制器完成共有行为,   具体的bean万层特有行为


# SpringMVC 简介
## SpringMVC 概述
SpringMVC是一个基于Spring开发的MVC轻量级框架，Spring3.0后发布的组件，SpringMVC和Spring可以无缝整合，使用`DispatcherServlet` 作为前端控制器，且内部提供了处理器映射器、处理器适配器、视图解析器等组件，可以简化JavaBean封装，Json转化、文件上传等操作。

![[Pasted image 20240428194521.png]]

## SpringMVC 快熟入门

![[Pasted image 20240428201122.png|left|600]]



在pom文件 导入mvc的坐标
![[Pasted image 20240512111232.png|left|400]]

在web.xml 配 dispatcherServlet 
![[Pasted image 20240512111415.png]]

编写控制器 配置映射路径
![[Pasted image 20240512111540.png|left|300]]

还得添加组件扫描, 这样才能进入到容器
![[Pasted image 20240512111937.png]]

然后再web.xml中指定加载mvc的配置文件
![[Pasted image 20240512112114.png]]


servlet 一般是服务启动时才创建, 我们想让它在服务启动时就创建
![[Pasted image 20240512112204.png]]

控制器的方法默认是返回一个 视图


## controller 直接注入 容器中的 bean(service)

需要让 web 去集成 spring 

先写spirng的配置文件 编写包的扫描路径
再在 web.xml 文件中配置 
![[Pasted image 20240512113452.png]]


## SpringMVC 关键组件解析

上面已经完成的快速入门的操作，也在不知不觉中完成的Spring和SpringMVC的整合，我们只需要按照规则去定义Controller和业务方法就可以。但是在这个过程中，肯定是很多核心功能类参与到其中，这些核心功能类，般称为组件。当请求到达服务器时，是哪个组件接收的请求，是哪个组件帮我们找到的Controller，是哪个组件帮我们调用的方法，又是哪个组件最终解析的视图

| 组件                     | 描述                                                   | 常用组件                         |
| ---------------------- | ---------------------------------------------------- | ---------------------------- |
| 处理器映射器: HandlerMapping | 匹配映射路径对应的Handler，返回可执行的处理器链对象HandlerExecutionChain对象 | RequestMappingHandlerMapping |
| 处理器适配器: HandlerAdapter | 匹配HandlerExecutionch:in对应的适配器进行处理器调用，返回视图模型对象        | RequestMappingHandlerAdapter |
| 视图解析器: ViewResolver    | 对视图模型对象进行解析                                          | InternalResourceViewResolver |


![[Pasted image 20240429092217.png|left|600]]



## SpringMVC 加载组件的策略

默认加载的组件
![[Pasted image 20240512115111.png]]

![[Pasted image 20240512115123.png]]

默认的视图解析器就不管了

dispatcherServlet 中维护着几个集合, 用来保存这些组件
![[Pasted image 20240512115337.png|left|600]]


在 disp.. 的init 方法中加载这些文件

![[Pasted image 20240512115434.png]]

初始化方法中, 会先在容器中找有没有对应的组件, 有就用容器中已经存在的, 没有就用默认的组件

如果需要自定义的话, 就需要把自定义的组件放入到容器当中
![[Pasted image 20240512120058.png]]
# 请求路径资源的配置

## SpringMVC 的请求处理
### 请求映射路径的配置
配置映射路径，映射器处理器才能找到Controller的方法资源，目前主流映射路径配置方式就是 @RequestMapping

![[Pasted image 20240429192436.png]]


一个资源可以同时配置多个请求地址
```java
@RequestMapping({"/show2", "/showXxx"})
```

## 请求数据的接收

用 get 方法 把实体的属性放在url中 

![[Pasted image 20240512120848.png]]

用post 将实体属性放在请求体中 
得加 @RequestBody 注解

![[Pasted image 20240512121344.png]]

响应数据还得人为转化成json格式字符串

简洁得办法
指定json格式的消息转换器 
![[Pasted image 20240512121455.png]]


## 请求数据的接收
![[Pasted image 20240512122833.png]]

还需要手动配置文件上传解析器, 而解析器默认未被注册, 所以手动注册
还要导包, 因为顶层用到了
![[Pasted image 20240512123215.png]]


在 postman 这样提交
![[Pasted image 20240512123842.png]]

## 服务端获取客户端的请求头
这样去获得
获得一个具体的头
![[Pasted image 20240512124428.png]]

获取所有的头
![[Pasted image 20240512124735.png]]

## 获得客户端携带的cookie

![[Pasted image 20240512124818.png]]


## 获取域当中的数据

spirng 会自动帮我们自动注入 request 对象

存入数据
和
获取数据

![[Pasted image 20240512124957.png]]
