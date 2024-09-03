---
title: 如何配置 HTTP 自签名证书
tags:
  - 运维
---
## 需求

项目在内网运行, 项目使用到一些 API, 比如麦克风, 摄像头, USB 串口等都要求浏览器域名跑在 https 下或者 localhost 下, 内部部署不可能通过 localhost 访问, 要么 ip 访问, 要么 127.0.0.1 访问

## 自签名证书

是一种由自己生成和签署的 SSL/TLS 证书, 不由受信任的证书颁发机构 (CA) 颁发, 这种证书通常用于测试、开发环境或在内网中使用，而不适用于公网, 配置完就能在服务器下使用 https 访问, 但会提示不安全, 点继续访问即可

## 生成

生成 ssl.key 文件

```shell
openssl genrsa -out ssl.key 4096
```

生成 ssl.csr 文件

```shell
openssl req -new -key ssl.key -out ssl.csr
```

生成 ssl.crt 文件

```shell
openssl x509 -req -days 365 -in ssl.csr -signkey ssl.key -out ssl.crt
```

## nginx 配置

```
server {
    listen 443 ssl;
    server_name  localhost;

    ssl_certificate /root/ssl/ssl.crt;
    ssl_certificate_key /root/ssl/ssl.key;
}
```

## Dockfile 配置

```
FROM nginx

COPY dist /usr/share/nginx/html

COPY ssl.crt /root/ssl/ssl.crt
COPY ssl.key /root/ssl/ssl.key

COPY index.conf /etc/nginx/conf.d/default.conf
```
