## 短信登录
### 基于 Session 登录
![[Pasted image 20240520153947.png]]


#### 登录验证功能

![[Pasted image 20240520194412.png|left|600]]


实现登录校验功能拦截器
![[Pasted image 20240520200322.png|left|600]]


![[Pasted image 20240520200350.png|left|400]]

### 集群的 session 共享问题
session 共享问题:  多台Tomcat并不共享session存储空间，当请求切换到不同tomcat服务时导致数据丢失的问题。

![[Pasted image 20240520201638.png|left|500]]

session 拷贝存在的问题: 消耗内存, 拷贝需要时间

session 的替代方案应该满足的需求:
- 数据共享
- 内存存储
- key  value 结构
### 基于 Redis 实现共享 session 登录
![[Pasted image 20240520202424.png|left|500]]



![[Pasted image 20240520203201.png]]

为什么不能用 手机号 来 代替 token
因为 相比于 token 手机号相对来说就透明许多, 那么只要其他人拿到手机号就可以进行登录了

#### 实现
保存验证码:
```java
// 4. 保存验证码  
stringRedisTemplate.opsForValue().set(RedisConstants.LOGIN_CODE_KEY + phone, code, 2, TimeUnit.MINUTES);
```

#编码技巧/将bean转成map
保存用户信息
```java
// 7. 保存用户信息到 redis 中  
// 7.1 随机生成token, 作为登录令  
String token = UUID.randomUUID().toString(true);  
// 7.2 将 User 对象转成 Hash 存储  
UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);  
Map<String, Object> beanToMap = BeanUtil.beanToMap(userDTO);  
// 7.3 存储  
String tokenKey = RedisConstants.LOGIN_USER_KEY + token;  
stringRedisTemplate.opsForHash().putAll(tokenKey + token, beanToMap);
```

#redis/redis设置有效期
给 tocken 设置有效期
```java
// 7.4 设置有效期  
stringRedisTemplate.expire(tokenKey, RedisConstants.LOGIN_USER_TTL, TimeUnit.MINUTES);
```

如果用户不断访问我们就要不断更新有效期:
使用拦截器, 在拦截器获取 请求头中的 token 数据, 然后通过token 从redis 拿到数据去构造 userDTO, 经过判断后就可以

#编码技巧/将map转成bean 
```java
UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);
```

存在的问题: StringRedisTemplate 要求 k 与 v 都是 String 类型
#编码技巧/将bean转成map
```java
Map<String, Object> beanToMap = BeanUtil.beanToMap(userDTO, new HashMap<>(),  
	CopyOptions.create()
		.setIgnoreNullValue(true) // 忽略空值
		.setFieldValueEditor((fieldKey, fieldValue) -> fieldValue.toString()) // 转换规则  
);
```

##### 拦截器登录优化
![[Pasted image 20240601225325.png]]

#拦截器/控制拦截器的执行顺序
默认执行顺序序号都为0 按照添加顺序执行
执行序号越小, 执行级别越高 
```java
public void addInterceptors(InterceptorRegistry registry) {  
    registry.addInterceptor(new LoginInterceptor())  
            .excludePathPatterns("/user/code",  
                    "/user/login",  
                    "/shop/**",  
                    "/shop-type/**",  
                    "/blog/hot",  
                    "/update/**",  
                    "/voucher/**"  
                    ).order(1);  
    registry.addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate)).order(0);  
}
```


## 商户查询缓存
### 什么是缓存
缓存就是数据交换的缓冲区（称作Cache [ kæʃ ] ），是存贮数据的临时地方，一般读写性能较高。

![[Pasted image 20240601231529.png]]
缓存作用
- 降低后端负载
 - 提高读写效率，降低响应时间
缓存成本
- 数据一致性成本
- 代码维护成本
- 运维成本
### 添加Redis缓存

![[Pasted image 20240601233232.png|400]]

![[Pasted image 20240601233330.png|400]]


### 缓存更新策略
![[Pasted image 20240601235044.png]]

业务场景：
- 低一致性需求：使用内存淘汰机制。例如店铺类型的查询缓存
- 高一致性需求：主动更新，并以超时剔除作为兜底方案。例如店铺详情查询的缓存

![[Pasted image 20240602204711.png]]
02: 编码复杂
03: 优点:效率高, 缺点: 维护复杂, 一致性差, 可靠性不高

操作缓存和数据库时有三个问题需要考虑：
1.删除缓存还是更新缓存？
- 更新缓存：每次更新数据库都更新缓存，无效写操作较多  × (可能存在写很多读很少的情况, 那样无效操作太多)
- 删除缓存：更新数据库时让缓存失效，`查询时再更新缓存`  √

2.如何保证缓存与数据库的操作的同时成功或失败？
- 单体系统，将缓存与数据库操作放在一个事务
- 分布式系统，利用TCC等分布式事务方案

3.先操作缓存还是先操作数据库？
- 先删除缓存，再操作数据库
  ![[Pasted image 20240602231144.png|异常情况|300]]
- 先操作数据库，再删除缓存
 
![[Pasted image 20240602231634.png|异常情况|300]]
方案二 
正常: 写数据库之间插了个查询, 没关系
异常: 可能性低, 因为正常情况写缓存 一定比写数据库快


 总结
缓存更新策略的最佳实践方案：
1.低一致性需求：使用Redis自带的内存淘汰机制
2.高一致性需求：主动更新，并以超时剔除作为兜底方案
- 读操作：
	- 缓存命中则直接返回
	- 缓存未命中则查询数据库，并写入缓存，设定超时时间
- 写操作：
	- 先写数据库，然后再删除缓存
	- 要确保数据库与缓存操作的原子性
#### 给查询商铺的缓存添加超时剔除和主动更新的策略
修改ShopController中的业务逻辑，满足下面的需求：
- 根据id查询店铺时，如果缓存未命中，则查询数据库，将数据库结果写入缓存，并设置超时时间
- 根据id修改店铺时，先修改数据库，再删除缓存

### 缓存穿透
缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。
如果查询的数据不存在, 那么必会查询数据库, 造成数据库资源浪费

解决方法
缓存空值:   
![[Pasted image 20240602234514.png|300]]
- 优点: 实现简单
- 缺点: 
	- 额外的内存消耗
	- 可能造成短期的不一致: 控制 TTL 时间/新增时主动新增
布隆过滤:
![[Pasted image 20240602234855.png|300]]
l布隆过
- 优点：内存占用较少，没有多余key
- 缺点：
	- 实现复杂
	- 存在误判可能原理: 

```java
@Override  
public Result queryById(Long id) {  
    String key = RedisConstants.CACHE_SHOP_KEY + id;  
    // 1. 从redis 查询商铺缓存  
    ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();  
    String shopJson = operations.get(RedisConstants.CACHE_SHOP_KEY + id);  
    // 2. 判断是否存在  
    if (StrUtil.isNotBlank(shopJson)) {  
        // 3. 如果存在, 直接返回  
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);  
        return Result.ok(shop);  
    }  
    if (shopJson != null) {  
        return Result.fail("店铺信息不存在");  
    }  
    
    Shop shop = getById(id);  
    if (shop == null) {  
        // 将空值写入 redis
		operations.set(key, "",CACHE_NULL_TTL, TimeUnit.MINUTES);  
        // 5. 不存在, 返回错误  
        return Result.fail("店铺不存在!");  
    }  
    // 6. 查询成功, 放入缓存  
    String value = JSONUtil.toJsonStr(shopJson);                      
    operations.set(key, value, RedisConstants.CACHE_SHOP_TTL, TimeUnit.MINUTES);  
  
    return Result.ok(shop);  
}
```
#### 总结
缓存穿透产生的原因是什么？
- 用户请求的数据在缓存中和数据库中都不存在，不断发起这样的请求，给数据库带来巨大压力
缓存穿透的解决方案有哪些？
- 缓存null值
- 布隆过滤
- 增强id的复杂度，避免被猜测id规律
- 做好数据的基础格式校验
- 加强用户权限校验
- 做好热点参数的限流
### 缓存雪崩
缓存雪崩是指在同一时段大量的缓存key同时失效或者Redis服务宕机，导致大量请求到达数据库，带来巨大压力。
解决方案：
- 给不同的Key的TTL添加随机值
- 利用Redis集群提高服务的可用性
- 给缓存业务添加降级限流策略
- 给业务添加多级缓存
![[Pasted image 20240613104637.png]]



