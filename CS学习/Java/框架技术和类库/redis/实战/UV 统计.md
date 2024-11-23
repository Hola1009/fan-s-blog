> [!tip]
> - UV: 全程 Unique Visitor, 也叫独立访问量, 是指通过互联网访问, 浏览这个网页的自然人, 1 天内同一个用户多次访问该网站, 只记录 1 次
> - PV: 全称 Page View, 也叫页面访问量或点击量, 用户每访问网站的一个页面, 记录 1 次 PV, 用户多次打开页面, 则记录多次PV. 往往用来衡量网站的流量

> [!question] 
> UV统计在服务端做会比较麻烦，*因为要判断该用户是否已经统计过了*，需要将统计过的用户信息保存。但是如果每个访问的用户都保存到 Redis 中，数据量会非常恐怖。

## HyperLogLog 用法
Hyperloglog(HLL)是从Loglog算法派生的概率算法，用于确定非常大的集合的基数，而不需要存储其所有值。相关算法原理大家可以参考：[https://juejin.cn/post/6844903785744056333#heading-0](https://juejin.cn/post/6844903785744056333)

Redis中的HLL是基于string结构实现的，单个HLL的内存永远小于16kb，内存占用低的令人发指！作为代价，其测量结果是概率性的，有小于0.81％的误差。不过对于UV统计来说，这完全可以忽略。

 - pfadd: 添加元素
```bash
pfadd key element [element...]
```

- pfcount: 统计
```bash
pfcount key [key...]
```

- pfmerge: 合并多个 key
```bash
pfmerge 
```

### Java API
- 添加
```java
stringRedisTemplate.opsForHyperLogLog().add(key, values);
```

- 统计
```java
stringRedisTemplate.opsForHyperLogLog().size(key);
```

