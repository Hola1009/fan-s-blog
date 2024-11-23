> [!ruestion] 什么是会话
Web 会话（Web Session）是指在客户端与服务器之间持续的一段交互过程。在这段期间，*服务器可以识别并存储关于用户的信息，以增强用户体验*

## cookie
服务器返回的 cookie 数据会被浏览器保存, 当浏览器再次访问时会带上 cookie
1. Cookie 是服务器在客户端保存用户的信息, 比如登录名, 浏览历史, 就可以用 cookie 
2. Cookie 信息就像小甜饼, 数据量不大, **服务器需要时可以从请求头读取**

> [!question] 服务器需要时可以从请求头读取?
> 浏览器会将和这个网站相关的 Cookie 封装进请求头

> [!example] 一个简单的使用
java 服务器设置 cookie 数据, 下面这段代码会在响应头生成一个 `Set-Cookie: k1=v1` 的响应字段 
> ```java
> Cookie cookie = new Cookie("k1", "v1");
> response.addCookie(cookie);
> ```
> 浏览器接受响应后, 会解析 `Set-Cookie` 字段, 保存在本地

### 用途
1. 保存上次登录时间
2. 保存用户名
3. 网站的个性化
4. ...

### api
1. Cookie(String key, String value) : 构造器
2. setMaxAge(): 设置有效时间
3. response.addCookie(): 添加 cookie 到响应头
4. request.getCookies(): 从请求头获取 cookie

### JsessionId 
用来区分不同的客户端请求, 用来唯一标识会话

### 修改 cookie
添加新的 cookie 到响应头, 浏览器会覆盖已存在的同名 cookie
```java
Cookie cookie = new Cookie("k", "v");
response.addCookie(cookie):
```

### cookie 的生命周期
值如何管理cookie何时销毁, 过期后, 浏览器将不再携带这个 cookie, 单 cookie 还储存再浏览器
#### 核心方法
```java
setMaxAge()
```
- 正数: 指定秒数后过期
- 负数: 标识浏览器关闭, Cookie 就会被删除
- 0: 表示马上删除

> [!tip]
> - 在服务端让 cookie 失效, setMaxAge(0) 后再写回响应头, 浏览器接收到响应后会直接删除
> - 没有设置失效时间, cookie 默认失效时间是 -1


### cookie 的有效路径
cookie 的 path 属性可以有效过滤那些 Cookie 可以发送给服务器, 哪些不发. path 属性是通过请求地址来进行过滤的

#### 规则
```java
cookie1.setPath = / 工程路径
cookie2.setPath = / 工程路径/aaa
请求地址: http://ip: 端口/ 工程路径/ 资源
cookie1 会发给服务器
cookie2 不会发给服务器
请求地址: http://ip: 端口/ 工程路径/aaa/ 资源
cookie1 会发给服务器
cookie2 会发给服务器
```

> [!summary]
> 目录层次浅的可以访问目录层次深的
> 目录层次深的不可以访问目录层级浅的
> 

### cookie 注意事项
1. 一个 cookie 标识一种信息
2. 一个 web 站点可以给一个浏览器发送多个 cookie, 一个浏览器可以存储多个站点的 cookie
3. cookie 的总数据量没有限制, 但是每个域名的 cookie 数量和大小是有限制的
4. 删除 cookie 时, path 必须一致, 否则不会删除 (因为删除 cookie 是创建新 cookie 覆盖来完成), 所以要为新cookie 设置有效路径)
5. 解决中文乱码问题: 使用 URLEncoder 编码器将中文转成 url 编码

## session
1. 浏览器访问网站时并操作 session 时, 服务端会为其分配一个 session 对象
2. session 对象也可以看做是一个容器, 默认存储时间为 30 min, 可以在 tomcat conf 目录下 web.xml 文件内修改
```xml
<session-config>
  <session-timeout>30</session-timeout>
</session-config>
```

### session 用途
1. 网上商城中的购物车
2. 保存登录用户信息
3. 将数据放入到 Session 中, 供用户在访问不同页面时, 实现跨页面访问数据
4. 防止用户非法登录到某个页面
5. ...

### session 存储结构
k-v 结构, 可以将其看作时 HashMap, key 是 String , value 是 Object

### api
- getId() : 返回 JsessioinId
- setAttribute(String name, Object value) : 存放数据, 名字相同则覆盖
- getAttribute(String name) : 获取数据
- request.getSession() : 创建和获取session
- removeAttribute() : 删除某个属性
- isNew() : 判断是不是刚创建的 session

> [!warning] session 不需要手动创建
> 通过 HttpRequest 来获取
> ```java
> request.getSession();
> ```

### session 底层实现机制
当Tomcat服务器启动时，StandardManager会创建一个 SessionContext 对象，用于存储所有的session。每个session都有一个唯一的ID，这个ID在创建session时生成，并通过cookie或URL重写的方式传递给客户端。客户端在后续的请求中会携带这个ID，Tomcat根据这个ID来查找对应的session对象。

#### getSession() 方法
![[Pasted image 20241116182241.png|400]]
> [!tip] 
> 如果一次会话中创建了 session , tomcat 会将在响应头的 `set-cookie` 字段, 添加 `JsessionId` 属性

### session 的生命周期
session 指的是两次会话之间的最大间隔时间

#### api
1. setMaxInactiveInterval(int interval) 设置 Session 的超时时间 (以秒为单位), 超过指定的时长, Session 就会被销毁
	- 值为正数时, 设定 Session 的超时时长
	- 负数表示永不超时
2. getMaxInactiveInterval() 获取 Session 的超时时间
3. invalidate() 让当前 Session 会话立即无效
4. 如果没有设置默认为30min































