### 缓存击穿 
缓存击穿问题也叫热点Key问题，就是一个被高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。
常见的解决方案有两种：
- 互斥锁
- 逻辑过期

![[Pasted image 20240613173526.png]]


#### 基于互斥锁方式解决缓存击穿问题
```java
@Override  
public Result queryById(Long id) {  
    String key = RedisConstants.CACHE_SHOP_KEY + id;  
    // 1. 从redis 查询商铺缓存  
    ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();  
    String shopJson = operations.get(RedisConstants.CACHE_SHOP_KEY + id);  
    // 2. 判断是否存在  
    if (StrUtil.isNotBlank(shopJson)) {  
        // 3. 如果存在, 直接返回  
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);  
        return Result.ok(shop);  
    }  
    if (shopJson != null) {  
        return Result.fail("店铺信息不存在");  
    }  
    // 4. 如果没有, 从数据库查询  
    // 4.1 获取互斥锁  
    String lockKey = "lock:shop" + id;  
    Shop shop = null;  
    try {  
  
        boolean isLock = tryLock(lockKey);  
        // 4.2 判断是否获取成功  
        if (!isLock) {  
            // 4.3 失败, 则休眠并重试  
            Thread.sleep(50);  
            queryById(id);  
        }  
        // 4.4 成功, 根据 id 查询数据库  
        shop = getById(id);  
        // 模式重建的延时  
        Thread.sleep(100);  
        // 4.5 判断是否存在  
        if (shop == null) {  
            // 将空值写入 redis
            // 防止缓存穿透            
            operations.set(key, "",CACHE_NULL_TTL, TimeUnit.MINUTES);  
            // 5. 不存在, 返回错误  
            return Result.fail("店铺不存在!");  
        }
        // 6. 查询成功, 放入缓存  
        String value = ;  
        operations.set(key, value, CACHE_NULL_TTL, TimeUnit.MINUTES);    
    } catch (InterruptedException e) {  
        throw new RuntimeException(e);  
    } finally {  
        unLock(lockKey);  
    }  
  
    return Result.ok(shop);  
}
```

互斥锁
![[Pasted image 20240613173639.png]]

缓存击穿
![[Pasted image 20240613173717.png]]

#### 基于逻辑过期方式解决缓存击穿问题
基于互斥锁不同的是 未命中 不会 去查询数据库, 因为, 设置了逻辑过期时间, 理论上 不会把数据从数据库中删除
但这种方法我认为与 与先查再缓存的缓存方式 之间矛盾, 
![[Pasted image 20240613175308.png]]
```java
@Override  
public Result queryById(Long id) {  
    String key = RedisConstants.CACHE_SHOP_KEY + id;  
    // 1. 从redis 查询商铺缓存  
    ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();  
    String shopJson = operations.get(RedisConstants.CACHE_SHOP_KEY + id);  
    // 2. 判断是否存在  
    if (StrUtil.isBlank(shopJson)) {  
        // 3. 如果存在, 直接返回  
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);  
        return Result.ok(shop);  
    }  
  
    // 4. 命中 , 需要把json反序列化成对象  
  
    // 4. 如果没有, 从数据库查询  
  
    RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);  
    Shop shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class);  
    LocalDateTime expireTime = redisData.getExpireTime();  
  
    // 5 判断是否过期  
    if (expireTime.isAfter(LocalDateTime.now())) {  
        // 5.1 过期, 直接返回  
        return Result.ok(shop);  
    }  
    // 5.2 已经过期, 需要重新入缓存  
  
    // 6 缓存重建  
    // 6.1 获取互斥锁  
    String lockKey = LOCK_SHOP_KEY + id;  
    boolean isLock = tryLock(lockKey);  
    // 6.2 判断是否获取成功  
    if (isLock) {  
        // 6.3 成功, 开启独立线程, 实现缓存重建  
        CACHE_REBUILD_EXECUTOR.submit(() -> {  
            try {  
                this.saveShop2Redis(id, 20L);  
            } catch (Exception e) {  
                throw new RuntimeException(e);  
            } finally {  
                unLock(lockKey);  
            }  
        });  
    }  
    return Result.ok(shop);  
}
```

```java
private void saveShop2Redis(Long shopId, Long expireSeconds) {  
    // 1. 查询店铺数据  
    Shop shop = getById(shopId);  
    // 2. 封装逻辑过期时间  
    RedisData redisData = new RedisData();  
    redisData.setData(shop);  
    redisData.setExpireTime(LocalDateTime.now().plusSeconds(expireSeconds));  
    // 3. 写入redis  
    stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + shopId, JSONUtil.toJsonStr(redisData));  
}
```

### 缓存工具封装
基于StringRedisTemplate封装一个缓存工具类，满足下列需求：
- 方法1：将任意Java对象序列化为json并存储在string类型的key中，并且可以设置TTL过期时间
- 方法2：将任意Java对象序列化为json并存储在string类型的key中，并且可以设置逻辑过期时间，用于处理缓存击穿问题
- 方法3：根据指定的key查询缓存，并反序列化为指定类型，利用缓存空值的方式解决缓存穿透问题
- 方法4：根据指定的key查询缓存，并反序列化为指定类型，需要利用逻辑过期解决缓存击穿问题
```java
@Component  
@Slf4j  
@RequiredArgsConstructor  
public class CacheClient {  
    private final StringRedisTemplate stringRedisTemplate;  
    private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);  
  
  
    /**  
     * 封装 set json 字符串 方法  
     * @param key  
     * @param value  
     * @param time  
     * @param unit  
     */  
    public void set(String key, Object value, Long time, TimeUnit unit) {  
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(value), time, unit);  
    }  
  
    /**  
     * 基于逻辑过期解决缓存击穿  
     * @param key  
     * @param value  
     * @param time  
     * @param unit  
     */  
    public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit unit) {  
        // 设置逻辑过期对象  
        RedisData redisData = new RedisData();  
        redisData.setData(value);  
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(unit.toSeconds(time)));  
        // 写入 Redis        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(redisData));  
    }  
  
    /**  
     * 解决缓存穿透问题  
     * @param keyPrefix  
     * @param id  
     * @param type  
     * @param dbFallback  
     * @param time  
     * @param unit  
     * @return  
     * @param <R>  
     * @param <ID>  
     */  
    public <R, ID> R queryWithThrough(String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {  
        String key = keyPrefix + id;  
        // 1. 从redis 查询商铺缓存  
        ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();  
        String json = operations.get(key);  
        // 2. 判断是否存在  
        if (StrUtil.isNotBlank(json)) {  
            // 3. 如果存在, 直接返回  
             return JSONUtil.toBean(json, type);  
        }  
  
        // json == null 说明没有这个 key        // 缓存穿透解决  
        if (json != null) {  
            // 返回一个错误信息 key 的 value 不存在  
            return null;  
        }  
  
        R resource = dbFallback.apply(id);  
  
        if (resource == null) {  
            // 将空值写入 redis            operations.set(key, "",CACHE_NULL_TTL, TimeUnit.MINUTES);  
            // 5. 不存在, 返回错误  
            return null;  
        }  
        // 6. 查询成功, 放入缓存  
        this.set(key, resource, time, unit);  
  
        return resource;  
    }  
  
    /**  
     * 解决缓存击穿问题  
     * 当 key 失效时 只重置一次缓存  
     * @param keyPrefix  
     * @param id  
     * @param type  
     * @param dbFallback  
     * @param time  
     * @param unit  
     * @return  
     * @param <R>  
     * @param <ID>  
     */  
    public <R, ID> R queryWithLogicalExpire(String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {  
        String key = RedisConstants.CACHE_SHOP_KEY + id;  
        // 1. 从redis 查询商铺缓存  
        ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();  
        String json = operations.get(RedisConstants.CACHE_SHOP_KEY + id);  
        // 2. 判断是否存在  
        if (StrUtil.isBlank(json)) {  
            // 未命中  
            // 3. 如果存在, 直接返回  
            return null;  
        }  
  
        // 4. 命中 , 需要把json反序列化成对象  
        RedisData redisData = JSONUtil.toBean(json, RedisData.class);  
        R resource = JSONUtil.toBean((JSONObject) redisData.getData(), type);  
        LocalDateTime expireTime = redisData.getExpireTime();  
  
        // 5 判断是否过期  
        if (expireTime.isAfter(LocalDateTime.now())) {  
            // 5.1 过期, 直接返回  
            return resource;  
        }  
        // 5.2 已经过期, 需要重新入缓存  
  
        // 6 缓存重建  
        // 6.1 获取互斥锁  
        String lockKey = LOCK_SHOP_KEY + id;  
        boolean isLock = tryLock(lockKey);  
        // 6.2 判断是否获取成功  
        if (isLock) {  
            // 6.3 成功, 开启独立线程, 实现缓存重建  
            CACHE_REBUILD_EXECUTOR.submit(() -> {  
                try {  
                    R newResource = dbFallback.apply(id);  
                    // 写入redis  
                    this.setWithLogicalExpire(key, newResource, time, unit);  
                } catch (Exception e) {  
                    throw new RuntimeException(e);  
                } finally {  
                    unLock(lockKey);  
                }  
            });  
        }  
        return resource;  
    }  
  
    /**  
     * 获取锁, 用 setnx 方法 如果没有 就创建 创建成功后, 就等于枷锁了, 其他线程就创建不了了, 除非删除  
     * @param key  
     * @return  
     */  
    private boolean tryLock(String key) {  
        return Boolean.TRUE.equals(stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.MINUTES));  
    }  
  
    /**  
     *释放锁  
     * @param key  
     */  
    private void unLock(String key) {  
        stringRedisTemplate.delete(key);  
    }  
}
```


