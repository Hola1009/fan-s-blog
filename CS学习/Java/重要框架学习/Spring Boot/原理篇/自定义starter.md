## 案例: 统计独立IP访问次数
业务 - 加上一个坐标后可以完成如下功能 
1. 每次访问网站行为均为进行统计
2. 后台每10秒输出一次统计信息(格式: IP + 访问次数)
而且可以在配置类里去, 修改配置

### 需求分析
1. 数据记录位置: Map/Redis
2. 功能触发位置: 每次 web 请求(拦截器)
	1) 步骤一: 降低难度, 主动调用, 仅统计单一操作访问次数
	2) 步骤二: 开发拦截器
3. 业务参数(配置项)
	1) 输出频度, 默认10秒
	2) 输出特征: 累计数据 / 极端数据, 默认累计数据
	3) 输出格式: 详细模式 / 极简模式

  



## 自定义starter
新开一个模块
![[Pasted image 20240526231514.png|left|200]]

1. Service 业务层 写 具体的实现
2. autoConfiguration 自动配置类用来 导入第三方bean
3. 在 spring.factories 里写自动配置类的路径


完成业务代码的编写, 再 clean 和 install 一下
![[Pasted image 20240526230757.png|left|200]]

再导包
![[Pasted image 20240526230921.png|left|400]]

在 @Inport 注入配置类

然后就可以开始使用了, 我是在自定义接口切面, 来调用的


属性设置, 弄一个类来读取配置文件中的属性
```java
@Data  
@Component("ipProperties")  
@ConfigurationProperties(prefix = "tools.ip")  
public class IpProperties {  
    /**  
     * 日志显示周期  
     */  
    private Long cycle = 5L;  
    /**  
     * 是否周期内重置数据  
     */  
    private Boolean cycleReset = false;  
    /**  
     * 日志输出模式  
     */  
    private String model = LogModel.DETAIL.value;  
  
  
    public enum LogModel {  
        DETAIL("detail"),  
        SIMPLE("simple");  
        private String value;  
        LogModel(String value) {  
            this.value = value;  
        }  
  
        public String getValue() {  
            return value;  
        }  
    }  
}
```


cron 表达式这样写  
```java
@Scheduled(cron = "*/#{ipProperties.cycle} * * * * ?")
```


### 用拦截器来搭配

```java
public class IpCountInterceptor implements HandlerInterceptor {  
  
    @Autowired  
    private IpCountService ipCountService;  
  
    @Override  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
        ipCountService.count();  
        return true;  
    }  
}
```

注册拦截器
```java
@Override  
public void addInterceptors(InterceptorRegistry registry) {  
    registry.addInterceptor(ipCountInterceptor()).addPathPatterns("/**");  
}  
  
@Bean  
public IpCountInterceptor ipCountInterceptor() {  
    return new IpCountInterceptor();  
}
```

## 辅助功能开发
