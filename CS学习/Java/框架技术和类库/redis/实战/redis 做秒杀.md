## 全局 ID 生成器
建张表用来保存每张优惠卷的信息
 > [!question] 主键不采用自增长
> - id的规律性太明显
> - 受单表数据量的限制(一张表存不下了, 就要使用多张表)

全局 id 生成器, 分布式系统下生成, 特性如下
- 唯一性
- 高可用性: 随时可用
- 高性能: 生成 id 速度快
- 递增性
- 安全性(递增规则不明显)

为了增加 ID 的安全性, 不直接使用 Redis 自增的数值, 而是拼接一些其他的信息

id 组成部分
- 符号位: 1
- 时间戳
- 序列号: 秒内的计数器，支持每秒产生2^32个不同ID
![[Pasted image 20241115095915.png]]

### 自定义通用 ID 生成器
```java
public long nextId(String keyPrefix) {  
    // 1.生成时间戳  
    LocalDateTime now = LocalDateTime.now();  
    long nowSecond = now.toEpochSecond(ZoneOffset.UTC);  
    long timestamp = nowSecond - BEGIN_TIMESTAMP;  
  
    // 2.生成序列号  
    // 2.1.获取当前日期，精确到天  
    String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));  
    // 2.2.自增长  
    String key = "icr:" + keyPrefix + ":" + date;  
  
    long count = stringRedisTemplate.opsForValue().increment(key);  
  
    // 3.拼接并返回  
    return timestamp << COUNT_BITS | count;  
}
```

## 实现优惠卷秒杀下单
### 库表设计
优惠卷库
- 商铺id
- 优惠卷信息等
- 支付方式
- 类行: 普通卷, 秒杀卷

秒杀卷
- 关联的优惠卷信息
- 生效时间
- 失效时间
- 库存

### 实现
下单时需要校验
- 时间
- 库存


## 超卖问题
如果在一个线程查询库存到减库存的间隔内, 另一个线程进行的查询, 就会出现线程安全问题
![[Pasted image 20241115131937.png|300]]
> [!summary] 出现线程安全问题的原因
> 操作共享资源的代码有好几行, 多个线程在这几行代码之间穿插操作就会存在线程安全问题


### 解决方案

悲观锁:认为线程安全问题一定发生 
- synchronized, Lock 性能差
乐观锁:认为线程安全问题不一定会发生，因此不加锁，只是在更新数据时去判断有没有其它线程对数据做了修改。
- 如果没有修改则认为是安全的，自己才更新数据。
- 如果已经被其它线程修改说明发生了安全问题，此时可以重试或异常


#### 乐观锁的实现
乐观锁的关键是判断之前查询到的数据是否被修改过, 只是简陋的确保操作是原子的

- `版本号法` : 再修改之前再查询一次, 根据版本号判断是否被修改,
- `CAS 法` : 和版本号法的思想是一样的, 不过需要要添加额外的字段, 而是判断目标数据是否被修改来判断 (这对数据的修改行为有要求), 因为, 库存被用户操作只能减少, 所以适用 CAS 法

> [!tip] 可以之间把判断条件写入SQL语句
> 在 where 子句中判断库存是否和第一次查询的相等


##### 优化
> [!warning] 成功率太低
> 只要之前有人修改了, 就会放弃尝试, 失败率太高

之前是判断库存是否与第一次查询相等去减少库存, 可以优化为判断库存是否大于 0

## 一人一单
为了避免同一个用户同时发送多个请求利用线程安全问题钻漏洞, 我们需要为查询一人一单的操作加锁, 所谓 userId 的字符串字面量 `userId.toString().itern()`

> [!warning] 注意
> 如果加锁的代码放在了事务方法里, 则仍然存在线程安全问题, 因为在释放锁了, 事务未提交, 数据库没有修改前, 其他线程会乘虚而入执行查询操作, 所以应该给把 synchronized 关键字提取出方法

