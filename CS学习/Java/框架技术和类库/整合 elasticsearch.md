## 通过 docker 安装
### 部署 elasticsearch
1. 下载 elasticsearch
```shell
docker pull elasticsearch:7.17.23
```
2. 创建启动容器
```bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx128m" -d elasticsearch:7.17.23
```
3. 复制目录到宿主机
```bash
docker cp elasticsearch:/usr/share/elasticsearch/config D:\docker\elasticsearch-7.17.23
docker cp elasticsearch:/usr/share/elasticsearch/data D:\docker\elasticsearch-7.17.23
docker cp elasticsearch:/usr/share/elasticsearch/plugins D:\docker\elasticsearch-7.17.23

```
4. 创建并启动容器
```bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx128m" -v D:\docker\elasticsearch-7.17.23\config:/usr/share/elasticsearch/config -v D:\docker\elasticsearch-7.17.23\data:/usr/share/elasticsearch/data -v D:\docker\elasticsearch-7.17.23\plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.17.23
```
5. 安装ik分词器
```bash
docker exec -it elasticsearch /bin/bash

./bin/elasticsearch-plugin install https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-7.17.23.zip
```



### 部署 kibana
1. 下载 kibana
```bash
docker pull kibana:7.17.23
```
2. 创建并启动容器
```bash
docker run --name kibana -p 5601:5601 -d kibana:7.17.23
```
3. 复制 config 文件夹中的文件到宿主机
```bash
docker cp kibana:/usr/share/kibana/config d:/docker/kibana/config
```
4. 挂载本地目录并启动
```bash
docker run --name kibana --privileged=true -p 5601:5601 -v d:/docker/kibana/config:/usr/share/kibana/config -d kibana:7.17.23
```
5. 修改配置文件后再重新启动
```properties
# 在 config 的配置文件上加上这一行
server.host: "0.0.0.0"
# elasticsearch 的地址页不能写成 localhost 否则监听不到, 通过
elasticsearch.hosts: ["http://172.28.128.1:9200"]
```

