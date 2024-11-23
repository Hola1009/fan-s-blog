Tomcat 本质是一个 Java 程序, 但这个 Java 程序可以处理来自浏览器的 HTTP 请求

## Tomcat 
### 启动和安装
安装不介绍了

启动: 双击bin目录下 startup.bat 默认端口 8080

> [!tip]
> netstat -anb 查看端口占用情况

### 目录结构
![[Pasted image 20241112002516.png|400]]

server.xml 用于配置 tomcat 的基本设置(启动端口, 主机名)
web.xml 用于指定运行时配置(比如 servlet 等)
webapps 是用来存放 web 应用的

### 关闭 tomcat
默认 8009 等待接收关闭请求, shutdown.bat 运行后会发送关闭请求到 8009

### tomcat 部署 web 应用
JavaWeb 程序/应用/工程目录结构
![[Pasted image 20241112003906.png|400]]
方式1: 将web工程目录拷贝到 Tomcat 的 webapps 目录下
方式2: 通过配置文件来
url 中如果没有写路径名则默认访问的是root中的资源

### 浏览器请求资源 UML 分析
通过时序图了解业务核心
![[Pasted image 20241112005641.png|600]]


### idea 开发部署 web 应用
创建 JavaWeb 项目, 在 Run/Debug Configurations 里配置 tomcat
![[Pasted image 20241112112208.png]]
### 项目工程目录
![[Pasted image 20241112101214.png|500]]

## Servlet
Tomcat 相当于 servlet 的容器, servlet 是与用户进行交互的工具, 在开发 web 工程中得到广泛的应用
servlet (服务器小程序), 特点:
- 由服务器(Tomcat)调用和执行
- 是用 Java 语言编写的, 本质就是 Java 类
- 他是按照 Servlet 规范开发的
- 功能强大, 可以完成几乎所有网站功能

### 上手
开发一个 HelloServlet
1. 编写 HelloServlet 实现 Servlet 接口
2. 实现 service 方法, 处理请求, 并响应数据
3. 在 web.xml 中去配置 servlet 程序的访问地址

#### servlet 接口
继承 servlet 接口要实现5个方法
1. init 方法: 创建servlet 实例时会调用 init 方法
2. getServletConfig 方法: 获取servlet的配置
3. service 方法: 用来处理请求, tomcat 会将 http 请求封装成 request 对象
4. getServletInfo 方法: 
5. destroy 方法:

实现核心方法
```java
@Override  
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {  
    servletResponse.setContentType("text/html");  
    servletResponse.getWriter().println("<h1>Hello, Servlet!</h1>");  
}
```

#### 配置 servlet 提供路径映射
在 web.xml 中配置, 注册 servlet 并配置路径映射
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"  
         version="4.0">  
    <!--配置HelloServlet-->  
    <servlet>  
        <servlet-name>HelloServlet</servlet-name>  
        <!--反射生成实例需要使用-->  
        <servlet-class>com.fancier.servlet.study.HelloServlet</servlet-class>  
    </servlet>  
    <!--配置路径映射-->  
    <servlet-mapping>  
        <servlet-name>HelloServlet</servlet-name>  
        <url-pattern>/helloServlet</url-pattern>  
    </servlet-mapping>  
</web-app>
```

### 浏览器请求 servlet 流程

![[Pasted image 20241112131638.png]]

如果是第一次请求会查询 web.xml 根据 url 找到对应的 servlet 的 name, 根据 servlet 名字去 tomcat 维护的 hashMap 中找, 没找到就通过反射创建实例, 找到了就取出来, 然后调用 service 方法
### servlet 生命周期

1. 启动预编译加载
```xml
<servlet>  
    <servlet-name>HelloServlet</servlet-name>  
    <!--反射生成实例需要使用-->  
    <servlet-class>com.fancier.servlet.study.HelloServlet</se rvlet-class>  
    <!--预编译加载-->  
    <load-on-startup>1</load-on-startup>  