> [!tip] 解决事务失效
> ![[Spring AOP#怎么解决自调用事务失效问题]]

## 基于 Redis 分布式锁
因为不同机器的 JVM 不同, 所以之前的加锁方案不生效了, 分布式锁是一种解决方案
- 多进程可见
- 互斥
- 高性能
- 高可用性
- 安全性
![[Pasted image 20241115153354.png]]


### 核心方法
- 获取锁: 为了避免服务宕机, 造成死锁, 需要设置超时时间
```bash
setnx lock thread1
expire lock 10
# 或者
set lock thread1 nx ex 10
```
- 释放锁
```bash
del lock
```

### 实现
将获取锁和释放锁的逻辑封装到一个类里, 然后和 Lock 类似的适用方式适用即可


### 误删问题
因为业务阻塞时间超过了超时时间, 导致业务没执行完锁就被释放了
![[Pasted image 20241115163918.png|600]]
存在的问题
1. 线程安全问题
2. 释放了别人锁

#### 解决方案
在获取锁时, 存入线程的标识, 释放锁的时候需要查询标识, 判断标识是否相同, 再释放
> [!tip] 获取线程 id 的方法
> ```java
> Thread.currentThread().getId();
>```

#### lua 脚本解决原子性问题
> [!error] 极端情况
> `比如垃圾回收被触发的时候会阻塞所有代码`
在*测试线程标识通过*后, 释放锁之前, 线程被阻塞了, 锁被超时释放了, 然后其他线程乘虚而入获取到了锁, 然后阻塞结束, 原来的线程释放了不属于他的锁

> [!important] redis call 方法执行命令
> ```sql
> redis.call('命令名称', 'key', '其他参数')
> # 例如
> redis.call('set', 'name', 'jack')
> ```


有了 redis call 就可以来写脚本了, 

```sql
# 调用脚本
# 第 3 个参数为 key 类型的参数个数
eval "return redis.call('set', KSYS[1], ARGV[1])" 0 name jack
```

如果 key , value 不想写死, 可以作为参数传递, key 放入 KEYS 数组, 其他参数 放入 ARGV 数组, 在脚本中调用这两个数组去获取参数, (数组下标是从 1 开始的)

```lua
-- 比较线程标示与锁中的标示是否一致  
if(redis.call('get', KEYS[1]) ==  ARGV[1]) then  
	-- 释放锁 del key
	return redis.call('del', KEYS[1])  
end  
return 0
```

##### 使用 Java 执行 lua 脚本
在 RedisTemplate 中专门执行脚本的 api 
```java
public <T> T execute(RedisScript<T> script, List<K> keys, Object... args) { /* compiled code */ }
```

脚本需要被 RedisScript 读取, 可以将其封装成一个静态成员变量, 避免每次调用的时候临时读取, 设置脚本的 api 如下
```java
redisScript.setLocation(new ClassPathResource("unlock.lua"));
```

> [!tip] 创造单元素集合的方法
>```java
> Collections.singletonList(KEY_PREFIX + name);
> ```

### 基于 Redis 的分布式锁优化
1. 不可重入
2. 不可重试
3. 超时释放: 虽然可以解决死锁, 但是如果业务被阻塞了, 锁会在业务完成前释放
4. 主从一致性

Redisson 是一个在 Redis 的基础上实现的Java驻内存数据网格, 提供了一系列分布式的 Java 常用对象, 还提供了许多分布式服务, 其中就包含了分布式锁的实现, 可以使用 Redisson 获取锁, 释放锁的 api 来代替我们原来自己编写的锁api

[[整合 Redissson]]

## 秒杀业务优化
将之前需要加锁的业务逻辑放进 redis 来做(即用 lua 脚本来执行), 秒杀库存 和 校验 一人一单 依赖的数据用 redis 来保存
![[Pasted image 20241118060442.png]]
这 4 处执行耗时比较久, 可以另起线程去执行
1. 查询优惠券: 用来校验秒杀是否开始
2. 查询订单: 要来校验一人一单(冗余校验, 因为 redis 已经校验过了)
3. 创建订单: 将订单数据插入数据库
![[Pasted image 20241118083950.png]]

### 存储的数据结构
对于优惠券
![[Pasted image 20241118090021.png]]
对于一人一单使用 set 集合来保存没一个买过用户id
![[Pasted image 20241118062851.png]]

发送获取优惠券请求后, 先 执行 lua 脚本判断库 和 一人一单, 校验通过, 生成订单 id 后将订单相关信息存入阻塞队列 

### 实现
1. 新增优惠券的同时, 将优惠券信息保存(优惠券id 和 存库)到 Redis 中
2. 基于 Lua 脚本, 判断秒杀库存, 一人一单, 决定是否抢购成功
3. 如果抢购成功, 将优惠券id和用户id封装后存入阻塞队列
4. 开启任务线程, 不断从阻塞队列中获取信息, 实现异步下单

#### 代码逻辑:
生成orderId , 执行 lua 脚本校验 库存 和 一人一单, 创建订单放入阻塞队列(这里暂且用原生的数据结构来做), 创建消费消息队列的任务, 启动线程来执行任务

##### 脚本逻辑
![[Pasted image 20241118090306.png|400]]

## Redis 消息队列实现异步秒杀
> [!question]
> 之前的消息对象使用的是java原生的数据结构, 存在的问题
> - 内存限制: jvm 内存限制
> - 数据安全问题: 服务器宕机, 数据就消失了, 消息没有被消费就出现了不一致问题

消息队列, 顾名思义就是存放消息的对象, 其最基本的 3 个角色如下
- 消息队列: 存储和管理消息 -> 消息代理
- 生产者: 发送消息
- 消费者: 消费消息

> [!tip] 与原生的阻塞队列比较
> - 独立于 jvm 之外, 不受其内存限制
> - 会对消息进行持久化, 数据更加安全

### 基于 List 结构模拟消息队列
Redis的list数据结构是一个双向链表，很容易模拟出队列效果。队列是入口和出口不在一边，因此我们可以利用：LPUSH 结合 RPOP、或者 RPUSH 结合 LPOP来实现。

> [!warning]
> 不过要注意的是，当队列中没有消息时RPOP或LPOP操作会返回null，并不像JVM的阻塞队列那样会阻塞并等待消息。因此这里应该使用BRPOP或者BLPOP来实现阻塞效果。

#### 优缺点
优点：
- 利用Redis存储，不受限于JVM内存上限
- 基于Redis的持久化机制，数据安全性有保证
- 可以满足消息有序性
缺点：
- 无法避免消息丢失
- 只支持单消费者: 一个消息只能被一个消费者消费


### 基于 PubSub 的消息队列
PubSub (发布订阅) 是 Redis2.0 版本一如的消息传递模型. 顾名思义，消费者可以订阅一个或多个channel，生产者向对应channel发送消息后，所有订阅者都能收到相关消息。

- SUBSCRIBE channel [channel] ：订阅一个或多个频道
- PUBLISH channel msg ：向一个频道发送消息
- PSUBSCRIBE pattern[pattern] ：订阅与pattern格式匹配的所有频道

#### 优缺点
优点：
 - 采用发布订阅模型，支持多生产、多消费
缺点：
- 不支持数据持久化
- 无法避免消息丢失
- 消息堆积有上限，超出时数据丢失

### 基于 Stream 消息队列
Stream 是 Redis 5.0 引入的一种新数据类型，可以实现一个功能非常完善的消息队列。

#### 发送消息指令 - XADD
![[Pasted image 20241118092121.png]]

```bash
# 创建名为 users 的队列，并向其中发送一个消息，内容是：{name=jack,age=21}，并且使用Redis自动生成ID
XADD users * name jack age 21
# 返回消息 id
>> "1644805700523-0"
```

> [!question] `*` 表示什么
> 表示有 redis 自动生成 id

#### 读取消息指令 - XREAD
![[Pasted image 20241118092622.png]]
```bash
XREAD count 1 block 2000 streams userss 0
```

> [!warning] 使用 $ 的注意点
> 如果使用 $ 而一直又有新消息就会一直读取新消息, 会出现消息漏镀的情况

#### 特点
XREAD命令特点：
- 消息可回溯: 消息读取后不会消失, 永远保存
- 一个消息可以被多个消费者读取
- 可以阻塞读取
- 有消息漏读的风险


#### 消费者组
`消费者组`（Consumer Group）：将多个消费者划分到一个组中，监听同一个队列。具备下列特点：
- 消息分流: 分流给组内不同的消费者, 加快消息处理, 避免消息堆积
- 消息标识: 会维护一个标识, 记录最后一个处理的消息, 确保每一个消息都被消费
- 消息确认: 消费者获取消息后，消息处于pending状态，并存入一个pending-list。当处理完成后需要通过XACK来确认消息，标记消息为已处理，才会从pending-list移除。 避免了消息丢失问题

##### 消费者组指令
###### 创建消费者组指令
```bash
XGROUP CREATE  key groupName ID [MKSTREAM]
```

- key: 队列名称
- groupName: 消费者组名称
- ID: 起始 ID 标示, $ 代表队列中最后一个消息, 0 则代表队列中第一个消息
- MKSTREAM：队列不存在时自动创建队列

###### 消费消息指令
```bash
XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] ID [ID ...]
```

- group：消费组名称
- consumer：消费者名称，如果消费者不存在，会自动创建一个消费者
- count：本次查询的最大数量
- BLOCK milliseconds：当没有消息时最长等待时间
- NOACK：无需手动ACK，获取到消息后自动确认
- STREAMS key：指定队列名称
- ID：获取消息的起始ID：
	- ">"：从下一个未消费的消息开始
	- 其它：根据指定id从pending-list中获取已消费但未确认的消息，例如0，是从pending-list中的第一个消息开始

> [!tip] 确认消息指令
> ```bash
> xack 队列名名 组名 消息id...
> ```

###### 查看 pending-list
```bash
xpending key group [最小空闲时间] minId maxId count [consumer]
```

```bash
xpending s1 g1 - + 10
```

> [!question] `-` `+` 符号表示什么
> `-` 表示最小id
> `+` 表示最大id

###### 其他指令
```bash
# 删除指定的消费者组
XGROUP DESTORY key groupName

# 给指定的消费者组添加消费者
XGROUP CREATECONSUMER key groupname consumername

# 删除消费者组中的指定消费者
XGROUP DELCONSUMER key groupname consumername
```

##### 代码逻辑
![[Pasted image 20241120081855.png|600]]

##### 指令特点
- 消息可回溯
- 可以多消费者争抢消息，加快消费速度
- 可以阻塞读取
- 没有消息漏读的风险
- 有消息确认机制，保证消息至少被消费一次

### summary
![[Pasted image 20241120082338.png]]

## 基于 Redis 的 Stream 结构作为消息队列, 实现异步秒杀
- 创建一个Stream类型的消息队列，名为stream.orders
```bash
xgroup create stream.orders g1 0 mkstream
```
- 修改之前的秒杀下单Lua脚本，在认定有抢购资格后，直接向 stream.orders 中添加消息，内容包含voucherId、userId、orderId(这样就可以将java中的阻塞队列代码给删了)
 - 项目启动时，开启一个线程任务，尝试获取stream.orders中的消息，完成下单

### 读取消息代码
```java
// 1.获取消息队列中的订单信息 XREADGROUP GROUP g1 c1 COUNT 1 BLOCK 2000 STREAMS s1>
List<MapRecord<String, Object, Object>> list = stringRedisTemplate.opsForStream().read(  
        Consumer.from("g1", "c1"),  
        StreamReadOptions.empty().count(1).block(Duration.ofSeconds(2)),  
        StreamOffset.create(queueName, ReadOffset.lastConsumed()) // 读取最新消息 
);
```

### 解析消息代码
```java
MapRecord<String, Object, Object> record = list.get(0);  
Map<Object, Object> value = record.getValue();  
VoucherOrder voucherOrder = BeanUtil.fillBeanWithMap(value, new VoucherOrder(), true);
```


































