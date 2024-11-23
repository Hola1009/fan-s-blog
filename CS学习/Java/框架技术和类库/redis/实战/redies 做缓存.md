	缓存: 临时存储数据的地方, 为了提高好的读写性
作用: 
- 降低后端负载
- 提高读写性, 降低响应时间
成本:
- 数据一致性成本
- 代码维护成本
- 运维成本

## 添加 redis 缓存
![[Pasted image 20241111164458.png|400]]
比较容易理解, 我的疑问是怎么去组织数据 -> 课程中是使用 前缀 + id 来作为key

## 缓存更新策略
- 内存淘汰
- 超时剔除
- 主动更新
![[Pasted image 20241111165257.png]]

### 主动更新策略
☑️ 更新数据库时同步更新缓存
❌ 操作者只操作缓存, 异步更新数据库

需要考虑的问题?
1. 删除缓存还是更新
	- 每次更新数据库都更新缓存, 无效写操作多(写了没查就更新了) ❌
	- 删除缓存: 更新数据库时让缓存失效, 查询时更新缓存 ☑️
2. 如何保证缓存与数据库的操作的同时成功或失败？
	- 单体系统，将缓存与数据库操作放在一个事务
	- 分布式系统，利用TCC等分布式事务方案
3. 先删除缓存还是先更新数据库?

> [!question] 我的问题
> 怎么将缓存和数据库操作放在一个事务

#### 先删缓存
存在的问题是, 在一次更新的删除缓存和更新数据库的间隔内, 另外一个线程来读取缓存后未击中后, 用数据库的旧值来更新缓存, 导致数据不一致问题
![[Pasted image 20241111171303.png|400]]

#### 先更新数据库
(线程1)在更新数据库和删除缓存期间, 另外一个线程(线程2)读取缓存未命中后查询数据库写入缓存, 这样不会有任何问题, 除非删除缓存的操作发生在(线程2)查询数据数据库和写入缓存操作之间, 但这个可能性是比较低的, 所以还是先更新数据库这个策略比较优秀
![[Pasted image 20241111171701.png|400]]
> [!summary] 关键点在于把耗时多的操作先执行

## 缓存穿透
指请求的数据在缓存和数据库都不存在, 从而造成每次请求都会走数据库

### 解决方法

#### 缓存空对象
查数据库没查到就缓存空对象
- 优点: 实现简单
- 缺点:
	- 额外内存消耗
	- 短期数据不一致 缓存空对象期间, 数据库插入了对应的数据, 因为使用主动更新策略, 会造成短期不一致
#### 布隆过滤
优点: 内存占用少
缺点: 实现复杂, 存在误判
![[Pasted image 20241113164405.png|400]]

## 缓存雪崩
缓存雪崩是指在同一时间段`大量的缓存 key 同时失效`或者 Redis 服务宕机, 导致`大量请求到达数据库`, 带来巨大压力

### 解决方案
- 给不同的 key 的 TTL 添加随机值
- 利用 Redis 集群极高服务的可用性
- 给缓存业务添加降级限流策略
- 给业务添加多级缓存

## 缓存击穿
也叫`热点 Key 问题`, 就是一个被`高并发访问`并且`缓存重建业务较复杂` (多表查询) 的 Key 突然失效了, 无数的请求访问会瞬间给数据库带来巨大的冲击 

### 解决方案
#### 互斥锁
未命中后需要获取到锁才能执行数据库操作
  ![[Pasted image 20241113170312.png|400]]
####  逻辑过期
不给 Key 设置 TTL, 而是设置一个 expire 的值, 超过了时间就尝试去获取锁然后`开新线程`更新数据库
> [!warning] differ with 互斥锁
> 这个方案没拿到锁不会去重试, 而是直接拿旧值

![[Pasted image 20241113170933.png|500]]

#### 优缺点比较
- 互斥锁优点:
	- 没有额外的内存消耗
	- 保证一致性
	- 实现简单
- 互斥锁缺点
	- 线程需要等待, 性能受到影响
	- 可能有死锁风险
