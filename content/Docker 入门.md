---
title: Docker 入门
tags:
  - 工程化
  - 运维
---
## 容器

### 查看正在运行的容器

```shell
docker ps
```

### 运行容器

```shell
docker run -p 3000:3000 <镜像名称>
```

## 镜像

### 拉取镜像

```shell
docker pull <镜像名称>
```

### 查看镜像

```shell
docker image ls
```

### 创建镜像

```shell
docker build -t <镜像名称> .
```

### 保存指定镜像为 tar 文件

```shell
docker save -o <保存路径/文件名.tar> <镜像名称>
```

### tar 文件转镜像

```shell
docker load -i <镜像文件路径/文件名.tar>
```

## 发布镜像

### 登录

```shell
docker login
```

### 创建仓库

1. 进入 https://hub.docker.com/
2. 点击 Create repository 按钮
3. 填写信息后点击 Create 按钮

![image](https://github.com/ReinerLau/imrobot-monitor/assets/103234074/e5b8a018-2243-42fd-821a-0b66b161b42c)

### 打标签

```shell
docker tag <已存在的镜像名> <用户名>/<仓库名>
```

### 推送

```shell
docker push <用户名>/<仓库名>
```

### 校验镜像是否上传成功

```shell
docker inspect <用户名>/<仓库名>
```

## 发布多架构镜像

### 创建 Buildx 构建器实例

```shell
docker buildx create --use
```

### 构建并发布多架构镜像

```shell
docker buildx build --platform linux/amd64,linux/arm64 -t <用户名>/<仓库名> . --push
```

## docker compose

### 运行

```shell
docker-compose up -d
```

### 代替 host.docker.internal

```
172.17.0.1
```