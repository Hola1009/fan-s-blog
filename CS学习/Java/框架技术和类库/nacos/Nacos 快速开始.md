## 启动服务
### Linux/Unix/Mac
启动命令(standalone代表着单机模式运行，非集群模式):
`sh startup.sh -m standalone`
如果您使用的是ubuntu系统，或者运行脚本报错提示 符号找不到，可尝试如下运行：
`bash startup.sh -m standalone`

### Windows
启动命令(standalone代表着单机模式运行，非集群模式):
`startup.cmd -m standalone`

## Nacos Spring 快速开始
### 启动配置管理
启动了 Nacos server 后，您就可以参考以下示例代码，为您的 Spring 应用启动 Nacos 配置管理服务了。
1. 添加依赖
```xml
<dependency>  
    <groupId>com.alibaba.nacos</groupId>  
    <artifactId>nacos-spring-context</artifactId>  
    <version>${latest.version}</version>  
</dependency>
```

2. 添加 `@EnableNacosConfig` 注解启用 Nacos Spring 的配置管理服务。以下示例中，我们使用 `@NacosPropertySource` 加载了 `dataId` 为 `example` 的配置源，并开启自动更新：

```java
@Configuration  
@EnableNacosConfig(globalProperties = @NacosProperties(serverAddr = "127.0.0.1:8848"))  
@NacosPropertySource(dataId = "example", autoRefreshed = true)  
public class NacosConfiguration {  
  
}
```

3. 通过 Nacos 的 `@NacosValue` 注解设置属性值。
```java
@Controller  
@RequestMapping("config")  
public class ConfigController {  
  
    @NacosValue(value = "${useLocalCache:false}", autoRefreshed = true)  
    private boolean useLocalCache;  
  
    @RequestMapping(value = "/get", method = GET)  
    @ResponseBody  
    public boolean get() {  
        return useLocalCache;  
    }  
}
```

4. 通过调用 [Nacos Open API](https://nacos.io/zh-cn/docs/open-api.html) 向 Nacos Server 发布配置：dataId 为`example`，内容为`useLocalCache=true`
```bash
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=example&group=DEFAULT_GROUP&content=useLocalCache=true"
```

5. 访问 `http://localhost:8080/config/get`，此时返回内容为`true`，说明程序中的`useLocalCache`值已经被动态更新了。

## Nacos Spring Boot 快速开始
### 启动配置管理
启动了 Nacos server 后，您就可以参考以下示例代码，为您的 Spring Boot 应用启动 Nacos 配置管理服务了。\
1. 添加依赖
```xml
<dependency>  
    <groupId>com.alibaba.boot</groupId>  
    <artifactId>nacos-config-spring-boot-starter</artifactId>  
    <version>${latest.version}</version>  
</dependency>
```

> [!warning] 
> 版本 [0.2.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.boot/nacos-config-spring-boot-starter) 对应的是 Spring Boot 2.x 版本，版本 [0.1.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.boot/nacos-config-spring-boot-starter) 对应的是 Spring Boot 1.x 版本。

2. 在 `application.properties` 中配置 Nacos server 的地址：
```properties
nacos.config.server-addr=127.0.0.1:8848
```

3. 使用 `@NacosPropertySource` 加载 `dataId` 为 `example` 的配置源，并开启自动更新：
```java
@SpringBootApplication  
@NacosPropertySource(dataId = "example", autoRefreshed = true)  
public class NacosConfigApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(NacosConfigApplication.class, args);  
    }  
}
```

4. 通过 Nacos 的 `@NacosValue` 注解设置属性值。
```java
@Controller  
@RequestMapping("config")  
public class ConfigController {  
  
    @NacosValue(value = "${useLocalCache:false}", autoRefreshed = true)  
    private boolean useLocalCache;  
  
    @RequestMapping(value = "/get", method = GET)  
    @ResponseBody  
    public boolean get() {  
        return useLocalCache;  
    }  
}
```

5. 启动 `NacosConfigApplication`，调用 `curl http://localhost:8080/config/get`，返回内容是 `false`。
```bash
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=example&group=DEFAULT_GROUP&content=useLocalCache=true"
```

### 启动服务发现
本节演示如何在您的 Spring Boot 项目中启动 Nacos 的服务发现功能。

1. 添加依赖
```xml
<dependency>  
    <groupId>com.alibaba.boot</groupId>  
    <artifactId>nacos-discovery-spring-boot-starter</artifactId>  
    <version>${latest.version}</version>  
</dependency>
```

2. 在 `application.properties` 中配置 Nacos server 的地址：
```properties
nacos.discovery.server-addr=127.0.0.1:8848
```

3. 使用 `@NacosInjected` 注入 Nacos 的 `NamingService` 实例：