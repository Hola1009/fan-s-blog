# 加载外部 properties 文件
Spring 整合其他组件时就不像 MyBatis 这么简单了，例如 Dubbo 框架在于 Spring 进行整合时，要使用 Dubbo 提供的
为了降低我们此处的学习成本，不在引入 Dubbo 第三方框架了，以 Spring 的 context 命名空间去进行讲解，该方式
也是命名空间扩展方式。

需求:加载外部 properties 文件，将键值对存储在Spring容器中

```java
jdbc.driver=com.mysql.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:3306/mybatis_test  
jdbc.username=root  
jdbc.password=mingyue
```

在 spring xml 文件中要导入 context 的命名空间 (就在这里面自己找去吧)

```java
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:mvc="http://www.springframework.org/schema/mvc"  
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"  
       xmlns:haohao="http://www.itheima.com/haohao"  
       xsi:schemaLocation="  
       http://www.springframework.org/schema/beans       http://www.springframework.org/schema/beans/spring-beans.xsd       http://www.springframework.org/schema/context       http://www.springframework.org/schema/context/spring-context.xsd       http://www.springframework.org/schema/mvc       http://www.springframework.org/schema/mvc/spring-mvc.xsd       http://www.itheima.com/haohao       http://www.itheima.com/haohao/haohao-annotation.xsd       http://dubbo.apache.org/schema/dubbo       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
```

在加载 和 苏区 加载的外部 properties 文件

```xml
<!--加载properties文件-->  
<context:property-placeholder location="classpath:jdbc.properties"/>  
  
<!--配置数据源信息-->  
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">  
    <property name="driverClassName" value="${jdbc.driver}"></property>  
    <property name="url" value="${jdbc.url}"></property>  
    <property name="username" value="${jdbc.username}"></property>  
    <property name="password" value="${jdbc.password}"></property>  
</bean>
```


## 测试一下

测试了能正常执行

```java
public class ApplicationContextTest {  
    public static void main(String[] args) throws Exception {  
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        UserMapper userMapper = (UserMapper)context.getBean("userMapper");  
        for (User user : userMapper.findAll()) {  
            System.out.println(user);  
        }  
    }  
}
```

代码在这, 自己试试去吧
# 自定义命名空间

## 解析原理

这块 挺绕的 方法的调用栈很深, 可得好好花功夫, 不贫了, 开始

从这开始 :
```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

点进去

```java
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {  
    this(new String[]{configLocation}, true, (ApplicationContext)null);  
}
```

再点

```java
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent) throws BeansException {  
    super(parent);  
    this.setConfigLocations(configLocations);  
    if (refresh) {  
        this.refresh();  
    }  
}
```

点 `refresh()` 方法

```java
public void refresh() throws BeansException, IllegalStateException {  
    synchronized(this.startupShutdownMonitor) {  
        ... ...  
        // 填充 BDMap 的过程
		ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
		... ...
		
        try {  
            ... ...   
            //实例化 Bean 的过程
            this.finishBeanFactoryInitialization(beanFactory);  
            ... ...
        } ... ...

		... ...
  
    }  
}
```

其他的不看 点进 `this.obtainFreshBeanFactory()` 这个方法

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {  
    this.refreshBeanFactory();  
    return this.getBeanFactory();  
}
```

点 `this.refreshBeanFactory()`

```java
protected abstract void refreshBeanFactory() throws BeansException, IllegalStateException;
```

选这个

![[Pasted image 20240420164821.png]]


点进 里面的这个方法

```java
this.loadBeanDefinitions(beanFactory);
```

之后进去的方法都点 这个 `this.loadBeanDefinitions();`, 如果 进到接口里面就选择 %Xml% 的实现类



```java
doLoadBeanDefinitions(inputSource, encodedResource.getResource());
```

