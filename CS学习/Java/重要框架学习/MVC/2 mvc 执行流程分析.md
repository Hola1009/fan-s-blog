进到前端控制器, 调用处理器映射器拿到处理器执行链后找到处理器适配器, 再调用Handler, 得到返回结果后通过视图解析进行渲染, 然后返回响应
![[Pasted image 20241119081227.png]]

## REST
- 即 Representational State Transfer。(资源)表现层状态转化。是目前流行的请求方式。它结构清晰, 很多网站采用
- HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源。

### REST 核心过滤器
1.  当前的浏览器只支持post/get 请求，因此为了得到 put/delete 的请求方式需要使用 Spring 提供的 HiddenHttpMethodFilter 过滤器进行转换.
2. HiddenHttpMethodFilter：浏览器 form 表单只支持 GET 与 POST 请求，而 DELETE、PUT等 method 并不支持，Spring 添加了一个过滤器，可以将这些请求转换为标准的 http 方法，使得支持 GET、POST、PUT 与 DELETE 请求
3.  HiddenHttpMethodFilter 能对 post 请求方式进行转换，因此我们需要特别的注意这一点
4.  这个过滤器需要在 web.xml 中配置

### 应用
1. 在 web.xml 中配置 HiddenHttpMethodFilter
```xml
<filter>  
    <filter-name>hiddenHttpMethodFilter</filter-name>  
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>  
</filter>  
<filter-mapping>  
    <filter-name>hiddenHttpMethodFilter</filter-name>  
    <!-- 表示所有请求都经过 hiddenHttpMethodFilter -->    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

2. 修改 springDispatcherServlet-servlet.xml
```xml
<!-- 加入两个常规配置 -->  
<!-- 能支持 SpringMVC 高级功能，比如 JSR303 校验，映射动态请求 -->  
<mvc:annotation-driven/>  
<!-- 将 SpringMVC 不能处理的请求交给 Tomcat, 比如请求 css,js 等 -->  
<mvc:default-servlet-handler/>
```

3. 编写页面, 对 form 表单添加一个额外的请求类型参数
```html
<form action="" method="post">
<form action="" method="get">
```

> [!tip] 将 `<a>` get 请求转成 springmvc 可以识别的 delete 请求
> 未 目标标签绑定一个点击事件, 阻止其默认行为, 在点击方法里用自己的逻辑发送请求

### HiddenHttpMethodFilter 底层机制
核心代码如下, 先判断请求方式是否为 post 如果是判断 `_mathod` 参数的值 是否为 DELETE PUT PATCH 中的一个, 是就进行特殊操作 
```java
if ("POST".equals(request.getMethod()) && request.getAttribute("javax.servlet.error.exception") == null) {  
    String paramValue = request.getParameter(this.methodParam);  
    if (StringUtils.hasLength(paramValue)) {  
        String method = paramValue.toUpperCase(Locale.ENGLISH);  
        if (ALLOWED_METHODS.contains(method)) {  
            requestToUse = new HttpMethodRequestWrapper(request, method);  
        }  
    }  
}
filterChain.doFilter((ServletRequest)requestToUse, response);
```

### Rest-Reirect
返回数据这么写: `"redirect:/user/success"` (在后端解析)


### 获取 http 请求头信息
通过 @RequestHeader 注解来获取, value="请求头参数名"


### 获取 javaBean 对象
- 直接将前端传来的数据封装成 pojo
- 如果提交的数据的参数名和对象的字段属性名相同, 将自动封装
如果需要封装的 pojo 里由对象成员变量, 则前端提交时参数名可以是这样 `pet.id` 

### servlet api
- 直接传参 HttpServletRequest req, HttpServletResponse resp;
- 除了上面两个, 其他如 HttpSession, java.security.Principal, InputStream, OutputStream, Reader, Writer, Session 都可以直接获取


















