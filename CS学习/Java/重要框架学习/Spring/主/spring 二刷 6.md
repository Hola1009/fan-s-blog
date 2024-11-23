### Bean 工厂后处理器 - BeanFactoryPostProcessor

Spring 提供了一个BeanFactoryPostProcessor的子接口BeanDefinitionRegistryPostProcessor专门用于注册BeanDefinition操作
```java
public class MyHostProcessor2 implements BeanDefinitionRegistryPostProcessor {  
    @Override  
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {  
        System.out.println("postProcessBeanDefinitionRegistry");  
        RootBeanDefinition beanDefinition = new RootBeanDefinition();  
        beanDefinition.setBeanClass(PersonDao.class);  
        registry.registerBeanDefinition("personDao", beanDefinition);  
    }  
  
    @Override  
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {  
        System.out.println("postProcessBeanFactory");  
    }  
}
```

```xml
<bean class="com.example.test1.process.MyHostProcessor2"></bean>
```

### 手写注解注入 Spring 容器
先写注解
```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface MyComponent {  
    String value();  
}
```
导入工具类
```java
public class BaseClassScanUtils {  
  
    //设置资源规则  
    private static final String RESOURCE_PATTERN = "/**/*.class";  
  
    public static Map<String, Class> scanMyComponentAnnotation(String basePackage) {  
  
        //创建容器存储使用了指定注解的Bean字节码对象  
        Map<String, Class> annotationClassMap = new HashMap<String, Class>();  
  
        //spring工具类，可以获取指定路径下的全部类  
        ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();  
        try {  
            String pattern = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +  
                    ClassUtils.convertClassNameToResourcePath(basePackage) + RESOURCE_PATTERN;  
            Resource[] resources = resourcePatternResolver.getResources(pattern);  
            //MetadataReader 的工厂类  
            MetadataReaderFactory refractory = new CachingMetadataReaderFactory(resourcePatternResolver);  
            for (Resource resource : resources) {  
                //用于读取类信息  
                MetadataReader reader = refractory.getMetadataReader(resource);  
                //扫描到的class  
                String classname = reader.getClassMetadata().getClassName();  
                Class<?> clazz = Class.forName(classname);  
                //判断是否属于指定的注解类型  
                if(clazz.isAnnotationPresent(MyComponent.class)){  
                    //获得注解对象  
                    MyComponent annotation = clazz.getAnnotation(MyComponent.class);  
                    //获得属value属性值  
                    String beanName = annotation.value();  
                    //判断是否为""  
                    if(beanName!=null&&!beanName.equals("")){  
                        //存储到Map中去  
                        annotationClassMap.put(beanName,clazz);  
                        continue;  
                    }  
                    //如果没有为"",那就把当前类的类名作为beanName  
                    annotationClassMap.put(clazz.getSimpleName(),clazz);  
                }  
            }  
        } catch (Exception exception) {  
        }  
        return annotationClassMap;  
    }  
}
```


实现 BeanDefinitionRegistryPostProcessor 后处理器 重写业务方法 , 注册BD
```java
@Slf4j  
public class MyHostProcess implements BeanDefinitionRegistryPostProcessor {  
    @Override  
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {  
  
    }  
    @Override  
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {  
        Map<String, Class> map = BaseClassScanUtils.scanMyComponentAnnotation("com.example.test1.Beans");  
        map.forEach((beanName, clazz) -> {  
            RootBeanDefinition beanDefinition = new RootBeanDefinition();  
            beanDefinition.setBeanClassName(clazz.getName());  
            registry.registerBeanDefinition(beanName , beanDefinition);  
        });  
    }  
}
```

在xml文件里去注入这个后处理器
```xml
<bean class="com.example.test1.process.MyHostProcess"></bean>
```

### Bean 后处理器 - BeanPostProcessor
Bean被实例化后，到最终缓存到名为singletonObjects单例池之前，中间会经过Bean的初始化过程，例如:属性的填充、初始方法init的执行等，其中有一个对外进行扩展的点BeanPostProcessor，我们称为Bean后处理。跟上面的Bean工厂后处理器相似，它也是一个接口，实现了该接口并被容器管理的BeanPostProcessor，会在流程节点上被
Spring自动调用。

```java
@Slf4j  
public class MyBeanPostProcessor implements BeanPostProcessor {  
    @Override  
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
        log.info("postProcessAfterInitialization");  
        return bean;  
    }  
  
    @Override  
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {  
        log.info("postProcessBeforeInitialization");  
        return bean;  
    }  
}
```
两个方法是在每个 bean  [[spring 二刷 2#Beand的初始化和销毁方法配置|初始化方法]] 前后执行


### 对 Bean 方法进行执行时间日志增强
要求如下:
- Bean 的方法执行之前控制台打印当前时间
- Bean 的方法执行之后控制台打印当前时间

分析:
- 对方法进行增强主要就是代理设计模式和包装模式;
- 由于 Bean 方法不确定, 所以使用[动态代理](动态代理.md)在运行期间执行增强
- 在 Bean 实例创建完毕之后, 进入到单例池, 使用 Procy 代替 真正的Bean

bean后处理器
```java
@Slf4j  
public class MyBeanPostProcessor implements BeanPostProcessor {  
    @Override  
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
        log.info("postProcessAfterInitialization");  
        return bean;  
    }  
  
    @Override  
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {  
        log.info("postProcessBeforeInitialization");  
        //使用动态代理对目标 Bean 进行增强  
        Object proxyBean = Proxy.newProxyInstance(  
                bean.getClass().getClassLoader(),  
                bean.getClass().getInterfaces(),  
                (proxy, method, args) -> {  
                    System.out.println("方法: " + method.getName() + " 开始时间:");  
                    Object result = method.invoke(bean, args);  
                    System.out.println("方法: " + method.getName() + " 开始时间:");  
                    return result;  
                }  
        );  
        return proxyBean;  
    }  
}
```


### BeanPostProcessor 在 SpringBean 中 实例化过程的体现
![[Pasted image 20240413203643.png|left|600]]
#### bug记录
因为 创建出来的 动态代理是一个新的类的对象, 所以从 容器中取出来后 要将其转成同有的接口才行
