## 1. 使用 @PostConstruct 注解
```java
@PostConstruct public void init() {
	// 初始化逻辑 
	System.out.println("MyBean initialized"); 
}
```

## 2. 使用 @Bean 注解的 initMethod 属性
```java
@Configuration  
public class AppConfig {  
  
    @Bean(initMethod = "initMethod")  
    public MyBean myBean() {  
        return new MyBean();  
    }  
}
```

## 3. 实现 InitializingBean 接口
```java
public class MyBean implements InitializingBean {  
  
    @Override  
    public void afterPropertiesSet() throws Exception {  
        // 初始化逻辑  
        System.out.println("MyBean initialized via InitializingBean interface");  
    }  
}
```