## 优惠券秒杀
### 全局唯-ID
每个店铺都可以发布优惠券：
![[Pasted image 20240614062823.png]]

当用户抢购时，就会生成订单并保存到tb_voucher_order这张表中，而订单表如果使用数据库自增ID就存在一些问题：
- id的规律性太明显
- 受单表数据量的限制

#### 全局 id 生成器
全局ID生成器，是一种在分布式系统下用来生成全局唯一ID的工具，一般要满足下列特性：
![[Pasted image 20240614063325.png]]

为了增加ID的安全性，我们可以不直接使用Redis自增的数值，而是拼接一些其它信息：
![[Pasted image 20240614063821.png]]
ID的组成部分：
- 符号位：1bit，永远为0
- 时间戳：31bit，以秒为单位，可以使用69年
- 序列号：32bit，秒内的计数器，支持每秒产生2^32个不同ID
```java
public Long nextId(String keyPrefix) {  
    // 1. 生成时间戳  
    LocalDateTime now = LocalDateTime.now();  
    long nowSecond = now.toEpochSecond(ZoneOffset.UTC);  
    Long timeStamp = nowSecond - BEGIN_TIME;  
  
    // 2. 生成序列号  
    // 2.1 获取当前日期, 精确到天  
    String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));  
    // 2.2 自增长  
    Long count = stringRedisTemplate.opsForValue().increment("icr:" + keyPrefix + ":" + date);  
    // 3. 拼接并返回  
    return timeStamp << COUNT_BIT | count;  
}
```

全局唯一ID生成策略：
- UUID
- Redis自增
- snowflake算法
- 数据库自增
Redis自增ID策略：
- 每天一个key，方便统计订单量
- ID构造是 时间戳 + 计数器
### 实现优惠券秒杀下单

每个店铺都可以发布优惠券，分为平价券和特价券。平价券可以任意购买，而特价券需要秒杀抢购：
![[Pasted image 20240614111014.png]]

表关系如下：
- tb_voucher：优惠券的基本信息，优惠金额、使用规则等
- tb_seckill_voucher：优惠券的库存、开始抢购时间，结束抢购时间。特价优惠券才需要填写这些信息
```mysql
create table tb_voucher  
(  
    id           bigint unsigned auto_increment comment '主键'  
        primary key,  
    shop_id      bigint unsigned                               null comment '商铺id',  
    title        varchar(255)                                  not null comment '代金券标题',  
    sub_title    varchar(255)                                  null comment '副标题',  
    rules        varchar(1024)                                 null comment '使用规则',  
    pay_value    bigint(10) unsigned                           not null comment '支付金额，单位是分。例如200代表2元',  
    actual_value bigint(10)                                    not null comment '抵扣金额，单位是分。例如200代表2元',  
    type         tinyint(1) unsigned default 0                 not null comment '0,普通券；1,秒杀券',  
    status       tinyint(1) unsigned default 1                 not null comment '1,上架; 2,下架; 3,过期',  
    create_time  timestamp           default CURRENT_TIMESTAMP not null comment '创建时间',  
    update_time  timestamp           default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'  
)
```

```mysql
create table tb_voucher_order  
(  
    id          bigint                                        not null comment '主键'  
        primary key,  
    user_id     bigint unsigned                               not null comment '下单的用户id',  
    voucher_id  bigint unsigned                               not null comment '购买的代金券id',  
    pay_type    tinyint(1) unsigned default 1                 not null comment '支付方式 1：余额支付；2：支付宝；3：微信',  
    status      tinyint(1) unsigned default 1                 not null comment '订单状态，1：未支付；2：已支付；3：已核销；4：已取消；5：退款中；6：已退款',  
    create_time timestamp           default CURRENT_TIMESTAMP not null comment '下单时间',  
    pay_time    timestamp                                     null comment '支付时间',  
    use_time    timestamp                                     null comment '核销时间',  
    refund_time timestamp                                     null comment '退款时间',  
    update_time timestamp           default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'  
)
```

下单时需要判断两点：
- 秒杀是否开始或结束，如果尚未开始或已经结束则无法下单
- 库存是否充足，不足则无法下单

### 超卖问题
正常情况
![[Pasted image 20240614120924.png|300]]

异常情况
![[Pasted image 20240614121022.png|500]]


超卖问题是典型的多线程安全问题，针对这一问题的常见解决方案就是加锁：
![[Pasted image 20240614121058.png]]

#### 乐观锁
乐观锁的关键是判断之前查询得到的数据**是否有被修改过**，常见的方式有两种：
- CAS 法 把 stock 当做库存
![[Pasted image 20240614161302.png|600]]

如果基于 数据 变更来判断 成功率 会太低
所以, 第二次查就 判断 stock 大于 0 即可

### 一人一单
#想法加在简历上
需求：修改秒杀业务，要求同一个优惠券，一个用户只能下一单
防止黄牛 重复 买卷

![[Pasted image 20240614184406.png]]
但是 这种 请况 还是会出现超卖问题, 假如 10 个线程同一时刻, 都查到了0, 就会插入 10 个订单

解决方法给 从查询订单开始的代码加锁

不能在方法上 加 synchronized 因为 这样就是 锁对象就是 this 那么 所有请求 整个就是一个串行执行了

因为 我们 是 的业务是一人一单, 也就是说对同一个用户不能线程的相同请求来加锁, 所以 锁对象 应该 弄成 用户id
 因为 每次用户请求进来 进行用户id查询都会得到一个新的 UserId  所以我们将用户id给toString() , 但是 的话, toString 底层也是 新创建一个对象, 所以要调用 intern() 方法, 到常量池去拿 String , 这样就确定了 同一个用户 拿到的都是同一把锁
