## 部署 etcd
1. 拉取 etcd 镜像
```bash
docker pull bitnami/etcd
```

2. 启动 etcd 容器
```bash
docker run -p 2379:2379 -p 2380:2380 --restart=always --name etcd -e ALLOW_NONE_AUTHENTICATION=yes -d bitnami/etcd:3.5.15
```

## 部署 hotkey
1. 创建 Dockerfile
```Dockerfile
FROM openjdk:8-jdk-alpine
add worker-0.0.4-SNAPSHOT.jar  hotkey-worker.jar
ENTRYPOINT [ "java", "-jar", "worker-0.0.4-SNAPSHOT.jar", "--etcd.server=127.0.0.1:2379"]
```

2. 构建 Docker 镜像
```bash
docker build -t hotkey-worker D:\JetBrains
```


