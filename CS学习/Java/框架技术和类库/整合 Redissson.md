## 整合
1. 引入依赖
```xml
<dependency>  
    <groupId>org.redisson</groupId>  
    <artifactId>redisson</artifactId>  
    <version>3.13.6</version>  
</dependency>
```

2. 配置 Redisson 客户都
```java
@Configuration  
public class  RedissonConfig {  
  
    @Bean  
    public RedissonClient redissonClient(){  
        // 配置  
        Config config = new Config();  
	    // 这块可以把配置文件中的数据封装进一个 RedisProperties 的 Bean, 然后读取
	    // 这里使用单节点模式
		config.useSingleServer()
			.setAddress("redis://192.168.150.101:6379")
			.setPassword("123321");  
        // 创建RedissonClient对象  
        return Redisson.create(config);  
    }  
}
```

## 使用
### 可重入锁
第一个参数为 `等待时间` , 第二个参数为 `超时释放时间` , 第三个参数为 `时间单位`

```java
boolean isLock = lock.tryLock(1, 10, TimeUnit.SECONDS);  
```

```java
@Resource
private RedissonClient redissonClient;  
  
@Test  
void testRedisson() throws InterruptedException {  
    // 获取锁（可重入），指定锁的名称  
    RLock lock = redissonClient.getLock("anyLock");       
    // 尝试获取锁，参数分别是：获取锁的最大等待时间（期间会重试），锁自动释放时间，时间单位  
    boolean isLock = lock.tryLock(1, 10, TimeUnit.SECONDS);      
    // 判断释放获取成功  
    if(isLock){  
        try {  
            System.out.println("执行业务");  
        }finally {              
			// 释放锁  
            lock.unlock();  
        }  
    }  
}
```

#### 可重入原理
redis 中的锁结构选用 hash 结果, 存入 `线程标识` 和 `重入次数` 两个字段, 获取锁的时候, 判断线程标识相等后会重入次数 + 1, 释放锁的时候, 判断线程标识相等后, 对重入次数减1
![[Pasted image 20241116072115.png|600]]
为了保证原子性, 这些代码应该用lua脚本来执行, 用 java 代码的化不同的线程会在代码间隔之间穿插
- 获取锁
```lua
local key = KEYS[1]; -- 锁的key  
local threadId = ARGV[1]; -- 线程唯一标识  
local releaseTime = ARGV[2]; -- 锁的自动释放时间  
-- 判断是否存在  
if(redis.call('exists', key) == 0) then
    -- 不存在, 获取锁   
     redis.call('hset', key, threadId, '1');
    -- 设置有效期    
    redis.call('expire', key, releaseTime);    
    return 1; -- 返回结果  
end;  
-- 锁已经存在，判断threadId是否是自己  
if(redis.call('hexists', key, threadId) == 1) then
    -- 不存在, 获取锁，重入次数+1    
    redis.call('hincrby', key, threadId, '1');
    -- 设置有效期
    redis.call('expire', key, releaseTime);    return 1; -- 返回结果  
end;  
return 0; -- 代码走到这里,说明获取锁的不是自己，获取锁失败
```

- 释放锁
```lua
local key = KEYS[1]; -- 锁的key  
local threadId = ARGV[1]; -- 线程唯一标识  
local releaseTime = ARGV[2]; -- 锁的自动释放时间  
-- 判断当前锁是否还是被自己持有  
if (redis.call('HEXISTS', key, threadId) == 0) then  
    return nil; -- 如果已经不是自己，则直接返回  
end;  
-- 是自己的锁，则重入次数-1  
local count = redis.call('HINCRBY', key, threadId, -1);  
-- 判断是否重入次数是否已经为0  
if (count > 0) then    
	-- 大于0说明不能释放锁，重置有效期然后返回    
	redis.call('EXPIRE', key, releaseTime);    
	return nil;  
else  -- 等于0说明可以释放锁，直接删除
	redis.call('DEL', key);
	return nil;  
end;
```