```java
userId.toString().intern()
```
```java
@Transactional // 事务是在方法执行后才提交的
public Result createVoucherOrder(Long voucherId, Integer stock) {  
    // 5. 校验 一人一单  
    // 5.1 获取用户id  
    Long userId = UserHolder.getUser().getId();  
  
    synchronized (userId.toString().intern()) {  
        // 5.2 查询是否存在订单  
        Integer cnt = query()  // 如果事务提交慢一步, 就算更新了, 也拿到的更新之间的值. 即产生线程安全问题
                .eq("user_id", userId)  
                .eq("voucher_id", voucherId).count();  
        if (cnt > 0) {  
            // 用户已经购买过了  
            return Result.fail("用户已经购买过一次了");  
        }  
  
        // 6. 扣减库存  
        Voucher voucher = new Voucher();  
  
        voucher.setStock(stock - 1);  
        int count = voucherMapper.update(voucher, Wrappers.<Voucher>lambdaUpdate()  
                .eq(Voucher::getId, voucherId)  
                .gt(Voucher::getStock, 0)  
        );  
  
        if (count <= 0) {  
            // 扣减失败  
            return Result.fail("库存不足");  
        }  
        // 7. 创建订单并保存  
        VoucherOrder voucherOrder = new VoucherOrder();  
        // 7.1 订单 id        Long orderId = redisIDWorker.nextId("order");  
        voucherOrder.setId(orderId);  
  
        voucherOrder.setUserId(userId);  
        // 7.3 代金卷 id        voucherOrder.setVoucherId(voucherId);  
        save(voucherOrder);  
  
        return Result.ok(orderId);  
    }  
}
```

又来一个细节问题, 那就是 锁是在代码块执行完毕后释放的, 事务是在方法执行完毕后由spring提交的, 所以在事务提交 之前锁还是会被其他线程拿到, 所以我们应该在整个事务提交之后再释放锁,  那就是 把给在调用方法处给的代码枷锁, (所以 业务代码必须分离, 方便理解) 

(这里我学到一个之前没有注意到的知识点, 那就是: 判断是否进行操作的逻辑应该写在锁住代码块里面)



```java
Long userId = UserHolder.getUser().getId();  
// 8. 返回结果  
synchronized (userId.toString().intern()) {  
    return createVoucherOrder(voucherId, userId, stock);  
}
```

但是又来了一个问题, 那就是,  createVoucherOrder() 调用时走的是 this.createVoucherOrder(), 并不是代理对象的 createVoucherOrder() 方法, 所以这里调用 这个方法是不具备事务功能的, 导致事务失效

所以 我们 必须拿到 代理的对象, 然后给他, 而且还要在接口里声明这个方法
#拿到代理对象
```java
Long userId = UserHolder.getUser().getId();  
// 8. 返回结果  
synchronized (userId.toString().intern()) {  
	
    return createVoucherOrder(voucherId, userId, stock);  
}
```

这样做 底层会用到一个 AspectJ 的这样一个依赖
```xml
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
</dependency>
```

还需要在启动类 添加 注解来暴露代理对象
```java
@EnableAspectJAutoProxy(exposeProxy = true)  
public class HmDianPingApplication {}
```

#### 一人一单的并发安全问题
通过加锁可以解决在单机情况下的一人一单安全问题，但是在集群模式下就不行了。
1. 我们将服务启动两份，端口分别为8081和8082：
![[Pasted image 20240615100913.png]]
2. 然后修改nginx的conf目录下的nginx.conf文件，配置反向代理和负载均衡
![[Pasted image 20240615101011.png]]
现在，用户请求会在这两个节点上负载均衡，再次测试下是否存在线程安全问题。

因为之前在单机上运行是拿常量池中的String 作为锁对象, 但是 分布式 不同机器上有不同的常量池, 所以加锁对其他机器上的进程是无效的
![[Pasted image 20240615102240.png]]
### 分布式锁
![[Pasted image 20240615102727.png]]

这样 就可以使多个jvm的进程都可以看到同一把锁 

#### 什么是分布式锁
分布式锁：满足分布式系统或集群模式下多进程可见并且互斥的锁
![[Pasted image 20240615102818.png]]

#### 分布式锁的实现
![[Pasted image 20240615103240.png]]
#### 基于 Redis 的分布式锁
实现分布式锁时需要实现的两个基本方法：
- 获取锁：
	- 互斥：确保只能有一个线程获取锁
	- 非阻塞：尝试一次，成功返回true，失败返回false
```bash
# 添加锁, 利用setnx的互斥特性
setnx lock thread 1
# 添加过期时间, 避免服务宕机形成死锁
expire lock 10
```
- 释放锁：
	- 手动释放
	- 超时释放: 避免服务宕机, 没法释放锁, 处于死锁状态
```bash
del key
```

为避免 还未添加超时时间 服务就宕机, 需要让 加锁和添加超时时间同步进行  
用如下语句来代替 把 nx 和 ex 编程原子操作
```bash
set lock thread1 ex 10 nx
```
![[Pasted image 20240615112342.png|300]]

##### 基于 redis 实现分布式锁初级版本
需求：定义一个类，实现下面接口，利用Redis实现分布式锁功能。
![[Pasted image 20240615112833.png]]

```java
public interface ILock {  
  
    /**  
     * 尝试获取锁  
     * @param timeout 锁持有的超时时间, 过期自动释放锁  
     * @return true 获取锁成功  
     */  
    boolean tryLock(Long timeout);  
  
    /**  
     * 释放锁   
*/  
    void unlock();  
}
```

```java
public class SimpleRedisLock implements ILock {  
    private String name;  
    private StringRedisTemplate stringRedisTemplate;  
    private static final String KEY_PREFIX = "lock:";  
  
    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {  
        this.name = name;  
        this.stringRedisTemplate = stringRedisTemplate;  
    }  
  
    /**  
     * 获取锁  
     * @param timeout 锁持有的超时时间, 过期自动释放锁  
     * @return  
     */  
    @Override  
    public boolean tryLock(Long timeout) {  
        // 获取线程的id  
        long threadId = Thread.currentThread().getId();  
        // 获取锁  
        return Boolean.TRUE.equals(stringRedisTemplate.opsForValue().setIfAbsent(KEY_PREFIX + name, threadId + "", timeout, TimeUnit.SECONDS));  
  
    }  
  
    /**  
     * 释放锁  
     */  
    @Override  
    public void unlock() {  
        stringRedisTemplate.delete(KEY_PREFIX + name);  
    }  
}
```

使用
```java
SimpleRedisLock lock = new SimpleRedisLock("order" + userId, stringRedisTemplate);  
// 获取代理对象 (事务)  
boolean isLock = lock.tryLock(1200L);  
  
if (isLock) {  
    return Result.fail("不允许重复下单");  
}
```

![[Pasted image 20240615201509.png]]


如果 一个线程在, 业务完成之前, 锁超时释放后, 被其他线程拿到了, 那在其业务完成后去释放锁时, 释放的将是其他线程的锁解决 方法, 给 锁设置一个唯一标识, 怎么设置, 用 UUID + threadId

需求：修改之前的分布式锁实现，满足：
1. 在获取锁时存入线程标示（可以用UUID表示）
2. 在释放锁时先获取锁中的线程标示，判断是否与当前线程标示一致
	- 如果一致则释放锁
	- 如果不一致则不释放锁

```java
public class SimpleRedisLock implements ILock {  
    private String name;  
  
    private final StringRedisTemplate stringRedisTemplate;  
  
    private static final String ID_PREFIX = UUID.randomUUID().toString(true) + "-";  
    private static final String KEY_PREFIX = "lock:";  
  
    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {  
        this.name = name;  
        this.stringRedisTemplate = stringRedisTemplate;  
    }  
  
    /**  
     * 获取锁  
     * @param timeout 锁持有的超时时间, 过期自动释放锁  
     * @return  
     */  
    @Override  
    public boolean tryLock(Long timeout) {  
        // 获取线程的id  
        String threadId = ID_PREFIX + Thread.currentThread().getId();  
        // 获取锁  
        return Boolean.TRUE.equals(stringRedisTemplate.opsForValue().setIfAbsent(KEY_PREFIX + name, threadId, timeout, TimeUnit.SECONDS));  
  
    }  
  
    /**  
     * 释放锁  
     */  
    @Override  
    public void unlock() {  
        String threadId = ID_PREFIX + Thread.currentThread().getId();  
        String id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);  
        if (threadId.equals(id)) {  
            stringRedisTemplate.delete(KEY_PREFIX + name);  
        }  
    }  
  
}
```

但是 `判断锁标识` 和 `释放锁` 是两个动作 不具备**原子性**, 所以 如果在这之间发生了阻塞, 锁被超时释放了, 其他线程乘虚而入了, 那么阻塞结束后进行释放锁, 就会把其他人的锁给释放掉了

