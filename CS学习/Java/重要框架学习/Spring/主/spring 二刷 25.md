## 拦截器 Interceptor 简介
SpringMVC的拦截器Interceptor规范，主要是对Controller资源访问时进行拦截操作的技术，当然拦截后可以进行权限控制，功能增强等都是可以的。拦截器有点类似Javaweb 开发中的Filter，拦截器与Filter的区别如下图:

![[Pasted image 20240503215337.png]]

![[Pasted image 20240503220349.png]]


![[Pasted image 20240503220918.png]]



## 过滤器 Filter
- 概念: Filter 过滤器, 是 javaWeb 三大组件
- 过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能。
- 过滤器一般完成一些通用的操作，比如:登录校验、统一编码处理、敏感字符处理等


## 拦截器
怎么写拦截器

![[Pasted image 20240513191025.png]]

在 mvc 配置文件中

![[Pasted image 20240513191200.png]]










pre方法执行 post 次之, complete最次

有多个拦截器的话, 就按顺序先执行先注册的pre方法, 在按照相反的顺序执行 post 方法 和 complete 方法, 就像洋葱一样一层一层的包裹

![[Pasted image 20240504222157.png]]

存在拦截器pre返回false的情况 post 方法都不执行

![[Pasted image 20240504222313.png]]

拦截器执行原理

![[Pasted image 20240504223157.png]]

处理器映射器 会找到可以执行的 interceptor 和目标资源
然后把他们封装成对象,  然后交给 adapter 进行处理

最终在 DispatcherServlet 执行如下方法

![[Pasted image 20240513193033.png]]

根据请求获取可以执行的 handler
![[Pasted image 20240513193145.png]]

![[Pasted image 20240513193103.png]]



执行preHandle
![[Pasted image 20240513193318.png]]

![[Pasted image 20240513193336.png]]