#### 重试原理
tryLock 如果没传释放时间, 默认会设置一个看门狗等待时间 `30s`
执行 lua 获取锁的脚本, 成功会返回null, 失败返回 剩余有效期
返回的剩余有效期不为null, 计算出消耗的时间, 判断时间还有剩余否, 有则继续尝试去获取锁
订阅释放锁信号, 等待剩余时间清零否, 还有剩余时间就继续之前的逻辑

#### 解决因阻塞而释放锁
在获取到锁后会触发一个定时任务, 每过 1/3 ttl 就自动续约 (循环是依赖于递归实现的: 在递归方法的设置定时任务, 定时任务方法触发后会调用递归方法)

ExpirationEntry 封装了 线程 id 和 定时任务(每次递归都会重新赋值) 每一个 ExpirationEntry 被 RedissonLock 的静态成员 EXPIRATION_RENEWAL_MAP 维护

在释放锁的方法里有取消定时任务的方法, 从 EXPIRATION_RENEWAL_MAP 拿到 ExpirationEntry 后拿到任务然后取消
![[Pasted image 20241116104318.png]]

> [!tip]
> 不传 waitTime 表示不会重试

> [!warning]
> 如果, 设置了 ttl 则不会触发看门狗机制

> [!warning]
> tryLock 重载了3个方法, 
> - 无参: 没有 (waitTime) 只会尝试一次
> - 3 个参数: (waitTime. leaseTime, timeUnit) , 会重试, 不会触发看门狗
> - 2个参数: (waitTime, imeUnit ) 会重试, 会触发看门狗


### Redisson 分布式锁主从一致性问题
> [!question] 主从模式存在的问题
> 主节点负责写, 而从节点负责写
> 如果在同步完成之前 主节点宕机了, 从节点中一个节点晋升为主节点, 但由于主从同步未完成, 其他线程来获取锁也能获取成功, 就出现了并发安全问题

一个解决方案是, 部分主从, 把每个节点都当主节点看, 都可以进行读写, 获取锁时, 依次向每个节点都获取锁

#### api

##### 创建联锁
```java
RLock lock1 = redissconClient.getLock("order");
RLock lock2 = redissconClient.getLock("order");
RLock lock3 = redissconClient.getLock("order");

lock = redissonClient.getMultilock(lock1, lock2, lock3);
```

源码逻辑:
如果传了 leaseTime 且 waitTime 没传 -> 只尝试一次且不触发看门狗
如果都传了 将 leaseTime 设未 waitTime 的两倍, 防止还没获取到锁, 锁就释放了

遍历锁集合依次尝试取获取锁, 获取锁成功了将其放入获取成功的锁集合, 失败了看一下是否获取到了足够的锁(可能不要求全部获取), 如果是就返回跳出循环, 如果不是再看一下是否是不允许失败, 不是就接着遍历, 是则判断 waitTime 是否等于 -1 等于 -1 则不再经行重试直接返回 false 不是就清空已获得锁的容器, 再将指针移至开头, 重新开始遍历. 另外, 没次遍历结尾都会计算下剩余时间是否充足

遍历结束后, 如果 leaseTime != -1 表示, 不触发看门狗机制, 因为之前获取到的没把锁的时间没有对齐, 所以超时时间也没有对其, 需要重新续约, 来对其一下, 

> [!warning]
> 如果没有设置 waitTime 则只会尝试一次

> [!summary]


1) 不可重入Redis分布式锁：
	- 原理：利用setnx的互斥性；利用ex避免死锁；释放锁时判断线程标示
	- 缺陷：不可重入、无法重试、锁超时失效
2) 可重入的Redis分布式锁：
	- 原理：利用hash结构，记录线程标示和重入次数；利用watchDog延续锁时间；利用信号量控制锁重试等待
	- 缺陷：redis宕机引起锁失效问题
3) Redisson的multiLock：
	- 原理：多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功
	- 缺陷：运维成本高、实现复杂