最后在 `XmlBeanDefinitionReader` 中找到了这个 方法, 点进去

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)  
       throws BeanDefinitionStoreException {  
  
    try {  
       ... ...
       // 看这里 看这里
       int count = registerBeanDefinitions(doc, resource);  
       ... ...   
    }  
    ... ...
```

看到了 `registerBeanDefinitions()` 再点进去

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {  
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();  
    int countBefore = getRegistry().getBeanDefinitionCount();  
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));  
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

进 `documentReader.registerBeanDefinitions(doc, createReaderContext(resource));` 这个方法
再进这个方法

```java
doRegisterBeanDefinitions(doc.getDocumentElement());
```

到关键点了

```java
parseBeanDefinitions(root, this.delegate);
```

进去吧

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {  
    if (delegate.isDefaultNamespace(root)) {  
       NodeList nl = root.getChildNodes();  
       for (int i = 0; i < nl.getLength(); i++) {  
          Node node = nl.item(i);  
          if (node instanceof Element) {  
             Element ele = (Element) node;  
             if (delegate.isDefaultNamespace(ele)) { //判断是否为默认命名空间 
                parseDefaultElement(ele, delegate);  
             }  
             else {  
                delegate.parseCustomElement(ele); // 解析自定义元素
             }  
          }  
       }  
    }  
    else {  
       delegate.parseCustomElement(root);  
    }  
}
```

打上断点操作一番

![[Pasted image 20240420170417.png|left|350]]

看到了我们要用 加载的外部文件的那个标签了

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>  
```

再看看 `parseCustomElement()` 方法
```java
@Nullable  
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {  
    String namespaceUri = getNamespaceURI(ele);  
    if (namespaceUri == null) {  
       return null;  
    }  
    // 关键一步
    NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri); 
     
    if (handler == null) {  
       error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);  
       return null;  
    }  
    return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));  
}
```

发现根据标签 咱们获得到了 命名空间 look 
![[Pasted image 20240420171148.png|left|400]]

获得命名空间后 会根据 这个 `resolve()` 方法获取命名空间对应的 命名空间处理器; 每个命名空间都有自己对应的处理器, 不多说点进去看看

```java
@Override  
@Nullable  
public NamespaceHandler resolve(String namespaceUri) {  
    Map<String, Object> handlerMappings = getHandlerMappings();  
    Object handlerOrClassName = handlerMappings.get(namespaceUri);  
    if (handlerOrClassName == null) {  
       return null;  
    }  
    else if (handlerOrClassName instanceof NamespaceHandler) {  
       return (NamespaceHandler) handlerOrClassName;  
    }  
    else {  
       String className = (String) handlerOrClassName;  
       try {  
          Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);  
          if (!NamespaceHandler.class.isAssignableFrom(handlerClass)) {  
             throw new FatalBeanException("Class [" + className + "] for namespace [" + namespaceUri +  
                   "] does not implement the [" + NamespaceHandler.class.getName() + "] interface");  
          }  
          NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);  
          namespaceHandler.init();  
          handlerMappings.put(namespaceUri, namespaceHandler);  
          return namespaceHandler;  
       }  
       catch (ClassNotFoundException ex) {  
          throw new FatalBeanException("Could not find NamespaceHandler class [" + className +  
                "] for namespace [" + namespaceUri + "]", ex);  
       }  
       catch (LinkageError err) {  
          throw new FatalBeanException("Unresolvable class definition for NamespaceHandler class [" +  
                className + "] for namespace [" + namespaceUri + "]", err);  
       }  
    }  
}
```

handlerMappings 就是用来存 对应关系的
![[Pasted image 20240420171731.png|left|500]]

k 命名空间名, v 处理器的全限定类名

map的数据都从哪里来呢? 点进 `getHandlerMappings()` 的那个类瞅瞅

`DefaultNamespaceHandlerResolver` 在创建时 干了这么件事, 调用了下面这构造
```java
public static final String DEFAULT_HANDLER_MAPPINGS_LOCATION = "META-INF/spring.handlers";

