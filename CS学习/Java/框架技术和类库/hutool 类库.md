> [! info] 用到了就会记录下来
## maven 坐标
```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.4.3</version>
</dependency>
```
## API
![[Pasted image 20241015075805.png]]
### 类型转换
1. 转整型
```java
String param = "10";
int paramInt = Convert.toInt(param);
int paramIntDefault = Convert.toInt(param, 0);
```

2. 转日期
```java
String dateStr = "2020年09月29日";
Date date = Convert.toDate(dateStr);
```

### 日期时间
```java
Date date = DateUtil.date();
```
这个方法返回的实际运行类型是 DateTime, 重写了 toString 方法, 返回 yyyy-MM-dd HH:mm:ss 的字符串

字符串转日期
```java
String dateStr = "2020-09-29";
Date date = DateUtil.parse(dateStr);
```

### 扩展 HashMap
```java
Dict dict = Dict.create()
        .set("age", 18)
        .set("name", "沉默王二")
        .set("birthday", DateTime.now());

int age = dict.getInt("age");
String name = dict.getStr("name");
```

### 字段验证器
Validator


### 双向查找Map
BiMap

### 图片工具 ImgUtil
可以给图片加水印

### RandomUtil.randomNumbers(6)
用来生成指定位数的随机数字

## JSONUtil
### JSONUtil.toJsonStr(list)
将列表转成string字符串, 像下面这样
```json
["JavaScript", "基础"]
```

### JSONUtil.toList
接收两个参数, 一个是 JSONArray 对象, 一个是 类对象
JSONUtil.toList 方法会遍历 JSONArray 中的每个元素, 然后将它们转为指定类型, 然后添加到一个列表中, 最后返回
```java
JSONUtil.toList(JSONUtil.parseArray(question.getTags()), String.class)
```

