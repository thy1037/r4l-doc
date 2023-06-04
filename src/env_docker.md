# Docker 安装
为了统一开发环境，同时于其它开发环境做隔离，选择在 Docker 中做开发。

主要参考[官网](https://docs.docker.com/engine/install/)的 [Debian 系统安装教程](https://docs.docker.com/engine/install/)

## 卸载旧版本
```bash
sudo apt remove docker docker-engine docker.io containerd runc
```
其中：

`docker-engin` 在 Debian 仓库中已不存在，所以可能会报错，删除后重新执行即可。

## 使用仓库安装
Docker 支持多种安装方式，这里选择通过 apt 仓库的形式安装。

### 设置仓库
1. 更新 apt 包索引，并允许 apt 通过 HTTPS 使用仓库：
```bash
sudo apt update
sudo apt install ca-certificates curl gnupg
```

2. 添加 Docker 的官方 GPG 密钥：
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

3. 使用以下命令设置仓库
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 安装 Docker Engine
1. 更新 apt 包索引
```bash
sudo apt update
```

2. 安装 Docker Engine, containerd 和 Docker Compose

最新版：
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. 通过运行 `hello-world` 镜像来确认 Docker Engine 安装成功：
```bash
sudo docker run hello-world
```
成功输出：
```bash
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:4e83453afed1b4fa1a3500525091dbfca6ce1e66903fd4c01ff015dbcb1ba33e
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## Debian 镜像
因为个人喜好，并且宿主机使用的 Debian 系统，所以 Docker 中依旧选择 Debian，但要说的是 Ubuntu 是支持最完善的系统镜像。

## 安装镜像
通过运行 `hello-world` 发现，在本地没有镜像的情况下，Docker 会自动从[官方镜像仓库](https://hub.docker.com/)中下载，所以执行运行：
```bash
sudo docker run -it debian bash
```
其中：

**-i**：开启交互（interactive） <br>
**-t**：开启终端（pseudo-TTY） <br>
**debian**：镜像名称 <br>
**bash**：应用名称

输出如下：
```bash
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
b0248cf3e63c: Pull complete 
Digest: sha256:0a78ed641b76252739e28ebbbe8cdbd80dc367fba4502565ca839e5803cfd86e
Status: Downloaded newer image for debian:latest
root@8bfa66c20540:/# 
```

这是一个非常基础的 Debian 系统，在安装过程中可以看到下载的大小只有 55M 左右，其中包含的命令（可执行文件）非常少，需要根据使用需求安装各种开发工具。

## 添加 sudo 权限
正常在安装完 Docker 后，所有命令都需要 sudo 权限，使用很不方便（但安全），可以将 docker 加入 sudo 用户组，即默认 sudo 权限运行，具体步骤如下：

1. 创建用户组
首先查看是否有用户组：
```bash
cat /etc/group | grep docker
```
如果没有则创建用户组：
```bash
sudo groupadd docker
```

2. 当前用户加入用户组
```bash
sudo gpasswd -a ${USER} docker
```

3. 重启 docker 服务
```bash
sudo service docker restart
```

4. docker.sock 添加权限
```bash
sudo chmod a+r2 /var/run/docker.sock
```

