## 导入坐标
```xml
<dependency>  
  <groupId>com.github.xiaoymin</groupId>  
  <artifactId>knife4j-openapi3-spring-boot-starter</artifactId>  
  <version>4.4.0</version>  
</dependency>
```

## 配置文件
```yml
springdoc:  
  api-docs:  
    path: /v3/api-docs  
  group-configs:  
    - group: '项目后台'  
      paths-to-match: '/**'  
      packages-to-scan: com.fancier.fancyblog.admin  
    - group: '项目前台'  
      paths-to-match: '/**'  
      packages-to-scan: com.fancier.fancyblog.web  
    - group: '校验鉴权'  
      paths-to-match: '/**'  
      packages-to-scan: com.fancier.fancyblog
```

