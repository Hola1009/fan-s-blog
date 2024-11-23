## Jedis
1. 依赖
```xml
<dependency>  
    <groupId>redis.clients</groupId>  
    <artifactId>jedis</artifactId>  
    <version>3.7.0</version>  
</dependency>
```
2. 建立连接
![[Pasted image 20241111083011.png]]
3. 使用
```java
@Test  
void testString() {    
	// 插入数据，方法名称就是redis命令名称，非常简单
	String result = jedis.set("name", "张三");  
    System.out.println("result = " + result);    
    // 获取数据
	String name = jedis.get("name");  
    System.out.println("name = " + name);  
}
```
4. 释放资源

## Jedis 连接池
jedis 线程不安全, 频繁的创建和销毁连接会有性能消耗

```java
public class JedisConnectFactory {  
    private static final JedisPool jedisPool;  
      
    static  {  
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();  
        jedisPoolConfig.setMaxTotal(500); // 最大连接  
        jedisPoolConfig.setMaxIdle(100); // 最大空闲连接  
        jedisPoolConfig.setMinIdle(50); // 最小空闲连接  
        jedisPoolConfig.setMaxWaitMillis(200); // 等待市场  
        jedisPool = new JedisPool(jedisPoolConfig, "localhost", 6379, 1000, "fancier123");  
    }  
          
	public static Jedis getResource() {  
        return jedisPool.getResource();  
    }  
}
```

## SpringDataRedis
SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis，官网

- 提供了对不同Redis客户端的整合（Lettuce和Jedis）
- 提供了RedisTemplate统一API来操作Redis
- 支持Redis的发布订阅模型
- 支持Redis哨兵和Redis集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
- 支持基于Redis的 JDKCollection 实现


### 快速入门
SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

|                             |                 |                     |
| --------------------------- | --------------- | ------------------- |
| API                         | 返回值类型           | 说明                  |
| redisTemplate.opsForValue() | ValueOperations | 操作 `String` 类型数据    |
| redisTemplate.opsForHash()  | HashOperations  | 操作 `Hash` 类型数据      |
| redisTemplate.opsForList()  | ListOperations  | 操作 `List` 类型数据      |
| redisTemplate.opsForSet()   | SetOperations   | 操作 `Set` 类型数据       |
| redisTemplate.opsForZSet()  | ZSetOperations  | 操作 `SortedSet` 类型数据 |
| redisTemplate               |                 | 通用的命令               |


1. 依赖
```xml
<!--  redis 依赖  -->  
<dependency>  
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>  
<!--  连接池依赖  -->  
<dependency>  
  <groupId>org.apache.commons</groupId>  
  <artifactId>commons-pool2</artifactId>  
</dependency>
```

2. 配置文件
```yml
spring:  
  redis:  
    host: 127.0.0.1  
    port: 6379  
    password: fancier123  
    lettuce:  
      pool:  
        max-active: 100  
        max-idle: 50  
        min-idle: 20  
        max-wait: 100
```

3. 注入 RedisTemplate 
```java
private final RedisTemplate<String, Object> redisTemplate;
```
4. 执行操作
```java
redisTemplate.opsForValue().set(key, value);  
```

### 序列化
RedisTemplate 内部维护了这么写序列化工具默认为 null
![[Pasted image 20241111091638.png|500]]

如果没有为其赋值会使用默认的序列化工具, 只不过默认采用JDK序列化，会把Object`序列化为字节形式`，得到的结果是这样的：
![[Pasted image 20241111092237.png]]
这样做的缺点是: 可读性差 和 内存占用大

## 序列化
1. GenericJackson2JsonRedisSerializer 转json对象序列化方式

### 修改序列化方式
修改序列化, 注册一个 bean 把之前的 bean 顶掉
```java
@Bean  
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {  
    // 创建Template  
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();  
    // 设置连接工厂  
    redisTemplate.setConnectionFactory(redisConnectionFactory);  
    // 设置序列化工具  
    GenericJackson2JsonRedisSerializer jsonRedisSerializer =   
            new GenericJackson2JsonRedisSerializer();  
    // key和 hashKey采用 string序列化  
    redisTemplate.setKeySerializer(RedisSerializer.string());  
    redisTemplate.setHashKeySerializer(RedisSerializer.string());  
    // value和 hashValue采用 JSON序列化  
    redisTemplate.setValueSerializer(jsonRedisSerializer);  
    redisTemplate.setHashValueSerializer(jsonRedisSerializer);  
    return redisTemplate;  
}
```

使用 json 格式序列化在序列化的时候会会写入类的信息, 所以反序列化的时候可以正确序列化成预期的对象
![[Pasted image 20241111094114.png]]


### StringRedisTemplate
上面的序列化方式虽然方便但是美中不足的是, 因为需要额外存储类信息, 需要占用更多的内存, 为了节省空间, 可以使用 String 序列化器, 当存储取 Java 对象时, 手动完成序列化和反序列化, `StringRedisTemplate` 就是一个的解决方案
序列化和反序列化可以使用 `ObjectMapper` 来实现
