所以需要保证这两个动作的原子性
#Lua脚本
#### Redis 的 Lua 脚本
Redis提供了Lua脚本功能，在一个脚本中编写多条Redis命令，确保多条命令执行时的原子性。Lua是一种编程语言，它的基本语法大家可以参考网站：[https://www.runoob.com/lua/lua-tutorial.html](https://www.runoob.com/lua/lua-tutorial.html)
这里重点介绍Redis提供的调用函数，语法如下：
```lua
# 执行redis命令
redis.call('命令名称', 'key', '其它参数', ...)
```
例如，我们要执行set name jack，则脚本是这样：
```lua
# 执行 set name jack
redis.call('set', 'name', 'jack')
```
例如，我们要先执行set name Rose，再执行get name，则脚本如下：
```lua
# 先执行 set name jack
redis.call('set', 'name', 'jack')
# 再执行 get name
local name = redis.call('get', 'name')
# 返回
return name
```

#### 再次改进 Redis 的分布式锁逻辑
![[Pasted image 20240615220752.png]]

释放锁的业务流程是这样的：
1. 获取锁中的线程标示
2. 判断是否与指定的标示（当前线程标示）一致
3. 如果一致则释放锁（删除）
4. 如果不一致则什么都不做
如果用Lua脚本来表示则是这样的
```lua
-- 这里的 KEYS[1] 就是锁的key，这里的ARGV[1] 就是当前线程标示
-- 获取锁中的标示，判断是否与当前线程标示一致
if (redis.call('GET', KEYS[1]) == ARGV[1]) then
  -- 一致，则删除锁
  return redis.call('DEL', KEYS[1])
end
-- 不一致，则直接返回
return 0
```
：
![[Pasted image 20240615221030.png]]

建议把脚本 写在文件当中

#执行lua脚本
```java
/**  
 * lua脚本  
 */  
private static final DefaultRedisScript<Long> UNLOCK_SCRIPT;  
  
static {  
    UNLOCK_SCRIPT = new DefaultRedisScript<>();  
    UNLOCK_SCRIPT.setLocation(new ClassPathResource("unlock.lua"));  
    UNLOCK_SCRIPT.setResultType(Long.class);  
}

@Override  
public void unlock() {  
    // 调用 lua 脚本  
    Long execute = stringRedisTemplate.execute(UNLOCK_SCRIPT,  
            Collections.singletonList(KEY_PREFIX + name),  
            Collections.singletonList(ID_PREFIX + Thread.currentThread().getId()));  
}
```

#### 基于Redis 的分布式锁优化

![[Pasted image 20240616155023.png]]
1. 比如方法A中获取了锁 后 去掉方法B 这是时候方法B 也需要这把锁, 这就会导致 死锁

#### Redisson
Redisson是一个在Redis的基础上实现的Java驻内存数据网格（In-Memory Data Grid）。它不仅提供了一系列的分布式的Java常用对象，还提供了许多分布式服务，其中就包含了各种分布式锁的实现。
![[Pasted image 20240616160056.png]]
##### Redission 快速入门

1. 引入依赖
```xml
<dependency>  
    <groupId>org.redisson</groupId>  
    <artifactId>redisson</artifactId>  
    <version>3.13.6</version>  
</dependency>
```
2. 配置 Redission 客户端
```java
@Controller  
public class RedisConfig {  
  
    @Bean  
    public RedissonClient redisClient() {  
        // 配置类  
        Config config = new Config();  
        // 添加 ip 地址  
        config.useSingleServer()
	        .setAddress("redis://196.168.200.137:6379")
	        .setAddress("123456");  
        // 创建客户端  
        return Redisson.create(config);  
    }  
}
```

```java
//SimpleRedisLock lock = new SimpleRedisLock("order" + userId, stringRedisTemplate);  
RLock lock = redissonClient.getLock("lock:order");
```

#### Redisson 可重入锁原理

除了要保存 线程唯一标识 还要保存 加锁的层数, 如此需要在一个key下保存两个字段, 我们需要换一种数据类型即 hash 
![[Pasted image 20240616202611.png]]

具体流程我描述一遍, 进来先判断 当前线程的锁是否存在 如果存在就 + 1 如果不存在就 创建后+ 1, 并重置锁的有效期, 执行完业务后, 需要先判断锁是否是自己的如果是就 -1 减到 0 了 就释放锁, 没有减到 就刷新有效期

lua 脚本
获取锁
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
    redis.call('expire', key, releaseTime);    
    return 1; -- 返回结果  
end;  
return 0; -- 代码走到这里,说明获取锁的不是自己，获取锁失败
```

释放锁
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
if (count > 0) then    -- 大于0说明不能释放锁，重置有效期然后返回    
	redis.call('EXPIRE', key, releaseTime);
	return nil;  
else  -- 等于0说明可以释放锁，直接删除    
	redis.call('DEL', key);    
	return nil;  
end;
```
#### Redisson分布式锁原理
##### 解决不可重试 和 超时释放问题
Redisson 的 trylock 的3个参数
如果要 设置重试 需要传 timeWait 这个参数 leaseTime 可以有默认值
```java
public boolean tryLock(long waitTime, TimeUnit unit) throws InterruptedException {  
    return this.tryLock(waitTime, -1L, unit);  
}
```

```java
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {  
    long time = unit.toMillis(waitTime);  
    long current = System.currentTimeMillis();  
    long threadId = Thread.currentThread().getId();  
    // 如果没有后取到锁, 则返回 被人的锁的有效时长
    Long ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);  
    if (ttl == null) { //为空说明获取到了锁   
        return true;  
    } else {  
	    // 看看时间是否有剩余
        time -= System.currentTimeMillis() - current;  
        if (time <= 0L) {  
            this.acquireFailed(waitTime, unit, threadId);  
            return false;  
        } else {  // 有剩余就继续尝试
            current = System.currentTimeMillis();  
            // 订阅释放锁信号
            RFuture<RedissonLockEntry> subscribeFuture = this.subscribe(threadId);  
            // 如果在剩余时间内没获取到锁 -> 就放弃
            if (!subscribeFuture.await(time, TimeUnit.MILLISECONDS)) {  
                if (!subscribeFuture.cancel(false)) {  
                    subscribeFuture.onComplete((res, e) -> {  
                        if (e == null) {  
                            this.unsubscribe(subscribeFuture, threadId);  
                        }
                    });  
                } 
                this.acquireFailed(waitTime, unit, threadId);  
                return false;  
            } else {  
                try {  
                    time -= System.currentTimeMillis() - current;  
                    if (time <= 0L) {  
                        this.acquireFailed(waitTime, unit, threadId);  
                        boolean var20 = false;  
                        return var20;  
                    } else {  
                        boolean var16;  
                        do {  
                            long currentTime = System.currentTimeMillis();  
                            // 重试
                            ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);  
                            if (ttl == null) {  
                                var16 = true;  
                                return var16;  
                            }  
							
                            time -= System.currentTimeMillis() - currentTime;  
                            if (time <= 0L) {  
                                this.acquireFailed(waitTime, unit, threadId);  
                                var16 = false;  
                                return var16;  
                            }  
  
                            currentTime = System.currentTimeMillis();  
                            if (ttl >= 0L && ttl < time) { // 取小的时间 
                                ((RedissonLockEntry)subscribeFuture.getNow()).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);  // 
                            } else {  
                                ((RedissonLockEntry)subscribeFuture.getNow()).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);  
                            }  
  
                            time -= System.currentTimeMillis() - currentTime;  
                        } while(time > 0L); // 时间充足继续尝试 
  
                        this.acquireFailed(waitTime, unit, threadId);  
                        var16 = false;  
                        return var16;  
                    }  
                } finally {  
                    this.unsubscribe(subscribeFuture, threadId);  
                }  
            }  
        }  
    }  
}
```

根据 对前面这段代码的分析, 又引出了一个问题, 如果我因为阻塞锁到期了, 那么根据这个逻辑, 其他人就可以在我业务没完成之前, 就拿到锁了, 所以我该怎么保证让我的锁在业务完成后再释放呢?

##### 解决超时释放存在的安全隐患

```java
private RFuture<Boolean> tryAcquireOnceAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId) {  
    if (leaseTime != -1L) {  
        return this.tryLockInnerAsync(waitTime, leaseTime, unit, threadId, RedisCommands.EVAL_NULL_BOOLEAN);  
    } else {  
        RFuture<Boolean> ttlRemainingFuture = this.tryLockInnerAsync(waitTime, this.commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_NULL_BOOLEAN);  
        ttlRemainingFuture.onComplete((ttlRemaining, e) -> {  
            if (e == null) {  
                if (ttlRemaining == null) {
	                // 如果获取锁成功了  就续约超时时间
                    this.scheduleExpirationRenewal(threadId);  
                }  
  
            }  
        });  
        return ttlRemainingFuture;  
    }  
}
```

```java
private void scheduleExpirationRenewal(long threadId) {  
    ExpirationEntry entry = new ExpirationEntry();  
    // 往map里放 entry map保证同一把锁拿到的都是同一个entry(可以理解为锁的名称)
    ExpirationEntry oldEntry = (ExpirationEntry)EXPIRATION_RENEWAL_MAP.putIfAbsent(this.getEntryName(), entry);  
    if (oldEntry != null) {  //第一次来
        oldEntry.addThreadId(threadId);  
    } else {  // 重入的时候
        entry.addThreadId(threadId);  
        this.renewExpiration();  
    }  
  
}
```

// 更新有效期
```java
private void renewExpiration() {  
    ExpirationEntry ee = (ExpirationEntry)EXPIRATION_RENEWAL_MAP.get(this.getEntryName());  
    if (ee != null) {  
    // 添加一个延时任务 在 10 秒后执行(看门狗超时的1/3)
        Timeout task = this.commandExecutor.getConnectionManager().newTimeout(new TimerTask() {  
            public void run(Timeout timeout) throws Exception {  
                ExpirationEntry ent = (ExpirationEntry)RedissonLock.EXPIRATION_RENEWAL_MAP.get(RedissonLock.this.getEntryName());  
                if (ent != null) {  
                    Long threadId = ent.getFirstThreadId();  
                    if (threadId != null) { 
                    // 刷新有效期 
                        RFuture<Boolean> future = RedissonLock.this.renewExpirationAsync(threadId);  
                        future.onComplete((res, e) -> {  
                            if (e != null) {  
                                RedissonLock.log.error("Can't update lock " + RedissonLock.this.getName() + " expiration", e);  
                            } else {  
                                if (res) {  
                                // 递归执行 永不过期
                                    RedissonLock.this.renewExpiration();  
                                }  
                            }  
                        });  
                    }  
                }  
            }  
        }, this.internalLockLeaseTime / 3L, TimeUnit.MILLISECONDS);
        // 把任务设置到 entry 中, 方便后续 从map中 拿到entry 后去结束任务  
        ee.setTimeout(task);  
    }  
}
```

取消更新任务
```java
public RFuture<Void> unlockAsync(long threadId) {  
    RPromise<Void> result = new RedissonPromise();  
    RFuture<Boolean> future = this.unlockInnerAsync(threadId);  
    future.onComplete((opStatus, e) -> {
	    //取消更新任务 
        this.cancelExpirationRenewal(threadId);  
        if (e != null) {  
            result.tryFailure(e);  
        } else if (opStatus == null) {  
            IllegalMonitorStateException cause = new IllegalMonitorStateException("attempt to unlock lock, not locked by current thread by node id: " + this.id + " thread-id: " + threadId);  
            result.tryFailure(cause);  
        } else {  
            result.trySuccess((Object)null);  
        }  
    });  
    return result;  
}
```

```java
void cancelExpirationRenewal(Long threadId) {  
    ExpirationEntry task = (ExpirationEntry)EXPIRATION_RENEWAL_MAP.get(this.getEntryName());  
    if (task != null) {  
        if (threadId != null) {  
            task.removeThreadId(threadId);  
        }  
  
        if (threadId == null || task.hasNoThreads()) {  
            Timeout timeout = task.getTimeout();  
            if (timeout != null) {  
                timeout.cancel();  
            }  
  
            EXPIRATION_RENEWAL_MAP.remove(this.getEntryName());  
        }  
  
    }  
}
```

用自己的语言梳理一遍
首先 我的线程会去尝试获取锁, 如果没有获取到会返回, 一个锁的剩余有效时间, 以此为依据在等待时间没有耗尽之前就可以去等待锁的释放了重复尝试获取锁了, 如果获取锁成功了, 是第一次获取就会把锁的标识放进一个 expiration_renewal_map 中, 是重入的话就 开启一个延时任务来不断刷新锁的演示时间, 知道释放锁的时候把这个延时任务给关了

![[Pasted image 20240617172941.png]]

Redisson分布式锁原理：
- 可重入：利用hash结构记录线程id和重入次数
- 可重试：利用信号量和PubSub功能实现等待、唤醒，获取锁失败的重试机制
- 超时续约：利用watchDog，每隔一段时间（releaseTime / 3），重置超时时间

#### Redisson分布式锁主从一致性问题
![[Pasted image 20240618162438.png]]

![[Pasted image 20240618162452.png]]


使用联锁的方式解决主从一致性问题

![[Pasted image 20240618162525.png]]

Redisson 联锁的使用

```java
@Controller  
public class RedisConfig {  
  
    @Bean  
    public RedissonClient redisClient() {  
        // 配置类  
        Config config = new Config();  
        // 添加 ip 地址  
        config.useSingleServer().setAddress("redis://196.168.200.137:6379").setAddress("123456");  
        // 创建客户端  
        return Redisson.create(config);  
    }  
}
```

首先在 配置文件里 多添加几个 RedissonClient 的 Bean
然后  去 配置 联锁
```java
RLock lock1 = redissonClient1.getLock("order");  
RLock lock2 = redissonClient2.getLock("order");  
RLock lock3 = redissonClient3.getLock("order");  
// 调哪个个 getMultiLock 的方法都一样
lock = redissonClient1.getMultiLock(lock1, lock2, lock3);
```

RedissonMultiLock 的 tryLock() 方法
```java
@Override  
    public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {  
//        try {  
//            return tryLockAsync(waitTime, leaseTime, unit).get();  
//        } catch (ExecutionException e) {  
//            throw new IllegalStateException(e);  
//        }  
        long newLeaseTime = -1;  
        if (leaseTime != -1) {  // 判断有没有传释放时间
            if (waitTime == -1) {  
	            // 只执行一次
                newLeaseTime = unit.toMillis(leaseTime);   
            } else {  
	            // 用 wait 代替 lease 时间
                newLeaseTime = unit.toMillis(waitTime)*2;  
            }  
        }  
        
        long time = System.currentTimeMillis();  
        // 剩余时间
        long remainTime = -1;  
        if (waitTime != -1) {  
            remainTime = unit.toMillis(waitTime);  
        }  
        // 计算锁的等待时间
        long lockWaitTime = calcLockWaitTime(remainTime);  
        // 锁失败的限制是0
        int failedLocksLimit = failedLocksLimit();  
        // 保存获取成功的锁
        List<RLock> acquiredLocks = new ArrayList<>(locks.size());  
        // 遍历3个独立的锁
        for (ListIterator<RLock> iterator = locks.listIterator(); iterator.hasNext();) {  
            RLock lock = iterator.next();  
            boolean lockAcquired;  
            try {  
                if (waitTime == -1 && leaseTime == -1) { 
	                // 只尝试一次 , 不重试
                    lockAcquired = lock.tryLock();  
                } else {  
	                // 回去重试
                    long awaitTime = Math.min(lockWaitTime, remainTime); 
                    // 返回获取锁结果 
                    lockAcquired = lock.tryLock(awaitTime, newLeaseTime, TimeUnit.MILLISECONDS);  
                }  
            } catch (RedisResponseTimeoutException e) {  
                unlockInner(Arrays.asList(lock));  
                lockAcquired = false;  
            } catch (Exception e) {  
                lockAcquired = false;  
            }  
            // 判断获取锁是否成功
            if (lockAcquired) {  
	            // 放入已经成功的锁的集合
                acquiredLocks.add(lock);  
            } else {  // 获取锁失败
				// failedLocksLimit() 返回锁失败的上限, 默认为 0
	            // 判断 
                if (locks.size() - acquiredLocks.size() == failedLocksLimit()) {  
                    break;  
                }  
			  
                if (failedLocksLimit == 0) {  
                    unlockInner(acquiredLocks);  
                    if (waitTime == -1) {  // == -1 证明不想重试
                        return false;  
                    }  
                    failedLocksLimit = failedLocksLimit(); 
                    // 把锁全部清空 
                    acquiredLocks.clear();  
                    // 把迭代器往前去调整
                    while (iterator.hasPrevious()) {  
                        iterator.previous();  
                    }  
                } else {  
                    failedLocksLimit--;  
                }  
            }  
            // 如果剩余时间小于 0 
            if (remainTime != -1) {  
                remainTime -= System.currentTimeMillis() - time;  
                time = System.currentTimeMillis();  
                // 获取锁超时了
                if (remainTime <= 0) {
	                // 失败后 将已将获取的锁给释放  
                    unlockInner(acquiredLocks);  
                    return false;  
                }  
            }  
        }  
  
        if (leaseTime != -1) {  // 如果传了 lease 就不会 开始 看门狗任务
	        // 重新 设置 每一把锁的有效期, 因为每一把锁获取到的时间不一样, 所以再次需要统一设置, 保证同步
            List<RFuture<Boolean>> futures = new ArrayList<>(acquiredLocks.size());  
            for (RLock rLock : acquiredLocks) {  
                RFuture<Boolean> future = ((RedissonLock) rLock).expireAsync(unit.toMillis(leaseTime), TimeUnit.MILLISECONDS);  
                futures.add(future);  
            }  
            for (RFuture<Boolean> rFuture : futures) {  
                rFuture.syncUninterruptibly();  
            }  
        }  
        return true;  
    }
```

#### 总结
1）不可重入Redis分布式锁：
- 原理：利用setnx的互斥性；利用ex避免死锁；释放锁时判断线程标示
- 缺陷：不可重入、无法重试、锁超时失效
2）可重入的Redis分布式锁：
- 原理：利用hash结构，记录线程标示和重入次数；利用watchDog延续锁时间；利用信号量控制锁重试等待
- 缺陷：redis宕机引起锁失效问题
3）Redisson的multiLock：
- 原理：多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功
- 缺陷：运维成本高、实现复杂

### Redis优化秒杀
之前的秒杀逻辑
![[Pasted image 20240619085443.png]]

因为 操作数据库 的时间比较长 所以, 所以这套流程的效率极低,  所以 我们将 判断秒杀库存 和 校验一人一单 还有 返回订单号 这些业务给抽离出来 给 redis 来做, 而操作数据库 我们再新开一个线程来做, 这样效率就得到了极大的提高


![[Pasted image 20240619090054.png]]


#### 如何再 Redis 完成 库存判断 和 一人一单判断

库存 用 string 来存, 一人一单 用 set 来存
![[Pasted image 20240619090359.png]]


#### 改进秒杀业务, 提高并发性能
需求：
- 新增秒杀优惠券的同时，将优惠券信息保存到Redis中
```java
// 保存秒杀库存到 redis 当中  
stringRedisTemplate.opsForValue()
	.set(RedisConstants.SECKILL_STOCK_KEY + voucher.getId(), voucher.getStock().toString());
```
- 基于Lua脚本，判断秒杀库存、一人一单，决定用户是否抢购成功
```lua
-- 1. 参数列表  
-- 1.1. 优惠卷id  
local voucherId = ARGV[1]  
-- 1.2. 用户id  
local userId = ARGV[1]
  
-- 2. 数据 key
-- 2.1. 库存 key
local stockKey = 'seckill:stock' .. voucherId  
-- 2.2. 订单key  
local orderKey = 'seckill:order' .. voucherId  
  
-- 3. 脚本业务  
-- 3.1 判断库存是否充足 get stockKey
if (tonumber(redis.call('get', stockKey)) <= 0) then  
    -- 3.2. 库存不足, 返回1  
    return 1  
end  
  
-- 3.3 判断用户是否下单 SISMEMBER 判断是否是set中的成员  
if (redis.call('SISMEMBER', orderKey, userId) == 1) then  
    -- 存在说明是重复下单  
    return 2  
end  
  
-- 3.4 扣减库存  
redis.call('decr', stockKey, -1)  
-- 3.5 下单 (保存用户) sadd orderKey userId  
redis.call('sadd', orderKey, userId)
```

java 判断购买资格代码
```java
// 1. 执行 lua 脚本  
UserDTO userId = UserHolder.getUser();  
Long result = stringRedisTemplate.execute(SECKILL_SCRIPT,  
        Collections.emptyList(),  
        voucherId, userId.toString()  
);  
// 2. 判断结果是否为 0assert result != null;  
int res = result.intValue();  
// 2.1. 不为 0 没有购买资格  
if (res != 0) {  
    // 2.1. 不为 0 , 代表没有购买资格  
    return Result.fail(res == 1 ? "库存不足" : "不能重复下单");  
}
```
- 如果抢购成功，将优惠券id和用户id封装后存入阻塞队列 (异步下单的过程)
```java
// 2.2. 创建订单并保存  
// 爲什麽在這裏就創建 訂單, 因爲 方便封裝信息然後放入阻塞隊列中  
VoucherOrder voucherOrder = new VoucherOrder();  
voucherOrder.setId(orderId);  
voucherOrder.setUserId(userId);  
voucherOrder.setVoucherId(voucherId);  
// 2.2. 为 0 , 有购买资格, 把下单信息保存到阻塞队列  
orderTask.add(voucherOrder);
```


- 开启线程任务，不断从阻塞队列中获取信息，实现异步下单功能
```java
private void handleVoucherOrder(VoucherOrder voucherOrder) {  
  
  
    // 1. 获取用户id  
    Long userId = voucherOrder.getUserId();  
    Long voucherId = voucherOrder.getVoucherId();  
    // 2. 创建锁对象  
    RLock lock = redissonClient.getLock("lock:order");  
  
    // 3. 获取代理对象 (事务)  
    boolean isLock = lock.tryLock();  
  
    if (isLock) {  
        // 获取锁失败, 返回错误信息  
        log.error("不允许重复下单");  
        return;  
    }  
    try {  
        // 获取代理对象  
        // AopContext.currentProxy() 底层是通过 ThreadLocal 所以子线程调用是拿不到代理对象的  
        // IVoucherOrderService proxy = (IVoucherOrderService) (AopContext.currentProxy());  
        proxy.createVoucherOrder(voucherOrder);  
    } finally {  
        // 释放锁  
        lock.unlock();  
    }  
}
```

```java
@Transactional  
public void createVoucherOrder(VoucherOrder voucherOrder) {  
  
    Long voucherId = voucherOrder.getVoucherId();  
  
    Long userId = UserHolder.getUser().getId();  
      
    // 6. 扣减库存  
    boolean success = seckillVoucherService.update()  
            .setSql("stock = stock - 1")  
            .eq("voucher_id", voucherId)  
            .gt("stock", 0)  
            .update();  
      
    if (!success) {  
        throw new RuntimeException("扣減庫存失敗");  
        // 扣减失败  
    }  
  
    save(voucherOrder);  
  
}
```


秒殺業務的優化思路是什麽?
1. 先利用redis彎沉庫存餘量, 一人一單判斷, 完成搶單業務
2. 再將下單業務放入阻塞隊列, 利用獨立綫程異步下端
基於阻塞隊列的異步秒殺存在的問題
- 内存受到限制
- 數據安全問題

### Redis消息队列实现异步秒杀

消息队列（Message Queue），字面意思就是存放消息的队列。最简单的消息队列模型包括3个角色：
- 消息队列：存储和管理消息，也被称为消息代理（Message Broker）
- 生产者：发送消息到消息队列
- 消费者：从消息队列获取消息并处理消息

![[Pasted image 20240620221902.png]]
Redis提供了三种不同的方式来实现消息队列：
- list结构：基于List结构模拟消息队列
- PubSub：基本的点对点消息模型
- Stream：比较完善的消息队列模型

#### 基于List结构模拟消息队列
消息队列（Message Queue），字面意思就是存放消息的队列。而Redis的list数据结构是一个双向链表，很容易模拟出队列效果。

队列是入口和出口不在一边，因此我们可以利用：**LPUSH**结合 **RPOP**、或者 **RPUSH** 结合 **LPOP** 来实现。

不过要注意的是，当队列中没有消息时RPOP或LPOP操作会返回null，并不像JVM的阻塞队列那样会阻塞并等待消息。因此这里应该使用 **BRPOP** 或者 **BLPOP** 来实现阻塞效果。

基于List的消息队列有哪些优缺点？
优点：
- 利用Redis存储，不受限于JVM内存上限
- 基于Redis的持久化机制，数据安全性有保证
- 可以满足消息有序性
缺点：
- 无法避免消息丢失
- 只支持单消费者

#### 基于PubSub 的消息队列
PubSub（发布订阅）是Redis2.0版本引入的消息传递模型。顾名思义，消费者可以订阅一个或多个channel，生产者向对应channel发送消息后，所有订阅者都能收到相关消息。
- SUBSCRIBE channel [channel] ：订阅一个或多个频道
- PUBLISH channel msg ：向一个频道发送消息
- PSUBSCRIBE pattern[pattern] ：订阅与pattern格式匹配的所有频道
![[Pasted image 20240620222507.png]]

![[Pasted image 20240620222659.png]]

基于PubSub的消息队列有哪些优缺点？
优点：
- 采用发布订阅模型，支持多生产、多消费
缺点：
- 不支持数据持久化
- 无法避免消息丢失
- 消息堆积有上限，超出时数据丢失

#### 基于 Stream 的消息队列
Stream 是 Redis 5.0 引入的一种新数据类型，可以实现一个功能非常完善的消息队列。

发送消息的命令：
![[Pasted image 20240621054815.png]]
例如：
![[Pasted image 20240621054507.png]]

读取消息: xread
![[Pasted image 20240621055212.png]]
例如
![[Pasted image 20240621055227.png]]

XREAD阻塞方式，读取最新的消息：
![[Pasted image 20240621055723.png]]


![[Pasted image 20240621055745.png]]
STREAM类型消息队列的XREAD命令特点：
- 消息可回溯
- 一个消息可以被多个消费者读取
- 可以阻塞读取
- 有消息漏读的风险

#### 基于Stream的消息队列-消费者组
消费者组（Consumer Group）：将多个消费者划分到一个组中，监听同一个队列。具备下列特点：
![[Pasted image 20240621060306.png]]

创建消费者组：
![[Pasted image 20240621060649.png]]
- key：队列名称
- groupName：消费者组名称
- ID：起始ID标示，$代表队列中最后一个消息，0则代表队列中第一个消息
- MKSTREAM：队列不存在时自动创建队列
其它常见命令：
![[Pasted image 20240621060702.png]]

从消费者组读取消息:
![[Pasted image 20240621060910.png]]

- group：消费组名称
- consumer：消费者名称，如果消费者不存在，会自动创建一个消费者
- count：本次查询的最大数量
- BLOCK milliseconds：当没有消息时最长等待时间
- NOACK：无需手动ACK，获取到消息后自动确认
- STREAMS key：指定队列名称
- ID：获取消息的起始ID：
	- ">"：从下一个未消费的消息开始
	- 其它：根据指定id从pending-list中获取已消费但未确认的消息，例如0，是从pending-list中的第一个消息开始

如何确认一个消息
xack key group id..

怎么去查看 pending list (每个消费者都有一个 pending-list)
![[Pasted image 20240621062808.png]]

消费者监听的基本思路
![[Pasted image 20240621063741.png]]


STREAM类型消息队列的XREADGROUP命令特点：
- 消息可回溯
- 可以多消费者争抢消息，加快消费速度
- 可以阻塞读取
- 没有消息漏读的风险
- 有消息确认机制，保证消息至少被消费一次

#### 总结

![[Pasted image 20240621064424.png]]

### 基于Redis的Stream结构作为消息队列, 实现异步秒杀下单
-  创建一个Stream类型的消息队列，名为stream.orders
```bash
xgroup create stream.orders g1 0 mkstream 
```
- 修改之前的秒杀下单Lua脚本，在认定有抢购资格后，直接向stream.orders中添加消息，内容包含voucherId、userId、orderId

- 项目启动时，开启一个线程任务，尝试获取stream.orders中的消息，完成下单


## 达人探店
### 发布探店笔记
探店笔记类似点评网站的评价，往往是图文结合。对应的表有两个：
- tb_blog：探店笔记表，包含笔记中的标题、文字、图片等
- tb_blog_comments：其他用户对探店笔记的评价

![[Pasted image 20240625093055.png]]

### 实现查看探店笔记的接口

![[Pasted image 20240625093326.png]]


### 点赞功能
需求:
- 一个用户只能点赞一次, 再次点击则取消点击
- 如果当前用户已经点赞, 则点赞按钮高亮显示 (前端实现)

实现步骤：
① 给Blog类中添加一个isLike字段(无需修改数据库字段)，标示是否被当前用户点赞

② 修改点赞功能，利用Redis的set集合判断是否点赞过，未点赞过则点赞数+1，已点赞过则点赞数-1

③ 修改根据id查询Blog的业务，判断当前登录用户是否点赞过，赋值给isLike字段

④ 修改分页查询Blog业务，判断当前登录用户是否点赞过，赋值给isLike字段

### 点赞排行榜
在探店笔记的详情页面，应该把给该笔记点赞的人显示出来，比如最早点赞的TOP5，形成点赞排行榜：
![[Pasted image 20240625104718.png]]

#### 实现
![[Pasted image 20240625104809.png]]

首选 srotedSet

zscore z1 m1 查询分数 0 4 \\\\(排名)

## 好友关注
### 关注和取关
很简单, 弄一张表crud即可

### Feed 流模式
Feed流产品有两种常见模式：
Timeline：不做内容筛选，简单的按照内容发布时间排序，常用于好友或关注。例如朋友圈
- 优点：信息全面，不会有缺失。并且实现也相对简单
- 缺点：信息噪音较多，用户不一定感兴趣，内容获取效率低
智能排序：利用智能算法屏蔽掉违规的、用户不感兴趣的内容。推送用户感兴趣信息来吸引用户
- 优点：投喂用户感兴趣信息，用户粘度很高，容易沉迷
- 缺点：如果算法不精准，可能起到反作用
本例中的个人页面，是基于关注的好友来做Feed流，因此采用Timeline的模式。该模式的实现方案有三种：
①拉模式
②推模式
③推拉结合
### Feed 流的分页问题
![[Pasted image 20240626063240.png]]
### 基于推模式实现关注推送功能
①修改新增探店笔记的业务，在保存blog到数据库的同时，推送到粉丝的收件箱
②收件箱满足可以根据时间戳排序，必须用Redis的数据结构实现
③查询收件箱数据时，可以实现分页查询
```
// 1. 获取登录用户  
  
// 2. 保存探店笔记  
  
// 3. 查询笔记作者的所有粉丝  
  
// 4. 推送笔记id的所有粉丝 遍历推送 以 粉丝id 为 key 博客id 为value 时间戳 为 score 存进 sortedSet  
// 5. 返回id
```
### 基于推模式实现关注查询推送

需要实现动态分页
```redis
zrevrange z1 min max withscores
## 键 角标 角标 带上分数
```

按分数查
```
zrevrangebyscore z1 1000 0 withscores limit 0 3
key max min 带分数 偏移量 查几条
```

```
// 获取当前用户

// 获取收件箱 

// 非空判断

// 解析数据: blogId, score

// 根据id查询blog 要注意 博客要有序 order by field(id, 用字符串拼接的idstr) StrUtil.join(",", id)

// 封装并返回
```
## 附近的商户
### GEO 功能
```bash
## 添加
geoadd g1 xxx.xxxxxx xxx.xxxxxx name
## 计算距离
geodist g1 xxx xxx
## 搜索
geoserach g1 fromlonlat xxx.xxxxxx xxx.xxxxxx byradius 10 km withdist
```

## 用户签到

我本能将心向明月
## UV统计
