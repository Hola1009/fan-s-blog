## maven 仓库地址
仓库地址
```xml
<dependency>  
    <groupId>com.github.houbb</groupId>  
    <artifactId>sensitive-word</artifactId>  
    <version>0.20.0</version>  
</dependency>
```

## 核心方法
| 方法                                     | 参数                       | 返回值    | 说明           |
| :------------------------------------- | :----------------------- | :----- | :----------- |
| contains(String)                       | 待验证的字符串                  | 布尔值    | 验证字符串是否包含敏感词 |
| replace(String, ISensitiveWordReplace) | 使用指定的替换策略替换敏感词           | 字符串    | 返回脱敏后的字符串    |
| replace(String, char)                  | 使用指定的 char 替换敏感词         | 字符串    | 返回脱敏后的字符串    |
| replace(String)                        | 使用 `*` 替换敏感词             | 字符串    | 返回脱敏后的字符串    |
| findAll(String)                        | 待验证的字符串                  | 字符串列表  | 返回字符串中所有敏感词  |
| findFirst(String)                      | 待验证的字符串                  | 字符串    | 返回字符串中第一个敏感词 |
| findAll(String, IWordResultHandler)    | IWordResultHandler 结果处理类 | 字符串列表  | 返回字符串中所有敏感词  |
| findFirst(String, IWordResultHandler)  | IWordResultHandler 结果处理类 | 字符串    | 返回字符串中第一个敏感词 |
| tags(String)                           | 获取敏感词的标签                 | 敏感词字符串 | 返回敏感词的标签列表   |


### 列出我可能用到的
contains(String): 验证字符串是否包含敏感词
replace(String): 使用 `*` 替换敏感词

## IWordResultHandler 结果处理类
打个标, 要用到的时候再学习
IWordResultHandler 可以对敏感词的结果进行处理，允许用户自定义。
作为findAll 的入参
- WordResultHandlers.word(): findAll() 方法将返回敏感词字符串集
- WordResultHandlers.raw(): findAll() 方法将返回敏感词开始结束小标字符串集
- WordResultHandlers.wordTags(): 指定对应词的标签信息

## 配置说明

## 新增/删除 白名单

