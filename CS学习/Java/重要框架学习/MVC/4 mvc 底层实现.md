## 开发 DispatcherServlet
先初始化一个 DispatcherServlet 类, 暂且不需要填充逻辑, 然后在 web.xml 中去配置下
```xml
<servlet>  
  <servlet-name>DispatcherServlet</servlet-name>  
  <servlet-class>com.fancier.mvc.servlet.DispatcherServlet</servlet-class>  
  <init-param>    <param-name>contextConfigLocation</param-name>  
    <param-value>classpath:fancierMVC.xml</param-value>  
  </init-param>  
  <load-on-startup>1</load-on-startup>  
</servlet>  
<servlet-mapping>  
  <servlet-name>DispatcherServlet</servlet-name>  
  <url-pattern>/</url-pattern>  
</servlet-mapping>
```

## 完成客户端/浏览器可以请求控制层
### 1. 创建自己的 Controller 和自定义注解
```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface Controller {  
    String value() default "";  
}
```

### 2. 配置 fancierMvc.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans>  
    <component-scan basepackge="com.fancier.mvc.controller"/>  
</beans>
```

### 3. 编写 XMLParser 工具类, 可以解析 fancierMvc.xml
```java
public class XMLParser {  
    public static String getBeanPackage(String xmlFile) {  
        SAXReader saxReader = new SAXReader();  
        InputStream inputStream = XMLParser.class.getClassLoader()
	        .getResourceAsStream(xmlFile);  
        try {  
            Document document = saxReader.read(inputStream);  
            Element rootElement = document.getRootElement();  
            Element componentScanElement = rootElement.element("component-scan");  
  
            return componentScanElement.attribute("base-package").getText();  
        } catch (DocumentException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```

> [!tip]
> - classPath 而不是 src，这是因为 web 项目运行时, IDE 编译器会把 src 下的一些资源文件移至 `WEB-INF/classes`
> - getResourceAsStream() 是获取资源的输入流。**类加载器默认是从classPath路径加载资源。**
> - 使用 Class.getClassLoader.getResourceAsStream() 加载资源时，是从 classPath 路径下进行加载
> ![[Pasted image 20241121083254.png|300]]

### 4. 完善 WebApplicationContext, 充当 Spring
- 通过 XMLParser 拿到扫描路径, 然后以递归的方式遍历文件夹, 如果是文件将其类路径放入集合中
```java
public void scanPackage(String pack) {  
    URL url = this.getClass().getClassLoader()  
            .getResource("/" + pack.replaceAll("\\.", "/"));  
  
    String path = Objects.requireNonNull(url).getFile();  
    File dir = new File(path);  
  
    // 遍历目录  
    for (File file : Objects.requireNonNull(dir.listFiles())) {  
        if(file.isDirectory()) {  
            scanPackage(pack + "."  + file.getName());  
        } else {  
            classFullPathList  
                    .add(pack + "." + file.getName().replace(".class", ""));  
        }  
    }  
}
```

- 在 DispatcherServlet 初始化时也对容器进行初始化, 进行包扫描

### 5. 容器 - 实例化对象到容器中
- 将扫描到的类(集合中存着的), 在满足条件的情况下 (即有相应的注解 @Controller), 反射注入到 ioc 容器
> [!question] 怎么判断类被 @Controller 注解修饰
> ```java
> clazz.isAnnotationPresent(Controller.class)
> >> true
> ```


### 6. 完成请求 URL 和控制器方法的映射关系
遍历之前的加入 ioc 容器的 bean 通过反射获取其中被 @RequestMapping 修饰的方法, 然后提取出 url 信息, 并和 bean 和方法, 一起封装进一个 UrlHandler 对象, 再将 UrlHandler 对象放入集合中
```java
private void initHandlerMapping() {  
    if (webApplicationContext.ioc.isEmpty()) {  
        return;  
    }  
    for (Map.Entry<String, Object> entry : webApplicationContext.ioc.entrySet()) {  
  
        Object bean = entry.getValue();  
        // 通过反射获取到 methods        
        Class<?> clazz = bean.getClass();  
        
        if (!clazz.isAnnotationPresent(Controller.class)) {  
		    return;  
		}
		
        Method[] methods = clazz.getMethods();  
  
        for (Method method : methods) {  
            if (method.isAnnotationPresent(RequestMapping.class)) {  
                // 获取 url                
                RequestMapping annotation = method.getAnnotation(RequestMapping.class);  
                String url = annotation.value();  
                // 存入  
                handlerList.add(new UrlHandler(url, bean, method));  
            }  
        }  
    }  
}
```

### 7. 完成 DispatcherServlet 请求分发到控制器
```java
private void executeDispatcher(HttpServletRequest request, HttpServletResponse response) {  
    UrlHandler urlHandler = getUrlHandler(request);  
  
        // 请求资源不存在  
        try {  
            if (Objects.isNull(urlHandler)) {  
                response.getWriter().println("<h1>404 NOT FOUND</h1>");  
                return;  
            }  
  
            // 执行目标方法  
            Object controller = urlHandler.getController();  
            Method method = urlHandler.getMethod();  
            method.invoke(controller, request, response);  
        } catch (IOException | IllegalAccessException | InvocationTargetException e) {  
            throw new RuntimeException(e);  
        }  
}
```



> [!question] request.getRequestURL(); 和 request.getRequestURI(); 的 区别
> 1. 返回一个完整的请求 URL，包括协议、主机名、端口号，以及请求的路径和查询字符串。
> 2. 仅包括请求的路径及任何服务器应用程序特定的部分。

## 从 web.xml 动态获取 fancierMVC.xml
DispatcherServlet 在创建并初始化 WebApplicationContext 时, 动态从 web.xml 中获取到配置文件位置

## 实现 @Service 注解
被这个注解标识的类, 应该被 ioc 注入到 controller 中

## 实现 Spring 容器的自动装配

## 实现 @RequestParam
### 填充参数表策略
通过反射拿到参数表中的参数封装成 Parameter 数组, 然后进行遍历
- 如果匹配到参数类型是 HttpServletRequest 或是 HttpServletResponse 就直接拿现成的 request 和 response 对象经行赋值
- 如果没匹配上就看一下是否被 @RequestParam 修饰, 是的话就从 request 中获取同名参数的值经行赋值
- 还匹配上就用参数名到 request 查询进行匹配

> [!warning] java 反射中 Parameter 的 getName 后得到 arg0 的问题
> 在 maven 编译插件的配置中添加这样一个配置
> ```xml
> \<compilerArgs>  
>     <arg>-parameters</arg>  
> </compilerArgs>
> ```

## 实现视图解析
拿到请求对其进行处理, 如果返回结果是字符串的化, 就根据字符串前缀判断是否需要请求转发或是重定向
> [!question] 解决请求参数中文乱码问题
> ```java
> request.setCharacterEncoding("UTF-8");
> ```

## 实现 @RequestBody
将控制层返回的引用数据类型转成 json 字符串返回
