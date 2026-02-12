---
title: 'Docker学习笔记'
date: 2025-04-21 14:36:33
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
###  1. 什么虚拟机和docker？
   docker是容器的一种解决方案。
   虚拟机和容器是解决硬件资源高效运行的不同方案。两者最大的区别就是，容器间共用宿主机的操作系统。虚拟机可以安装不同的操作系统，而且必须安装操作系统。
###  2. 什么镜像和容器
镜像是容器的模板，如类和实例的关系，如模具和模型之间的关系。
### 3. 基本概念
![](https://h-pl.github.io/post-images/1745218730131.png)
### 4. docker体系结构
docker client与 server socket或restful api进行通信。
docker client--》docker daemon 处理 --》返回docker client。
### Dockerfile 构建镜像的步骤
Dockerfile，一条条的指令告诉docker如何构建镜像。
应用程序、各种依赖：
- 精简操作系统
- 运行时环境
- 应用程序
- 应用程序配置文件
- 应用程序第三方依赖包
- 应用程序插件

### 5. 修改docker源配置文件
**步骤**：
1. 编辑或创建配置文件：
```bash
sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json
```
2. 添加以下内容：
```json
{
  "registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"]
}
```
3. 如果你已经有 daemon.json 文件，注意保持 JSON 格式正确，不要覆盖已有配置项。

4. 重启 Docker，使配置文件生效：
```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

5. 验证是否生效
你可以使用如下命令验证镜像是否能正常拉取：

```bash
docker pull node:14-alpine
```
6. 从[dockerhub](https://hub.docker.com/_/node) 中查看是否包含这个tag。

```bash
docker info
```
检查输出中的 Registry Mirrors 是否包含你配置的镜像地址。
更多[镜像地址](https://www.coderjia.cn/archives/dba3f94c-a021-468a-8ac6-e840f85867ea#docker-%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F%E5%88%97%E8%A1%A820250320%E5%B7%B2%E6%9B%B4%E6%96%B0)，或采用[毫秒镜像](https://1ms.run/)。

### 6. 修改docker的代理

#### A. 检查或修改 Docker 代理设置
修改 Docker 守护进程的代理设置（适用于 Linux 系统）

1. **创建或编辑 Docker 服务的代理配置文件**  
   使用文本编辑器创建或编辑 `/etc/systemd/system/docker.service.d/http-proxy.conf` 文件，并加入如下内容（如果你需要代理就配置成代理服务器；如果不用，则移除代理环境变量）：
   ```ini
   [Service]
   Environment="HTTP_PROXY=http://127.0.0.1:10809"
   Environment="HTTPS_PROXY=http://127.0.0.1:10809"
   Environment="NO_PROXY=localhost,127.0.0.1"
   ```
   如果你已经在使用代理，但发现地址不对，可以修改成正确的代理地址和端口。如果你不需要代理，则可以删除这个文件或将内容清空。

2. **重新加载并重启 Docker 服务**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

#### B. 针对 `docker build` 使用 build-arg（如果只想在构建时传递代理设置）

1. 在构建镜像时你可以通过 `--build-arg` 指定代理变量：
```bash
docker build --build-arg HTTP_PROXY=http://127.0.0.1:10809 \
             --build-arg HTTPS_PROXY=http://127.0.0.1:10809 \
             --build-arg NO_PROXY=localhost,127.0.0.1 \
             -t your_image_name .
```
如果你不需要代理，也可以省略这些参数或者设置为空。

---

#### C. 检查环境变量

有时系统全局或用户级别可能设置了相关代理环境变量（如 `HTTP_PROXY`、`HTTPS_PROXY`）。你可以用下面的命令查看：
```bash
echo $HTTP_PROXY
echo $HTTPS_PROXY
```
如果这些环境变量不是你需要的，可以在当前终端会话中取消它们：
```bash
unset HTTP_PROXY
unset HTTPS_PROXY
```

### 7. Docker 打包示例
1. 文件目录。
```ini
- Dockerfile
- index.js
```
2.index.js 内容
docker build -t hello-docker .
```
