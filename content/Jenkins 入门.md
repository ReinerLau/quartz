---
title: Jenkins 入门
tags:
  - 工程化
---
## 为什么

实现 [[CICD|CI/CD]]

## 安装

[docker 安装](https://www.jenkins.io/doc/book/installing/docker/)

## 新建项目

点击 `New Item` , 输入项目名, 选择 `Freestyle Project`

### 配置仓库

安装 git

```sh
yum install git
```

`Source Code Management` 下输入仓库地址, 进入 `Add - Jenkins` , 输入拉去仓库代码的账号密码, `Branches to build` 指定拉取分支

### 配置 Node.js

`Manage Jenkins - Manage Plugins - Available plugins` 下搜索 `NodeJS`, 下载并安装完成后重启 Jenkins

`Manage Jenkins - Global Tool configuration - Node.js` 下选择 Node 版本, 保存, 进入 `项目 - Configure - Build Enviroment - Provide Node & npm bin / folder to PATH`

## 构建脚本

进入 `项目 - Configuration - Build Steps` 

配置脚本

```sh
npm install yarn -g
yarn install
yarn build
# 打包 build 后的文件
tar -zcvf dist.tar.gz dist/
# 删除 build 后的文件
rm -rf dist/
# 移动 build 后的压缩包到 nginx 托管目录下。
sudo mv dist.tar.gz /usr/share/nginx/html
# 进入托管目录下
cd /usr/share/nginx/html
sudo rm -rf dist/
# 解压
sudo tar -zxf dist.tar.gz
# 删除压缩包
sudo rm -rf dist.tar.gz
```

- `tar`：用于解压缩
- `-z`：在归档文件中使用 `gzip` 压缩算法
- `-c`：创建新的 `tar` 归档文件
- `-v`：显示 `tar` 命令执行过程中的详细信息
- `-f`：指定要操作的 `tar` 归档文件
- `-x`：从 `tar.gz` 文件中提取文件

保存后, 在 `/etc/sudoers` 文件中增加 `jenkins ALL=(ALL) NOPASSWD:ALL` 表示在执行 `sudo` 时不需要输入密码

## 配置 nginx

安装

```sh
yum install nginx
```

静态文件托管目录为 `/usr/share/nginx/html`，改为 `/usr/share/nginx/html/dist`

```sh
cd /etc/nginx
```

```diff
server {
        listen       80;
        listen       [::]:80;
        server_name  _;
+        root         /usr/share/nginx/html/dist;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
}
```

重启 nginx 服务

## 清空工作空间 (可选)

如果需要, 可以在每次构建完后清空工作空间

勾选 `项目 - Configure - Build Environment - Delete workspace before build starts`

## 使用

点击 `项目 - Build Now` 开始部署

## 监听 gitee 推送

### 安装插件

进入 `Manage Jenkins - Manage Plugins - Available`, 搜索 `gitee`, 点击 `Download now and install after restart`

### 配置插件

进入 `Manage Jenkins - Configure System - Gitee Configuration`

`Connection name` 填 `Gitee`

`Gitee host URL` 填 `https://gitee.com`

`Credentials` 填 [Gitee API token]([https://gitee.com/profile/personal_access_tokens](https://gitee.com/profile/personal_access_tokens))

点击 `Test Connection` 测试是否连接陈工

点击保存

### 配置项目

进入 `项目 - Configure → Source Code Management`

`Build Triggers` - 勾选 `Build when a change is pushed to Gitee. Gitee webhook URL`

`Build Triggers - Allowed branches` 指定想要触发构建的分支

`Secret Token for Gitee WebHook` 生成 `webhook` 密码

### WebHook

代码仓库 → 管理 → WebHooks → 添加 webHook

URL：配置项目时 Build Triggers → Build when a change is pushed to Gitee. Gitee webhook URL 后面的 URL

WebHook 密码：配置项目时生成的密码

勾选 Push 和激活

添加

点击测试模拟代码推送触发 jenkins 构建

## 版本回滚

### 配置

项目 → Configuration → General

勾选 This project is parameterized 表示通过参数构建

点击 Add Parameter 选择 Choice Paramter

Name 填一个 status 变量名表示构建的操作

Choices 加上 deploy 表示发布，rollback 表示回滚

点击 Add Parameter 选择 String Paramter

Name 填一个 version 变量名表示要回滚的版本

Default Value 为 0

### 脚本

```sh
bakpath="bak/$BUILD_NUMBER"
case $status in
    rollback)
    	if [ -d $bakpath ]
      then
        echo '已存在备份'
      else
        mkdir -p $bakpath
        sudo cp bak/$version/dist.tar.gz $bakpath
      fi
    	sudo cp bak/$version/dist.tar.gz /usr/share/nginx/html
      ;;
    *)
    	npm install yarn -g
      yarn install
      yarn build
      tar -zcvf dist.tar.gz dist/
      rm -rf dist/
      if [ -d $bakpath ]
      then
      	echo '已存在备份'
      else
      	mkdir -p $bakpath
        	cp dist.tar.gz $bakpath
      fi
      sudo mv dist.tar.gz /usr/share/nginx/html
      ;;
esac
cd bak
ls -t | awk 'NR>8'| xargs rm -rf
cd /usr/share/nginx/html
sudo rm -rf dist/
sudo tar -zxf dist.tar.gz
sudo rm -rf dist.tar.gz
```

- $BUILD_NUMBER：表示每次构建时递增的数字，是 jenkins 的内置变量
- ls -t | awk 'NR>8'| xargs rm -rf：表示只保留最新8个版本的备份

## 消息推送

### 全局配置

Manage Jenkins → Configure System → E-mail Notification

SMTP server： [smtp.gmail.com](http://smtp.gmail.com)

勾选 Use SMTP Authentication

输入 Gmail 邮箱地址

输入 Gmail App Password： [https://myaccount.google.com/security](https://myaccount.google.com/security)

勾选 SSL

端口输入 465

勾选 Test configuration by sending test e-mail 测试是否能发送

保存

### 项目配置

项目 → Configure → Post Build Actions

Add post-build action → email notification

输入邮箱地址

### 发布规则

- 每次构建失败
- 失败之后构建成功