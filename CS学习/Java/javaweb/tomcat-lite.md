说明: Tomcat 有三种运行模式（BIO, NIO, APR ）, 因为核心讲解的是 Tomcat 如何接收客户端请求，解析请求, 调用 Servlet , 并返回结果的 机制流程, 采用 BIO 线程模型来模拟 
![[Pasted image 20241114003530.png|600]]

怎么编写
自动定义 tomcat 接收 socket 连接然后新建线程来维护连接

## 封装 Http
### HttpRequest

## Tomcat 底层机制
维护了两个线程安全的 Map
一个存 ServletName 和 实例 的映射关系
一个存 Url 和 ServletName 的映射关系

> [!question] 怎么初始化这两个容器
> 使用 dom4j 解析 web.xml 文件, 然后注册 servlet 和 url

> [!warning] CurrentHashMap 不能存 null key


