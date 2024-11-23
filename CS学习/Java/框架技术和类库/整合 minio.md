## 坐标和配置文件

```xml
<!--minio-->  
<dependency>  
  <groupId>io.minio</groupId>  
  <artifactId>minio</artifactId>  
  <version>${minio.version}</version>  
</dependency>
```

```yml
minio:  
  endpoint: http://127.0.0.1:9000  
  accessKey: fancier  
  secretKey: hf128ve98a  
  bucketName: fancy.blog
```

## 配置类

### 读取配置文件
```java
@Data  
@ConfigurationProperties(prefix = "minio")  
@Component  
public class MinioProperties {  
    private String endpoint;  
    private String accessKey;  
    private String secretKey;  
    private String bucketName;  
}
```

### 注入 MinioClient
```java
@Data  
@Component  
@RequiredArgsConstructor  
public class MinioConfigure {  
  
    private final MinioProperties minioProperties;  
  
    /**  
     * 注入 minio 客户端  
     * @return minio 客户端  
     */  
    @Bean  
    public MinioClient minioClient(){  
  
        return MinioClient.builder()  
                .endpoint(minioProperties.getEndpoint())  
                .credentials(minioProperties.getAccessKey(), minioProperties.getSecretKey())  
                .build();  
    }  
}
```

## 上传文件的方法
```java
public String upload(MultipartFile file) {  
    String fileName = file.getOriginalFilename();  
  
    Preconditions.checkArgument(StringUtils.isNotBlank(fileName), "文件名不能为空");  
  
    String[] split = fileName.split("\\.");  
    String url = "";  
    String uuid = UUID.randomUUID().toString();  
    if (split.length > 1) {  
        fileName = uuid + "-" + System.currentTimeMillis() + "." + split[1];  
    } else {  
        fileName = uuid + System.currentTimeMillis();  
    }  
    InputStream in = null;  
    try {  
        in = file.getInputStream();  
        minioClient.putObject(PutObjectArgs.builder()  
                .bucket(minioProperties.getBucketName())  
                .object(fileName)  
                .stream(in, in.available(), -1)  
                .contentType(file.getContentType())  
                .build()  
        );  
        url = Joiner.on("/")  
                .join(Arrays.asList(minioProperties.getEndpoint(), minioProperties.getBucketName(), fileName));  
    } catch (Exception e) {  
        log.info("===> Exception", e);  
    } finally {  
        if (in != null) {  
            try {  
                in.close();  
            } catch (IOException e) {  
                log.info("===> Exception", e);  
            }  
        }  
    }  
    return url;  
}
```