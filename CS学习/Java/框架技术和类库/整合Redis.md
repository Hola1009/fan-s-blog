## 依赖
```xml
<!-- 提供Redis连接池 -->  
<dependency>  
  <groupId>org.apache.commons</groupId>  
  <artifactId>commons-pool2</artifactId>  
</dependency>  
  
<dependency>  
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>
```

## ReidsTemplate 序列化设置
设置 key 的序列化方法为字符串, value 的序列化方式为 json
```java
@Bean  
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {  
    RedisTemplate<String, Object> template = new RedisTemplate<>();  
    template.setConnectionFactory(connectionFactory);  
  
    template.setKeySerializer(new StringRedisSerializer());  
    template.setValueSerializer(new GenericJackson2JsonRedisSerializer());  
  
    return template;  
}
```