</servlet>
```

### GET 和 POST 请求的分发处理
将 ServletRequest 转成 HttpServletRequest 后获取请求类型, 然后再判断如何处理

### HttpServlet

实际开发使用 HttpSrevlet 比较多
![[Pasted image 20241112164839.png]]

基础的使用
1. 继承 HttpServlet
2. 重写 doGet 和 doPost 方法
3. 配置到 web.xml 文件中

### Idea 开发 Servlet
使用 Idea 便捷开发 Servlet

### servlet 注解方式
```java
@WebServlet(name = "OkServlet", urlPatterns = "/ok")
public class OkServlet extends HttpServlet {
	
}
```

原理, tomcat 对包进行扫描, 如果发现某个类被 @WebServlet 修饰了, 就读取注解里的值将其注册为 Servlet, 实例化后将其放入 HashMap 中

### WebServlet 注解参数说明
name 
urlPatterns
loadOnStartup 配置预编译加载, 值为权值

### url 匹配方式
优先级逐次递减
- 精确匹配: `"/ok"`
- 目录匹配: `"/ok/*"` 这样
- 拓展名匹配: `"*.action"` 这样 不能写`\` 不然会报错
- 任意匹配: `"/*"` 什么都能匹配
- `"/"`
### url 配置注意事项
如果配置 `"/"` `"/*"`会覆盖 tomcat 的 `DefaultServlet` , 当其他 urlPattern 都匹配不上时, 就会走这个 Servlet, 如果覆盖了那其他对静态资源的请求就会被拦截, 从而出错

`DefaultServlet` 在 tomcat 安装目录下 conf 目录下的 web.xml 中被配置

### ServletConfig
#### 介绍
1. 是为 Servlet 程序的配置信息的类
2. 由 Tomcat 管理
3. 在 Servlet 创建时, 就创建一个对应的 ServletConfig 对象

#### 作用
1. 获取 servlet-name 的值
2. 获取初始化参数 init-param
3. 获取 ServletContext 对象

#### 实验
在 init-param 中配置数据库连接的数据
```xml
<init-param>  
    <param-name>username</param-name>  
    <param-value>root</param-value>  
</init-param>  
<init-param>  
    <param-name>password</param-name>  
    <param-value>mingyue</param-value>  
</init-param>
```

获取 servletConfig 属性去获取数据库连接数据
```java
@Override  
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
    ServletConfig servletConfig = this.getServletConfig();  
    String username = servletConfig.getInitParameter("username");  
    String password = servletConfig.getInitParameter("password");  
    resp.setContentType("text/html");  
    resp.getWriter().printf("<h1>username:%s</h1>", username);  
    resp.getWriter().printf("<h1>password:%s</h1>", password);  
}
```

### ServletContext
需求: 存储 Servlet 之间的共享数据, 数据结构类似 Map

1. ServletContext 是一个接口
2. 在整个web工程只有一个实例
3. 在 web 工程启动时候创建, 在 web 工程停止时销毁

#### 作用
1. 获取 web.xml 中的上下文参数 context-param
2. 获取当前的工程路劲, 格式: /工程路径
3. 获取工程部署后在服务硬板的绝对路径
4. 像 Map 一样存取数据, 多个 Servlet 共享数据

#### 实验
自定义参数
```xml
<servlet-mapping>  
    <servlet-name>DBServlet</servlet-name>  
    <url-pattern>/db</url-pattern>  
</servlet-mapping>  
<context-param>  
    <param-name>k1</param-name>  
    <param-value>v1</param-value>  
</context-param>
```

获取数据
```java
ServletContext servletContext = getServletContext();  
String k1 = servletContext.getInitParameter("k1");  
servletContext.getContextPath();
```

获取工程路径
```java
servletContext.getContextPath();
```

获取硬盘绝对路径
```java
servletContext.getRealPath("/");
```

### HttpServletRequest
1. 代表客户都端的请求
2. 封装了 Http 请求头的所有信息
3. 通过这个对象的方法, 可以获取到客户信息

#### 常用方法
1. getRequestURI() 获取请求的资源路径
2. getRequestURL() 获 取 请 求 的 统 一 资 源 定 位 符 （ 绝 对 路 径 ）
3. getRemoteHost() 获取客户端的 主机, getRemoteAddr()
4. getHeader() 获取请求头
5. getParameter() 获取请求的参数
6. getParameterValues() 获取请求的参数（多个值的时候使用） , 比如 checkbox, 返回的数组
7. getMethod() 获取请求的方式 GET 或 POST
8. setAttribute(key, value); 设置域数据
9. getAttribute(key); 获取域数据
10. getRequestDispatcher() 获取请求转发对象, 请求转发的核心对象

#### 使用注意事项
解决请求参数乱码
```java
request.setCharacterEncoding("utf-8");
```

响应乱码 设置 contentType 

### 请求转发
1. 实现请求转发 ：个 请求转发指一个 web 资源收到客户端请求后 ， 通知服务器去调用另外个 一个 web 资源进行处理
2. HttpServletRequest 对象(也叫 Request 对象)提供了一个 getRequestDispatcher 方法，该方法返回一个 RequestDispatcher 对象，调用这个对象的 forward 方法可以实现请求转发
3. request 对象同时也是一个域对象，开发人员通过 request 对象在实现转发时，把数据通过 request 对象带给其它 web 资源处理
	-  setAttribute方法
	- getAttribute方法
	- removeAttribute方法
	- getAttributeNames方法

#### api
```java
// 获取分发器
RequestDispatcher requestDispatcher = request.getRequestDispatcher("/xxxx");
// 转发
requestDispatcher.forward(request, response);
```

#### 注意
不能转发到服务器外面的资源



## Http 协议

### 浏览器抓http包
![[Pasted image 20241112230427.png|600]]
























