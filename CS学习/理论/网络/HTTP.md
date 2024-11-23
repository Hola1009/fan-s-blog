
# 理解http
超文本就是指“含有指向其他资源链接”内容的文本。
http是属于应用层的协议 全称"[[#超文本]]传输协议"
tcp/ip协议 在网络种充当快递公司的存在 关注通信细节 
而http相当于 收到货之后的使用说明书 关注应用细节
![[Pasted image 20231207153502.png|500]]

## 理解http的工作流程
![](http工作流程图|500)
1. 发送请求
2. 服务器处理请求返回
---
# 应用场景
1. 浏览器
2. 手机app

---


# 学习重点
HTTP的报文格式

http协议 `一问一答` 的结构模型


## 查看http报文格式
也就是抓包
通过Fiddler 工具 查看
具体操作不做介绍了 
[fiddler的使用]([Fiddle 抓包小白一步带过超详细教程（含汉化）_fiddler汉化-CSDN博客](https://blog.csdn.net/weixin_42995152/article/details/130214628))

http协议是文本格式的, tcp/ip udp 是二进制格式的
![[Pasted image 20231207161604.png]]

---

# http 请求和响应格式
## http 请求和响应格式
### 1. 首行
http请求的第一行 包含3部分信息
1. GET 方法名
2. url 唯一资源定位符
3. 版本号
![[Pasted image 20231207162144.png]]
### 2. 请求头
```
一种键值对的数据结构
```
### 3. 空行
请求头结束标志

### 4. 正文
不是所有http请求都有

## http响应格式

### 1. 首行
1. 版本号
2. 状态码(向404这种就是状态码)
3. 状态描述
![[Pasted image 20231207162519.png|left]]

### 2. 响应头
```
也是种键值对的数据结构
```


### 3. 空行
结束标志

### 4. 正文
返回 html这类的格式 还有图片 音频之类的都在正文中
想要的数据就放在正文中


![|left](request协议格式.md)

> [!question] 为什么要有空行

1. 因为请求报头中的键值对数量未知 空行相当于一个报头结束标志
2. http 在传输层依赖tcp传输 如果没有空行会出现粘包问题

# http请求
## 认识url
### url的基本格式
url(统一资源定位符) 在互联网上的每个文件都有一个url 这就是这个文件的身份证
url中包含着文件的位置信息 和 浏览器需要如何操作该文件的信息

![](Url|600)

> [!example]

类比成生活中的例子
```
http://美食街:胡记烤面筋/烤面筋/大串?辣=微辣&打包=true
```

> [!tip] urlencode

一言以蔽之: `中文在query string 中出现要进行转码`
```
张三
%E5%BC%A0%E4%B8%89
```

它的逆过程是==urldecode==
## 认识http请求中的方法
![[Pasted image 20231207220212.png]]

> [!tip] 常用的是 GET 和 POST

### get方法
文服务端要资源的方法:
用于获取服务其中的指定资源 如何指定 --> 通过[[Url]]指定
所以在浏览器中输入url 回车后就除法get方法

```
GET https://www.bilibili.com/ HTTP/1.1
```
#### get请求的特点
1. 在请求的首位
2. url的query string 可为空
3. 报头有键值对
4. `body部分为空`

### post方法
给服务端数据的方法: 把用户数据提交给服务端的方法
> 通过 HTML 中的 form 标签可以构造 POST 请求, 或者使用 JavaScript 的 ajax 也可以构造 POST 请求.

```http
POST /user HTTP/1.1                       // 请求行
Host: www.user.com
Content-Type: application/x-www-form-urlencoded
Connection: Keep-Alive
User-agent: Mozilla/5.0.                  // 以上是请求头
                                          // 空行分割header和请求内容 
name=world                                // 请求体(可选，如get请求时可选)

```

#### post请求的特点
1. 也是在请求的首位
2. 有body部分 body的数据类型由 header 中的 `Content-Type` 指定 长度由 `Content-Length` 指定
3. query string 一般为空
4. header有键值对


> [!question] get 和 post 的区别

1. get用于申请数据post用于提交数据
2. get body一般部分为空 数据通过query string传递, post body一般部分不为空, 数据通过body传递
3. get幂等 post不幂等
4. get 可缓存 post不可



## 认识请求报头header

header为键值对结构
每个键值对占一行. 键和值之间使用分号分割.

1. Host: 表示服务器主机的地址和端口.
2. Content-Length: 表示 body 中的数据长度.
3. Content-Type: 表示请求的 body 中的数据格式
4. User-Agent (简称 UA) 表示浏览器/操作系统的属性
```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Chrome/91.0.4472.77 Safari/537.36
```
其中 Windows NT 10.0; Win64; x64 表示操作系统信息
AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36 表示浏览器
信息.
5. Referer: 表示这个页面是从哪个页面跳转过来的. 

## 理解登录过程

![[登录过程|500]]


# HTTP响应详解

## 认识状态码
状态码表示访问一个页面的结果
### 常见的状态码
1. 200 访问成功
2. 404 没有找到资源
3. 403 访问被拒绝(比如: 有些资源只允许登录了才能访问)
4. 405 服务器不支持方法
5. 500 服务器内部错误
6. 504 服务器超时
7. 302 重定向
8. 301 永久重定向

---

# http正文
正文具体格式取决于 `Content-Type`

## 通过form 发送GET请求
- form的action属性对应http请求的url
- form的method属性对应http请求的方法
- input的name属性定对应query string的key
- input的内容对应query string的key

## 通过form 发送POST请求
> [!info] 与提交GET的区别

- GET中通过query string 提交的数据 被放在body中了

# 通过ajax构造http请求
出了通过浏览器地址栏 和 form 可以构造http请求, 还可以通过ajax
> [!info] ajax 全称 Asynchronous Javascript And XML

通过名字可以看出ajax是通过javaScript来发送http请求的
> [!tip] 不需要刷新页面/页面跳转 就能传输数据

而通过输入url或form的形式要刷新

### 引入ajax后浏览器与服务器的交互过程
![[浏览器和服务器交互过程(引入 ajax 后)|500]]

### 发送POST请求
对于POST请求, 需要设置body的内容
1. 先使用setRequestHeader 设置 Content_Type
2. 再通过send的参数设置body内容

#### 发送 application/x-www-form-urlencoded 数据 (数据格式同 form 的 post)

```JavaScript
// 1. 创建 XMLHttpRequest 对象
let httpRequest = new XMLHttpRequest();
// 2. 默认异步处理响应. 需要挂在处理响应的回调函数.
httpRequest.onreadystatechange = function () {
  // readState 表示当前的状态.
  // 0: 请求未初始化
  // 1: 服务器连接已建立
  // 2: 请求已接收
  // 3: 请求处理中
  // 4: 请求已完成，且响应已就绪
  if (httpRequest.readyState == 4) {
    // status 属性获取 HTTP 响应状态码
    console.log(httpRequest.status);
    // responseText 属性获取 HTTP 响应 body
    console.log(httpRequest.responseText);
 }
}
// 3. 调用 open 方法设置要访问的 url
httpRequest.open('POST', 'http://42.192.83.143:8080/AjaxMockServer/info');
// 4. 调用 setRequestHeader 设置请求头
httpRequest.setRequestHeader('Content-Type', 'application/x-www-form-
urlencoded');
// 5. 调用 send 方法发送 http 请求
httpRequest.send('name=zhangsan&age=18');
```

#### 发送 application/json 数据
```JavaScript
// 4. 调用 setRequestHeader 设置请求头
httpRequest.setRequestHeader('Content-Type', 'application/json');
// 5. 调用 send 方法发送 http 请求
httpRequest.send(JSON.stringify({
  name: 'zhangsan',
  age: 18
}));
```


























###### 超文本
超文本就是指“含有指向其他资源链接”内容的文本。