- 逻辑过期优点:
	- 线程无需等待, 性能好
- 逻辑过期缺点:
	- 一致性差
	- 有额外的内存消耗
	- 实现复杂


> [!summry]
> 一个一致性好, 一个可用性好



#### 互斥锁的实现
> [!question]
> 1. 拿什么当锁?
因为没拿到锁的逻辑需要自定义所以不能使用 synchronized 而是使用 tryLock 或是 redis 中的 `setnx`
> 2. 锁的值?
>    每个资源对应一把锁, 所以 使用 锁前缀 + 资源id
> 3. 锁重试怎么实现?
>    我的想法是用循环, 但是使用`递归`确是更加优雅

> [!warning]
> 1. 使用注意来获取到锁后还要判断一遍资源是否存在
> 2. 因为涉及到释放锁, 所以需要在把释放锁的逻辑写道 finally


![[Pasted image 20241113174013.png|600]]


这里使用 setnx 来实现
##### api: 
```java
// 获取锁
redisTemplate.opsForValue().setIfAbsent("lock", 1, 10, TimeUnit.seconds);
// 释放锁
redisTemplate.delete("lock");
```

#### 逻辑过期的实现
> [!tip]
> 为了减轻代码侵入性, 不破坏原来代码的基础上, 为存入 redis 中的数据加上一些额外的字段, 比如过期时间, 可以编写一个包装类, 加过期时间和原来的数据封装进这个类
>```java
>@Data  
>public class RedisData {  
>    private LocalDateTime expireTime;  
>    private Object data;  
>}
> ```

> [!tip] data 怎么反序列化
> 将结果类型转成 JsonObject, 然后调用 hutool 包的 JSONUtil.toBean 方法

> [!tip]
> 存入过期时间到 redis 中, 为了方便后续比较应该存入的是终止时间

![[Pasted image 20241113192926.png]]


## 缓存工具封装
需要满足一下需求
1. 方法1：将任意 Java 对象序列化为 json 并存储在 string 类型的 key 中，并且可以设置 TTL 过期时间
2. 方法2：将任意 Java 对象序列化为 json 并存储在 string 类型的key中，并且可以设置逻辑过期时间，用于处理缓存击穿问题
3. 方法3：根据指定的 key 查询缓存，并反序列化为指定类型，利用缓存空值的方式解决缓存穿透问题
4. 方法4：根据指定的key查询缓存，并反序列化为指定类型，需要利用逻辑过期解决缓存击穿问题

1) 方法1
```java
public void set(String key, Object value, Long time, TimeUnit unit) {  
    stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(value), time, unit);  
}
```

2) 方法2
```java
public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit unit) {  
    RedisData data = RedisData.builder()  
            .data(value)  
            .expireTime(LocalDateTime.now().plusSeconds(unit.toSeconds(time)))  
            .build();  
    stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(data));  
}
```

>[!info] 
>时间单位对象将给定的时间转换为秒, 
>```java
>unit.toSeconds(time)
> ```

3) 方法3
```java
public <R,ID> R queryWithPassThrough(  
        String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit){  
    String key = keyPrefix + id;  
    // 1.从redis查询缓存  
    String shopJson = stringRedisTemplate.opsForValue().get(key);  
      
    // 2. 如果击中，直接返回  
    if (StrUtil.isNotBlank(shopJson)) {  
        return JSONUtil.toBean(shopJson, type);  
    }  
    // 判断命中的是否是空值 -> 即查到了 ""if (json != null) {  
    // 返回一个错误信息  
    return null;  
}
    // 3. 如果未击中, 调用 dbFallback 方法查询数据库  
    R entity = dbFallback.apply(id);  
      
    // 4. 不存在则缓存空来解决缓存穿透问题  
    if (entity == null) {  
        stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);  
        return null;  
    }  
    // 5. 如果数据库存在则进行缓存  
    return JSONUtil.toBean(shopJson, type);  
}
```

4) 方法4
方法太长了, 去项目里看吧





















