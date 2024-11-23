> [! info] 用到了就会记录下来

## 使用 Guava Preconditions 进行参数校验
### 引言
一般咱使用 if 做参数校验的代码是下面这样的, 这样3行代码看起来不够清爽和优雅, 而且如果要校验多个参数, 代码行数堆起来就很难看, 为了解决这个问题就可以使用 Guava Preconditions 
```Java
// 校验入参验证码是否为空 
if (StringUtils.isBlank(verificationCode)) { 
	return Response.fail(ResponseCodeEnum.PARAM_NOT_VALID.getErrorCode(), "验证码不能为空"); 
}
```

### 使用步骤
1. 导入包后就可以写出这样的代码, 第一个参数为校验条件, 第二个参数是异常提醒
```java
Preconditions.checkArgument(StringUtils.isNotBlank(verificationCode), "验证码不能为空");
```

2. 如果校验失败会抛出异常, 所以咱们还得显式捕获一下异常
```java
/**  
 * 捕获 guava 参数校验异常  
 */  
@ExceptionHandler({ IllegalArgumentException.class })  
@ResponseBody  
public Response<Object> handleIllegalArgumentException(HttpServletRequest request, IllegalArgumentException e) {  
    // 参数错误异常码  
    String errorCode = ResponseCodeEnum.PARAM_NOT_VALID.getErrorCode();  
  
    // 错误信息  
    String errorMessage = e.getMessage();  
  
    log.warn("{} request error, errorCode: {}, errorMessage: {}", request.getRequestURI(), rrorCode, errorMessage);  
  
    return Response.fail(errorCode, errorMessage);  
}
```

## 使用 Guava String utilities
这个我用到的地方

```java
List<ObjectError> allErrors = e.getBindingResult().getAllErrors();  
StringBuilder builder = new StringBuilder();  
String message = Joiner.on("; ").join(allErrors.stream()  
            .map(ObjectError::getDefaultMessage)  
        .collect(Collectors.toList()));
```


## ImmutableMap
用一种更简便的方式来构建并初始化map
```java
res.add(ImmutableMap.of(  
	"id",document.get("id"),  
	"title", toHtmlHighlight(query, "title", document.get("title"), analyzer),  
	"cover", document.get("cover"),  
	"createTime", document.get("createTime")  
));
```

