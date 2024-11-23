## Nacos镜像
```bash
docker run -d -p 8848:8848 -p 9848:9848 --name nacos-2.2.0 -e MODE=standalone -e TIME_ZONE='Asia/Shanghai' nacos/nacos-server:v2.2.0
```

## 挂载目录
创建 `D:/docker/nacos/conf` 和 `D:/docker/nacos/logs` 这两个文件夹
将容器中的文件复制到本地
```bash
docker cp nacos-2.2.0:/home/nacos/logs/ D:/docker/nacos/
docker cp nacos-2.2.0:/home/nacos/conf/ D:/docker/nacos/
```
把刚才的容器删掉, 再起一个容器
```bash
docker run -d --name nacos-2.2.0 -p 8848:8848  -p 9848:9848 -e MODE=standalone -e TIME_ZONE='Asia/Shanghai' -v D:/docker/nacos/logs/:/home/nacos/logs -v D:/docker/nacos/conf/:/home/nacos/conf/ nacos/nacos-server:v2.2.0
```

持久化将在配置文件中配置下mysql
```properties
db.url.0=jdbc:mysql://172.28.128.1:3307/pmhub-nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC

db.user=root
# 此为示例密码，请根据你的实际密码进行修改
db.password=123456
```
注意 url 中的ip要写成本机的IP地址, 不要写成 localhost 或 127.0.0.1



