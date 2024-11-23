快速构建, 运行, 管理应用的工具

# 2.配置Docker的yum库

首先要安装一个yum工具

```Bash
yum install -y yum-utils
```

安装成功后，执行命令，配置Docker的yum源：

```Bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

# 3.安装Docker

最后，执行命令，安装Docker

```Bash
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

  

# 4.启动和校验

```Bash
# 启动Docker
systemctl start docker

# 停止Docker
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps
```



## 镜像和容器
当我们利用Docker安装应用时，Docker会自动搜索并下载应用镜像(image)。镜像不仅包含应用本身，还包含应用运行所需要的环境、配置、系统函数库。Docker会在运行镜像时创建一个隔离环境，称为容器(container)
比如实现 mysql 多开

镜像仓库: 存储和管理镜像平台, Docker 官方维护了一个公共仓库: Docker Hub


## docker 命令解读
```shell
docker run -d \   # 创建并运行一个容器, -d 是让容器在后台运行
  --name mysql \   # 给容器命名 
  -p 3306:3306 \   # 设置端口映射 (一个容器相当于一个虚拟机, 且对外隔离, 所以需要宿主的端口来和外界交互)
  -e TZ=Asia/Shanghai \   # -e KEY=VALUE 设置端口映射
  -e MYSQL_ROOT_PASSWORD=123 \ 
  mysql   # 指定运行镜像的名字 [repository]:[tag] [镜像名]:[镜像版本(默认是最新版)]
```

```bash
docker run -d \ 
--name mysql \ 
-p 3306:3306 \ 
-e TZ=Asia/Shanghai \ 
-e MYSQL_ROOT_PASSWORD=123456 \ 
-v /root/mysql/data:/var/lib/mysql \
-v /root/mysql/init:/docker-entrypoint-initdb.d \
-v /root/mysql/conf:/etc/mysql/conf.d \
mysql   
```

```bash
docker run -d --name mysql -p 3307:3306 -e TZ=Asia/Shanghai -e MYSQL_ROOT_PASSWORD=123456 -v /root/mysql/data:/var/lib/mysql -v /root/mysql/init:/docker-entrypoint-initdb.d -v /root/mysql/conf:/etc/mysql/conf.d mysql
```



![[Pasted image 20240529133858.png|200]]

前面一个是宿主机的端口



# Docker 基础知识

Docker 最常见名命令就是操作镜像, 容器的命令, 详见官方文档: 

![[Pasted image 20240529135347.png]]

### docker 打包 指令 
```shell
docker image save -o nginx.tar nginx:latest # 打包
docker load -i load output # 加载
```
### 进入 容器 
```bash
docker exec -it mysql bash
```
### 简化命令  命令别名
修改这个文件
```bash
vi ~/.bashrc
```

```shell
alias dps='docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"'
alias dis='docker images'
```

## 数据卷挂载

- 创建Nginx容器, 修改nginx容器内的html目录下的index.html文件, 查看变化
- 将静态资源部署到nginx的html目录下

但在容器内很难修改文件, 没有 vi 指令

解决方法:

数据卷 是一个虚拟目录, 是容器内部与宿主机目录之间映射的桥梁

![[Pasted image 20240529201345.png]]

docker 会实现 容器 与 宿主机的`双向绑定`

![[Pasted image 20240529201635.png|left|500]]


- 在执行 docker run  命令时, 使用 -v 数据卷:容器内目录 可以完成数据卷挂载
```bash
docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx
```
- 当创建容器时, 如果挂载了数据卷且数据卷不存在, 会自动创建数据卷


查看卷

```bash
docker volume ls
```

查看卷挂载信息

```bash
docker volume inspect html
```

删除 容器
```bash
docker rm -f nginx
```


### 第二种挂载方式
查看容器详情
```bash
docker inspect [容器名]

"Mounts": [
	{
		"Type": "volume",
		"Name": "html",
		"Source": "/var/lib/docker/volumes/html/_data",
		"Destination": "/usr/share/nginx/html",
		"Driver": "local",
		"Mode": "z",
		"RW": true,
		"Propagation": ""
	}
],

```

