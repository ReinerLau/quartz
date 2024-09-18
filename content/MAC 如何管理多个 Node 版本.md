---
title: MAC 如何管理多个 Node 版本
tags:
  - node
---
# nvm

安装

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

```sh
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

安装指定 node 版本

```sh
nvm install 16
```

切换到指定 node 版本

```sh
nvm use 16
```

查看已下载的 node 版本

```sh
nvm ls
```

查看可下载的 node 版本

```sh
nvm ls-remote
```
## 疑难解答

> `nvm ls-remote` 只有 iojs 版

nvm 镜像地址的问题，修改对应环境变量 `NVM_NODEJS_ORG_MIRROR` 即可, 可以使用 淘宝源(`https://npmmirror.com/mirrors/node/`)

查看当前设置的值

```sh
echo $NVM_NODEJS_ORG_MIRROR
```

运行时修改

```sh
NVM_NODEJS_ORG_MIRROR=https://npmmirror.com/mirrors/node nvm ls-remote
```

临时修改环境变量

```sh
export NVM_NODEJS_ORG_MIRROR=https://npmmirror.com/mirrors/node
```

```sh
nvm ls-remote
```

> 每次重新打开 IDE, node 就会切换到最低版本

改变默认版本

```sh
nvm alias default <版本号>
```