# 自定义命名空间的实现
步骤分析:
1. 确定命名空间名称、schema虚拟路径、标签名称;
2. 编写schema约束文件fancier-annotatio.xsd
3. 在类加载路径下创建META目录，编写约束映射文件spring.schemas和处理器映射文件spring.handlers
4. 编写命名空间处理器 FancierNamespaceHandler，在init方法中注册FancierBeanDefinitionParser 
5. 编写标签的解析器 FancierBeanDefinitiolParser，在parse方法中注册FancierBeanPostProcessor
6. 编写 FancierBeanPostProcessor

1 确定命名空间名称、schema虚拟路径、标签名称;
	

2 schema约束文件 fancier-annotatio.xsd

```java
<?xml version="1.0" encoding="UTF-8"?>  
  
<xsd:schema xmlns="http://www.fancier.com/fancy"  
            xmlns:xsd="http://www.w3.org/2001/XMLSchema"  
            targetNamespace="http://www.fancier.com/fancy">  
  
    <xsd:element name="annotation-driven"></xsd:element>  
  
</xsd:schema>
```

3.1 约束映射文件 spring.schemas 

```java
http\://www.fancier.com/fancy/fancier-annotation.xsd=com/fancier/config/fancier-annotation.xsd
```

3.2 处理器映射文件spring.handlers

```java
http\://www.fancier.com/fancy=com.fancier.handlers.FancierNamespaceHandler
```

4 编写命名空间处理器 FancierNamespaceHandler，在init方法中注册FancierBeanDefinitionParser 

```java
public class FancierNamespaceHandler extends NamespaceHandlerSupport {  
    @Override  
    public void init() {  
        //初始化，一般情况下，一个名空间下有多个标签，会在init方法中为每一个标签注册一个标签解析器  
        this.registerBeanDefinitionParser("annotation-driven",new FancierBeanDefinitionParser()); 
    }  
}
```

5 编写标签的解析器 FancierBeanDefinitiolParser，在parse方法中注册FancierBeanPostProcessor

```java
public class FancierBeanDefinitionParser implements BeanDefinitionParser {  
    @Override  
    public BeanDefinition parse(Element element, ParserContext parserContext) {  
        //注入一个BeanPostProcessor  
        BeanDefinition beanDefinition = new RootBeanDefinition();  
        beanDefinition.setBeanClassName("com.fancier.processor.FancierBeanPostProcessor");
	    parserContext.getRegistry()
		    .registerBeanDefinition("fancierBeanPostProessor",beanDefinition); 
        return beanDefinition;  
    }  
}
```

6 编写 FancierBeanPostProcessor

```java
public class HaohaoBeanPostProcessor implements BeanPostProcessor {  
    @Override  
    public Object postProcessAfterInitialization
    (Object bean, String beanName) throws BeansException {  
        System.out.println("FancierBeanPostProcessor执行....");  
        return bean;  
    }  
}
```