需求:
- 查看mysql容器, 判断是否由数据卷挂载
```bash
docker inspect mysql
"Mounts": [
	{
		"Type": "volume",
		"Name": "2bc79151c5fc9705ff81634f2a57d83b8ec7e9c462391ebd6235d2732d79bc6a",
		"Source": "/var/lib/docker/volumes/2bc79151c5fc9705ff816/_data", ## 匿名卷
		"Destination": "/var/lib/mysql",
		"Driver": "local",
		"Mode": "",
		"RW": true,
		"Propagation": ""
	}
],
```
删除容器后 会开一个新匿名卷, 这将涉及到拷贝, 比较麻烦, 所以最好自己弄个卷

- 基于宿主机目录实习MySql 数据目录, 配置文件, 初始化脚本的挂载
	- ① 挂载/root/mysql/data到容器内的/var/lib/mysql目录
	- ② 挂载/root/mysql/init到容器内的/docker-entrypoint-initdb.d目录，携带课前资料准备的SQL脚本
	- ③ 挂载/root/mysql/conf到容器内的/etc/mysql/conf.d目录，携带课前资料准备的配置文件

> [!tip]
> - 在执行 docker run 命令时, 使用 -v 本地目录 : 容器内目录 可以完成本地目录挂载
> - 本地目录必须以 "/" 或 "./" 开头, 如果直接以名称开头, 会被识别为数据卷而非本地目录
> 	- -v mysql:/var/lib/mysql 会被识别为一个数据卷
> 	- mysql-v ./mysql:/var/lib/mysql 会被识别为当前目录下的mysql目录



## 自定义镜像: 制作自己 java 项目的镜像

镜像就是包含了应用程序, 程序运行的系统函数库, 运行配置等文件包. 构建镜像的过程其实就是把上述文件打包的过程

> [!suammary] 部署一个java应用的步骤
> ① 准备一个Linux服务器
② 安装JRE并配置环境变量
③ 拷贝Jar包
④ 运行Jar包

还需要准备 jre 所依赖的系统

> [!summaray] 构建一个Java镜像的步骤
> ①准备一个Linux运行环境
②安装JRE并配置环境变量
③拷贝Jar包
④编写运行脚本

### 镜像结构
层: 添加安装包, 依赖, 配置等, 每次操作都形成新的一层

这样基础的镜像可以被其他镜像共享了

![[Pasted image 20240530152008.png]]

### Dockerfile
Dockerfile就是一个文本文件，其中包含一个个的指令(Instruction)，用指令来说明要执行什么操作来构建镜像。将来Docker可以根据Dockerfile帮我们构建镜像。常见指令如下：  
![[Pasted image 20240530152407.png]]

我们可以基于Ubuntu基础镜像，利用Dockerfile描述镜像结构
```bash
# 指定基础镜像
FROM ubuntu:16.04
# 配置环境变量，JDK的安装目录、容器内时区
ENV JAVA_DIR=/usr/local
# 拷贝jdk和java项目的包
COPY ./jdk8.tar.gz $JAVA_DIR/
COPY ./docker-demo.jar /tmp/app.jar
# 安装JDK
RUN cd $JAVA_DIR \ && tar -xf ./jdk8.tar.gz \ && mv ./jdk1.8.0_144 ./java8
# 配置环境变量
ENV JAVA_HOME=$JAVA_DIR/java8
ENV PATH=$PATH:$JAVA_HOME/bin
# 入口，java项目的启动命令
ENTRYPOINT ["java", "-jar", "/app.jar"]
```


jdk 基础 镜像
```bash
# 基础镜像
FROM openjdk:11.0-jre-buster
# 拷贝jar包
COPY docker-demo.jar /app.jar
# 入口
ENTRYPOINT ["java", "-jar", "/app.jar"]
```


编写好Dockerfile, 可以利用下面的命令来构建镜像

```bash
docker build -t myImage:1.0 . # 文件名必须为 Dockerfile
```