public DefaultNamespaceHandlerResolver(@Nullable ClassLoader classLoader) {  
    this(classLoader, DEFAULT_HANDLER_MAPPINGS_LOCATION);  
}
```

找到 context 的那个包 发现还真有这么个文件
![[Pasted image 20240420172536.png|left|360]]

```properties
http\://www.springframework.org/schema/context=org.springframework.context.config.ContextNamespaceHandler  
http\://www.springframework.org/schema/jee=org.springframework.ejb.config.JeeNamespaceHandler  
http\://www.springframework.org/schema/lang=org.springframework.scripting.config.LangNamespaceHandler  
http\://www.springframework.org/schema/task=org.springframework.scheduling.config.TaskNamespaceHandler  
http\://www.springframework.org/schema/cache=org.springframework.cache.config.CacheNamespaceHandler
```

这里头啊 就保存这 map 中存的个那么些对应关系, map 中的数据就就从这里来

好回过头来

```java
NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
```
到这步就 获得了 命名空间处理器了, 然后再执行它的初始化方法, 初始化方法都干了啥呢??? 注册解析器, 不同标签使用不同的解析器, 

```java
public class ContextNamespaceHandler extends NamespaceHandlerSupport {  
    public ContextNamespaceHandler() {  
    }  
  
    public void init() {  
        this.registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());  
        this.registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());  
        this.registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());  
        this.registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());  
        this.registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());  
        this.registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());  
        this.registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());  
        this.registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());  
    }  
}
```

这个类父类中的 parse 方法 实际调用的是具体标签的解析器

```java
@Override  
@Nullable  
public BeanDefinition parse(Element element, ParserContext parserContext) {  
    BeanDefinitionParser parser = findParserForElement(element, parserContext);  
    return (parser != null ? parser.parse(element, parserContext) : null);  
}
```

回到这个方法,  所以, 这个 `handler.parse` 调用的就是标签的 parse 
```java
@Nullable  
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {  
    String namespaceUri = getNamespaceURI(ele);  
    if (namespaceUri == null) {  
       return null;  
    }  
    NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);  
    if (handler == null) {  
       error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);  
       return null;  
    }  
    return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));  
}
```


# 总结
如果 第三方 想要 用自定义命名空间的方式 和 spring 集成, 就要再 第三方的包里面定义一个 MATE-INF 文件夹 里面得有一个spring.handlers 文件 用来保存 命名空间 和 命名空间处理器 的对应关系.  解析器中有 init 方法, 用于给每个标签 配置一个解析器, 当要解析 xml 文件中的标签的时候就调用 命名空间处理器  的 parse 方法 它里面会调用具体标签的解析器的 parse 方法
 

## 梳理一下外部命名空间标签的执行流程
- 将自定义标签的约束 与 物理约束文件与网络约束名称的约束 以键值对形式存储到一个spring.schemas文件里该文件存储在类加载路径的 META-INF里，Spring会自动加载到;
- 将自定义命名`空间的名称 与 自定义命名空间的处理器映射关系` 以键值对形式存在到一个叫 `spring.handlers` 文件里，该文件存储在类加载路径的 META-INF里，Spring会自动加载到;
- 准备好NamespaceHandler，如果命名空间只有一个标签，那么直接在parse方法中进行解析即可，一般解析结果就是注册该标签对应的BeanDefinition。如果命名空间里有多个标签，那么可以在init方法中为每个标签都注册一个BeanDefinitionParser，在执行NamespaceHandler的parse方法时在分流给不同的BeanDefinitionParser进行解析(重写doParse方法即可)。

在扫描 xml 文件的时候, 扫描到标签如果发现它不是默认的标签, 就找到它的命名空间名, 然后根据命名空间名再找到 与之对应的命名空间处理器, 然后再调用它的解析方法,  这个解析方法底层调的是标签的解析方法
处理器在初始化的时候会为每个标签分配一个解析器, 所以在调用它解析方法的时候底层调用的是 标签解析器的方法.  

