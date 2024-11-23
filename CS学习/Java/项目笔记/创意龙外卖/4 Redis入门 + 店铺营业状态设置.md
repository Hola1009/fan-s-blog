### 简介
是基于一个 `内存` 的K-V结构的数据库
- 基于内存存储, 读写性能高
- 适合存储热点数据(热点新闻, 咨询, 新闻)
- 企业应用广泛


### Redis下载与安装
下载绿色版解压即用


### Redis服务启动与停止
#### 启动
在安装目录下输入cmd
```
redis-server.exe redis.windows.conf
```

#### 停止
ctrl + c  即可

#### 连接客户端
默认连接本地
```
redis-cli.exe
```

连接其他地方的redis
```
redis-cli.exe -h localhost -p 6379
```
有密码就在后面加上 -a '密码'



### Redis最常用的数据类型
#### 5中常用的数据类型
reids是k-v结构 其中的k是字符串类型, v常用的类型有下面5种
- 字符串 string
- 哈希 hash
- 列表 list
- 集合 set
- 有序集合 sorted set/zset

#### 各种数据类型的特点
![[Pasted image 20240323094211.png|left|700]]

### Redis中的常用命令
不同类型操作不一
#### 字符串操作
![[Pasted image 20240323094505.png|left|600]]

#### 哈希操作命令
![[Pasted image 20240323100011.png|left|600]]

#### 列表操作命令
![[Pasted image 20240323100903.png|left|500]]

#### 集合操作
![[Pasted image 20240323103008.png|left|500]]

#### 有序集合操作的命令
![[Pasted image 20240323105427.png|left]]

#### 同用命令
与具体数据类型无关
![[Pasted image 20240323105537.png|left|500]]

### 在java中操作Redis
#### Redis的Java客户端
- Jedis 易上手
- Lettuce 性能高
- Spring Data Redis 使用简单
#### Spring Data Redis 使用方式

##### 操作步骤
1. 导入Spring Data Redis 的 maven 的坐标
2. 配置 Redis 数据源
3. 编写配置类, 创建RedisTemplate对象
4. 通过RedisTemplate对象操作Redis
```java
<dependency>  
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-data-redis</artifactId>  
  <version>2.7.3</version>  
</dependency>
```

```yml
spring:
  redis:  
	  host: ${sky.redis.host}  
	  port: ${sky.redis.port}  
	  database: ${sky.redis.database} # 共有16个数据库 非必须 默认为0
```

```java
@Configuration  
@Slf4j  
public class RedisConfiguration {  
    @Bean  
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {  
        log.info("开始创建redis模板对象...");  
        RedisTemplate template = new RedisTemplate<>();  
        //设置连接工厂对象 引入的依赖会为我们能创建好工厂对象 并且放到Spring容器中  
        template.setConnectionFactory(redisConnectionFactory);  
        //设置redis key的序列化器  
        template.setKeySerializer(new StringRedisSerializer());  
  
        return template;  
    }  
}
```

```java
@Test  
public void restRedisTemplate() {  
    System.out.println(redisTemplate);  
    //可以使用这五个对象分别操作不同类型的数据  
    ValueOperations valueOperations = redisTemplate.opsForValue();  
    SetOperations setOperations = redisTemplate.opsForSet();  
    HashOperations hashOperations = redisTemplate.opsForHash();  
    ListOperations listOperations = redisTemplate.opsForList();  
    ZSetOperations zSetOperations = redisTemplate.opsForZSet();  
}
//以操作字符串类型为例
@Test  
public void testString() {  
    //set get setex setnx  
    ValueOperations valueOperations = redisTemplate.opsForValue();  
    valueOperations.set("name", "Fancier");  
    String name = (String) valueOperations.get("name");  
    System.out.println(name);  
    valueOperations.set("code" ,"1234", 3, TimeUnit.MINUTES);  
    //v 可以是任何值, 存进去时会序列化成字符串  
    valueOperations.setIfAbsent("lock", "1");  
    valueOperations.setIfAbsent("lock", "2");  
}
```

### 店铺营业状态设置
#### 需求分析
![[Pasted image 20240323140745.png|left|300]]

##### 接口设计
- 设置营业状态
- 管理端查询营业状态
- 用户端查询营业状态

因为营业状态只有一个字段, 而且查询数量大, 所以将其存储在Redis里
#### 代码开发






#### 功能测试