- -t ：是给镜像起名，格式依然是repository:tag的格式，不指定tag时，默认为latest
- .  ：是指定Dockerfile所在目录，如果就在当前目录，则指定为"."

```bash
docker build -t docker-demo .
```

```shell
# 基础镜像
FROM openjdk:11.0-jre-buster
# 设定时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# 拷贝jar包
COPY docker-demo.jar /app.jar
# 入口
ENTRYPOINT ["java", "-jar", "/app.jar"][root@Fancier520 demo]#
```

## 网络
默认情况下，所有容器都是以bridge方式连接到Docker的一个虚拟网桥上：
![[Pasted image 20240530160755.png]]

加入自定义网络的容器才可以通过容器名互相访问, Docker 的网络操作命令如下
![[Pasted image 20240530184142.png|600]]

创建容器的时候直接加入网络 

```bash
# 这样就不会加入默认网桥了
docker run -d --name mysql -p 8080:8080 --network xxx mysql
```

## 项目部署

### 部署 Java 应用

先将项目打包

然后 编写 镜像构建文件
```bash
# 基础镜像  
FROM openjdk:11.0-jre-buster  
# 设定时区  
ENV TZ=Asia/Shanghai  
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone  
# 拷贝jar包  
COPY hm-service.jar /app.jar  
# 入口  
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

将 构建文件 和 包 放到同一个目录下
通过命令构建镜像
```bash
docker build -t hmall .
```

创建并启动容器

```bash
# 放到 fencier 网段
docker run -d --name hm -p 8080:8080 --network fancier hmall
```

将 mysql 也 和 fancier 网段 联通

```bash
docker network connet fancier mysql
```


### 前端部署
需求: 创建一个新的nginx容器, 将 nginx.conf, html 目录与容器挂载

```bash
docker run -d \
--name nginx \
-p 18080:18080 \
-p 18081:18081 \
-v /root/nginx/html:/usr/share/nginx/html \
-v /root/nginx/nginx.conf:/etc/nginx/nginx.conf \
--network fancier \
nginx
```

给容器改名
```bash
docker rename CONTAINER NEW_NAM
```

查看 容器 日志
```bash
docker logs CONTAINER
```

### DockerCompose
Docker Compose通过一个单独的docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器，帮助我们实现多个相互关联的Docker容器的快速部署。
![[Pasted image 20240530202637.png|章鱼哥|300]]
模板文件
```yml
version: "3.8" # 项目
services:
  containerA: # 服务
    image: A
    container_name: A
    ports:
      - "11:11"
  containerB: # 服务
    image: B
    container_name: B
    ports:
      - "22:22"
  containerC: # 服务
    image: C
    container_name: C
    ports:
      - "33:33"
```

与 手动书写命令 对比
![[Snipaste_2024-05-30_20-32-56 1.png]]

```yml
version: "3.8"
services:
  mysql:
    image: mysql
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: 123456
    volumes:
      - "./mysql/conf:/etc/mysql/conf.d"
      - "./mysql/data:/var/lib/mysql"
      - "./mysql/init:/docker-entrypoint-initdb.d"
    networks:
      - hm-net
  hmall:
    build: # 自动 读取 Dockerfile 来构建 镜像
       context: .
       dockerfile: Dockerfile
    container_name: hmall
    ports:
      - "8080:8080"
    networks:
      - hm-net
    depends_on: # 依赖于 MySQL
      - mysql
  nginx:
    image: nginx
    container_name: nginx
    ports:
      - "18080:18080"
      - "18081:18081"
    volumes:
      - "./nginx/nginx.conf:/etc/nginx/nginx.conf"
      - "./nginx/html:/usr/share/nginx/html"
    depends_on:
      - hmall
    networks:
      - hm-net # 一个网络标识, 不是网络名
networks:
  hm-net: 
    name: fancier # 网络名
```

用 docker compose 文件 来部署

```bash
docker compose [OPTIONS] [COMMAND]
```

![[Pasted image 20240530204821.png|700]]






















