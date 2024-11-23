不多说\
```xml
<!--mapstruct编译-->  
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct-processor</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct</artifactId>  
</dependency>
```

不多说
```java
@Mapper  
public interface UserInfoConverter {  
    UserInfoConverter INSTANCE = Mappers.getMapper(UserInfoConverter.class);  
  
    /**  
     * 用户DO 转 用户信息VO  
     * @param userDO  
     * @return  
     */  
    UserInfoRspVO DO2UserInfoRspVO(UserDO userDO);  
}
```

## 属性名映射
用法
```java
@Mappings({  
        @Mapping(target = "value", source = "id"),  
        @Mapping(target = "label", source = "name")  
})
```

## 自定义映射规则
```java
@Mappings({  
        @Mapping(target = "isTop", expression = "java(mapValue(articleDO.getWeight()))")  
})

default boolean mapValue(int value) {  
    if (value > 0) {  
        return true;  
    } else if (value == 0) {  
        return false;  
    }  
    // 可以添加额外的逻辑，比如处理负数  
    return false; // 默认返回 false 或者其他逻辑  
}

```

## 设置默认信息
```java
@Mapping(target = "longProperty", source = "longProp", defaultValue = "-1")
```