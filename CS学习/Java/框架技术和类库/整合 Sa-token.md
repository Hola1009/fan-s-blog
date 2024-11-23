## Spring-boot 整合
导入依赖
```xml
<sa-token.version>1.38.0</sa-token.version>

<dependency>  
  <groupId>cn.dev33</groupId>  
  <artifactId>sa-token-reactor-spring-boot3-starter</artifactId>  
  <version>${sa-token.version}</version>  
</dependency>
```

编写配置
```yml
sa-token:  
  # token 名称（同时也是 cookie 名称）  
  token-name: Authorization  
  # token前缀  
  token-prefix: Bearer  
  # token 有效期（单位：秒） 默认30天，-1 代表永久有效  
  timeout: 2592000  
  # token 最低活跃频率（单位：秒），如果 token 超过此时间没有访问系统就会被冻结，默认-1 代表不限制，永不冻结  
  active-timeout: -1  
  # 是否允许同一账号多地同时登录 （为 true 时允许一起登录, 为 false 时新登录挤掉旧登录）  
  is-concurrent: true  
  # 在多人登录同一账号时，是否共用一个 token （为 true 时所有登录共用一个 token, 为 false 时每次登录新建一个 token）  
  is-share: true  
  # token 风格（默认可取值：uuid、simple-uuid、random-32、random-64、random-128、tik）  
  token-style: random-128  
  # 是否输出操作日志  
  is-log: true
```

## 整合 Redis
导入依赖
```xml
<!-- Redis -->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>  
  
<!-- Redis 连接池 -->  
<dependency>  
    <groupId>org.apache.commons</groupId>  
    <artifactId>commons-pool2</artifactId>  
</dependency>
```

添加配置
```yml
spring:
  data:  
    redis:  
      database: 1 # Redis 数据库索引（默认为 0）  
      host: 127.0.0.1 # Redis 服务器地址  
      port: 6379 # Redis 服务器连接端口  
      password: fancier123 # Redis 服务器连接密码（默认为空）  
      timeout: 5s # 读超时时间  
      connect-timeout: 5s # 链接超时时间  
      lettuce:  
        pool:  
          max-active: 200 # 连接池最大连接数  
          max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）  
          min-idle: 0 # 连接池中的最小空闲连接  
          max-idle: 10 # 连接池中的最大空闲连接
```

向容器中注入 RedisTemplate 通过 StringRedisSerializer 和 Jackson2JsonRedisSerializer 来设置序列化和反序列化的值为 String

## 发送验证码业务逻辑设计
![[Pasted image 20240923113707.png|500]]

## 登录业务逻辑设计
![[Pasted image 20240923134032.png|600]]

> [! info] 角色权限设计
> 因为博客系统只有两个角色, 而且游客只有浏览和评论的权限, 就不涉及角色表和权限表了, 直接将用户角色添加为用户表的一个字段

## SaToken 整合 Redis: 存储 Session
很简单, 导入的依赖如下
```xml
<!-- Sa-Token 整合 Redis （使用 jackson 序列化方式） -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-redis-jackson</artifactId>
    <version>1.39.0</version>
</dependency>

<!-- 提供Redis连接池 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>

```

再到项目配置文件中配置一下 redis 就可以
```yml
redis:  
  # Redis数据库索引（默认为0）  
  database: 1  
  # Redis服务器地址  
  host: 127.0.0.1  
  # Redis服务器连接端口  
  port: 6379  
  # Redis服务器连接密码（默认为空）  
  password: fancier123  
  # 连接超时时间  
  timeout: 10s  
  lettuce:  
    pool:  
      # 连接池最大连接数  
      max-active: 200  
      # 连接池最大阻塞等待时间（使用负值表示没有限制）  
      max-wait: -1ms  
      # 连接池中的最大空闲连接  
      max-idle: 10  
      # 连接池中的最小空闲连接  
      min-idle: 0
```