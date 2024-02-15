---
title: docker部署
abbrlink: 6
categories: docker学习笔记
tags: docker
updated: 2024-01-18
data: 2024-01-18

---



### 新建Dockerfile

```bash
# Docker 镜像构建
FROM maven:3.5-jdk-8-alpine as builder

# Copy local code to the container image.
WORKDIR /app

COPY target ./target
#COPY pom.xml .
#COPY src ./src

# Build a release artifact.
#RUN mvn package -DskipTests

# Run the web service on container startup.
CMD ["java","-jar","/app/target/user-center-0.0.1-SNAPSHOT.jar","--spring.profiles.active=prod"]
```

<!-- more -->

Dockerfile编写

- FROM 依赖的基础镜像
- WORKDIR工作目录
- COPY从本机复制文件
- RUN执行命令
- CMD/ENTRYPOINT（附加额外参数）指定运行容器时默认执行的命令

### 根据Dockerfile构建docker镜像

```bash
docker build -t usercenter-backend:v0.0.1 .
```

### 从远程仓库拉取镜像

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
// 示例
docker pull hello-world
```

### 根据镜像创建容器实例

```bash
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
// 启动实例，得到容器实例containerId
docker create hello-world
```

### 启动容器

```ba
docker start [OPTIONS] CONTAINER [CONTAINER...]
// 启动示例
sudo docker start mystifying_shamir
```

### docker run 启动

```bash
docker run -p 8080:8080 usercenter-backend:v0.0.1
```

### 查看镜像

```bash
docker images
```

### 查看进程

```bash
docker ps
```

### 查看日志信息

```bash
docker logs -f <container-id>
```

### 关闭容器

```bash
docker kill 
```

### 删除镜像

```bash
docker rmi <images-id>
# 强制删除
docker rmi -f <images-id>
```

### 停止容器

```bash
docker stop <container-id>
```

### 删除容器

```
docker rm <container-id>
```



### Q&A

Q: springboot项目部署后无法访问数据库

A: 修改application-prod.yml中数据库url中的地址为本机ip地址（172.xxx）

------

Q: 出现Host is not allowed to connect to this MySQL server错误

A: MySQL不允许远程登录 

1在装有MySQL的机器上登录

``` mysql
MySQL mysql -u root -p <密码>
```

执行

```mysql
use mysql;
select host from user where user = ‘root’;
```

查看当前的root用户限制在当前的ip内访问的，需要修改他的访问域。

执行

```mysql
update user set host = ‘%’ where user = ‘root’;
```

执行FLUSH PRIVILEGES 或者重启 MySQL 即可;