## SpringMVC 的请求处理
### 请求数据接收

#### 对于 @GetRequest
它是用来读取 url 上的参数的, 如果参数表上的入参名字与传过来的相同, 就不用加 @RequestParam 注解, 如果不一样, 则需要将这个注解里面的 value 赋值为传过来的参数名

如果传过来多个同名参数可以使用 基本数据类型的数组接收, 如果使用了List<> 这种集合接口接收 需要用 @RequestParam 声明一下

@RequestParam 的另外几个属性, required 是否为必须传的参数, defaultValue 如果没传值过来用什么参数代替

##### 提交实体数据
底层是通过反射的方式调用set方法设置值


#### 对于 @PostRequest
接收请求体中的数据 需要再 接收参数前加上 `@RequestBody `注解

#### 将数据转成实体
导入坐标
```xml
<dependency>  
    <groupId>com.fasterxml.jackson.core</groupId>  
    <artifactId>iackson-databind</artifactId>  
    <version>2.9.0</version>  
</dependency>
```

人为转化
![[Pasted image 20240503192921.png]]

也可以自己配转化器, 会自动解析json格式字符串

![[Pasted image 20240503193344.png]]

### Restful 风格的数据提交
什么是Rest风格?
Rest(Representational State Transfer)表象化状态转变(表述性状态转变)，在2000年被提出，基于HTTP、URI, xml、JSON等标准和协议，支持轻量级、跨平台、跨语言的架构设计。是Web服务的一种新网络应用程序的设计风格和开发方式。

- 用URL表示某个模块资源, 资源名称为名词
  ![[Pasted image 20240503193937.png]]
- 用 HTTP 响应状态码表示结果, 国内常用的响应包括三部分: 状态码, 状态信息, 响应数据
  ![[Pasted image 20240503194131.png|left|300]]
读取url参数
![[Pasted image 20240503194529.png]]

接收文件上传的数据, 文件上传的表单需要一定的要求
- 表单的提交方式必须是POST
- 表单的 enctype 属性必须是 multipart/form-data
- 文件上传需要有name属性

获取请求头
![[Pasted image 20240503201916.png]]

![[Pasted image 20240503201946.png]]



如果要访问静态资源需要配置映射地址
![[Pasted image 20240503204114.png]]

或者直接注册一个静态资源处理器

![[Pasted image 20240503204255.png|left|300]]

## 注解驱动 <mvc:annotation-driven> 标签


## SpringMVC 的响应处理

### 传统同步业务数据响应
- 传统同步方式: 准备好模型数据, 在跳转到执行页面进行展示, 此方法使用越来越少了
- 前后端分离异步方式:前端使用Ajax技术+Restful风格与服务端进行Json格式为主的数据交互，目前市场上几乎都是此种方式了

#### 传统同步业务数据响应
传统同步业务在数据响应时，SpringMVC又涉及如下四种形式
- 请求资源转发;
- 请求资源重定向;
- 响应模型数据;
- 直接回写数据给客户端;

![[Pasted image 20240503212606.png]]


告诉mvc 以响应体的方式返回数据
![[Pasted image 20240503213700.png]]


前后端分离异步业务数据响应
其实此处的回写数据，跟上面回写数据给客户端的语法方式一样，只不过有如下一些区别:
- 同步方式回写数据，是将数据响应给浏览器进行页面展示的，而异步方式回写数据一般是回写给Ajax引擎的，即谁访问服务器端，服务器端就将数据响应给谁
- 同步方式回写的数据，一般就是一些无特定格式的字符串，而异步方式回写的数据大多是Json格式字符串


`@RestController` = `@Controller` + `@RequestBody`



@RequestBody 指定返回的数据为json格式, 并将原来的要返回的实体数据给转成json格式

## 静态资源请求
原先的javaweb中, 会有一个 默认的servlet来负责匹配静态资源, 
![[Pasted image 20240513152049.png|left|400]]
默认servlet的映射路径为 / 

但是现在的前端控制器的servlet 把这个映射路径给抢占了
![[Pasted image 20240513152233.png|left|700]]



为了让其有访问静态资源的能力需要重新激活 默认的servlet
就是把映射路径给写得清楚些
![[Pasted image 20240513151741.png|left|400]]


其他方式

![[Pasted image 20240513152336.png]]
















