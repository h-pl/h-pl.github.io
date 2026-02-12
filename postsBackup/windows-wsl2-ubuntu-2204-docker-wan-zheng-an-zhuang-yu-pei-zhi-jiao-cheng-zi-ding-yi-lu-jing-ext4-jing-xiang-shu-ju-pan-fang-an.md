---
title: 'Windows + WSL2 + Ubuntu 22.04 + Docker: 完整安装与配置教程 (自定义路径 ext4 镜像数据盘方案)'
date: 2025-06-01 00:32:21
tags: [攻略,WSL]
published: true
hideInList: false
feature: 
isTop: false
---
**目标:** 在 WSL2 (Ubuntu 22.04) 中搭建一个稳定高效的 Docker 环境，其数据目录通过一个存储在 Windows D盘 `VHDX` 目录下的 `docker-ext4.img` 文件（格式化为 ext4）来提供，并挂载到 WSL 的 `/mnt/d/docker-data` 目录，优先使用 `overlay2` 存储驱动。

---
**1. 准备工作 (Windows)**

* **确保 WSL2 已安装并为默认版本：**
    打开 PowerShell (管理员权限)，运行：
    ```powershell
    wsl --set-default-version 2
    ```
* **确保 Windows 功能 "虚拟机平台" 已启用：**
    可以在 "控制面板" -> "程序" -> "启用或关闭 Windows 功能" 中检查并勾选 "虚拟机平台" 和 "适用于 Linux 的 Windows 子系统"。

---
**2. 安装/初始化 Ubuntu 22.04 on WSL2**

* **如果尚未安装 Ubuntu 22.04：**
    在 PowerShell (管理员权限) 中运行：
    ```powershell
    wsl --install -d Ubuntu-22.04
    ```
    安装完成后，系统可能会提示重启。
* **首次启动 Ubuntu：**
    重启后，从 Windows 开始菜单搜索并启动 "Ubuntu"。首次启动会进行初始化，并提示你创建 Linux 用户名和密码。

---
**3. 启用 Systemd (强烈推荐)**

启用 Systemd 可以使用标准的 Linux 服务管理和 `/etc/fstab` 自动挂载功能。

* **编辑或创建 `/etc/wsl.conf` 文件：**
    在你的 Ubuntu (WSL) 终端中：
    ```bash
    sudo nano /etc/wsl.conf
    ```
    确保文件包含以下内容：
    ```ini
    [boot]
    systemd=true
    ```
* **关闭并重启 WSL 实例 (关键步骤)：**
    保存 `/etc/wsl.conf` 文件后，**必须完全关闭所有 WSL 实例**。回到 Windows PowerShell (管理员权限)，运行：
    ```powershell
    wsl --shutdown
    ```
    等待几秒钟，然后重新从开始菜单启动你的 Ubuntu 发行版。
* **验证 Systemd 是否运行：**
    重新进入 Ubuntu 后，执行：
    ```bash
    ps --no-headers -o comm 1
    # 或者
    systemctl is-system-running --wait
    ```
    如果第一个命令输出 `systemd` 或者第二个命令最终输出 `running` 或 `degraded` (degraded 也可以工作)，则表示成功。

---
**4. 更新 Ubuntu 软件包**

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt autoremove -y && sudo apt clean
```

---
**5. 安装 Docker 引擎**

* **卸载旧版本 (如果存在)：**
    ```bash
    for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
    sudo apt-get autoremove -y
    ```
* **设置 Docker 的 APT 仓库：**
    ```bash
    sudo apt-get update
    sudo apt-get install -y ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```
* **安装 Docker 引擎、CLI、Containerd 和 Compose 插件：**
    ```bash
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

---
**6. 配置 Docker 用户组 (免 `sudo` 使用 Docker)**

* 将当前用户添加到 `docker` 组：
    ```bash
    sudo usermod -aG docker $USER
    ```
* **应用更改：** 退出当前的 WSL 终端会话并重新打开一个新的，或者在当前会话中执行 `newgrp docker`。推荐重新打开终端。

---
**7. 创建并挂载 ext4 镜像文件 (Docker 数据盘)**

* **7.1. 定义路径和大小 (这些是你指定的路径)：**
    * 镜像文件完整路径：`/mnt/d/VHDX/docker-ext4.img`
    * 镜像大小 (示例)：`60G` (你可以根据需要调整，例如 `40G`, `100G`)
    * WSL 中的挂载点：`/mnt/d/docker-data`

* **7.2. 在 D 盘上创建存储 `.img` 文件的目录 (通过 WSL)：**
    ```bash
    sudo mkdir -p /mnt/d/VHDX
    ```

* **7.3. 创建指定大小的稀疏镜像文件：**
    ```bash
    sudo truncate -s 60G /mnt/d/VHDX/docker-ext4.img # 将 60G 替换为你需要的大小
    ```

* **7.4. 将镜像文件格式化为 ext4：**
    ```bash
    sudo mkfs.ext4 -F /mnt/d/VHDX/docker-ext4.img
    ```

* **7.5. 在 WSL 中创建挂载点 (这就是你指定的 `/mnt/d/docker-data`)：**
    这个目录路径 `/mnt/d/docker-data` 比较特殊，因为它本身位于 WSL 对 Windows D 盘的挂载点 `/mnt/d/` 之下。通常我们会把 Linux 文件系统挂载到一个 WSL 内部更“纯粹”的路径（如 `/srv/docker-storage`）。但如果你坚持要用 `/mnt/d/docker-data` 作为 ext4 镜像的挂载点，技术上是可以的，只是路径看起来有点像仍在直接使用 D 盘，但实际上它会是 loop 设备挂载的 ext4。
    **为避免混淆和潜在问题，强烈建议将 ext4 镜像挂载到 WSL 内部的一个独立目录，例如 `/srv/docker_ext4_data` 或 `/opt/docker_data`。**
    但如果**你坚持要用 `/mnt/d/docker-data` 作为挂载点**，请确保这个目录在挂载前是空的，并且没有被其他重要用途占用。以下命令会创建它：
    ```bash
    sudo mkdir -p /mnt/d/docker-data
    ```
    **(推荐的备选挂载点：`sudo mkdir -p /srv/docker_ext4_data`，后续命令中的 `/mnt/d/docker-data` 也要相应修改)**

* **7.6. 配置自动挂载镜像文件 (回答“每次都要mount吗？”)：**
    **是的，如果不进行自动化配置，每次 WSL 完全重启后，你都需要手动重新挂载。** 以下是如何实现自动挂载：

    * **方案 A: 如果 Systemd 已成功启用 (强烈推荐，实现开机自动挂载)：**
        编辑 `/etc/fstab` 文件：
        ```bash
        sudo nano /etc/fstab
        ```
        在文件末尾添加新的一行：
        ```fstab
        /mnt/d/VHDX/docker-ext4.img   /mnt/d/docker-data   ext4   loop,defaults,nofail,x-systemd.requires=/mnt/d   0 0
        ```
        **解释 `nofail` 和 `x-systemd.requires=/mnt/d`：**
        * `nofail`: 即使 `.img` 文件无法挂载，WSL 系统也会正常启动。
        * `x-systemd.requires=/mnt/d`: 确保在尝试挂载此 loop 设备前，`/mnt/d` (Windows D盘) 自身已经被 WSL 挂载。
        保存文件。然后测试挂载（如果之前未挂载）：
        ```bash
        sudo mount -a
        df -h /mnt/d/docker-data # 检查是否挂载成功并显示大小
        ```
        通过 `fstab` 配置后，WSL (在 systemd 模式下) 启动时会自动尝试挂载。

    * **方案 B: 如果 Systemd 未启用或无法工作 (备选方案，每次开终端时尝试挂载)：**
        编辑你的 `~/.bashrc` (或 `~/.zshrc` 等)：
        ```bash
        sudo nano ~/.bashrc
        ```
        在文件末尾添加：
        ```bash
        # Auto-mount Docker image if not already mounted
        if ! mountpoint -q /mnt/d/docker-data; then
          echo "Attempting to mount Docker image for WSL session (/mnt/d/docker-data)..."
          sudo mount -o loop /mnt/d/VHDX/docker-ext4.img /mnt/d/docker-data > /dev/null 2>&1
          if mountpoint -q /mnt/d/docker-data; then
            echo "Docker image mounted at /mnt/d/docker-data."
          else
            echo "Failed to mount Docker image. Sudo password might be required or path is incorrect."
          fi
        fi
        ```
        这种方法会在每次打开新的终端时尝试挂载。`sudo` 可能会请求密码。

* **7.7. 执行首次挂载 (如果使用 fstab，`sudo mount -a` 已完成或重启WSL后生效；如果使用 `.bashrc`，打开新终端即可)**

---
**8. 配置 Docker 守护进程 (`daemon.json`)**

* **创建 Docker 配置目录 (如果不存在)：**
    ```bash
    sudo mkdir -p /etc/docker
    ```
* **创建或编辑 `/etc/docker/daemon.json` 文件：**
    ```bash
    sudo nano /etc/docker/daemon.json
    ```
    粘贴以下内容。Docker 的 `data-root` 应该是挂载点 `/mnt/d/docker-data` 内部的一个**子目录**，例如 `/mnt/d/docker-data/docker`：
    ```json
    {
      "data-root": "/mnt/d/docker-data/docker",
      "storage-driver": "overlay2",
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "20m",
        "max-file": "5"
      },
      "registry-mirrors": [
        // 你在日本，可以查找是否有日本地区的Docker镜像加速器，或者留空 [] 使用Docker Hub官方源
      ]
    }
    ```
* **在挂载的 ext4 文件系统中创建实际的 Docker 数据目录：**
    ```bash
    sudo mkdir -p /mnt/d/docker-data/docker
    ```

---
**9. 启动并管理 Docker 服务**

* **方案 A: 如果 Systemd 已成功启用 (推荐)：**
    ```bash
    sudo systemctl daemon-reload  # 如果修改了 daemon.json
    sudo systemctl restart docker.service
    sudo systemctl enable docker.service # 设置 Docker 开机自启 (WSL启动时)
    sudo systemctl status docker.service # 查看状态
    ```
* **方案 B: 如果 Systemd 未启用或无法工作：**
    手动启动 Docker 守护进程：
    ```bash
    sudo dockerd > /tmp/dockerd.log 2>&1 &
    ```
    你需要确保在每次 WSL 启动后，并且 loop 镜像已挂载后，再执行此命令。

---
**10. 验证 Docker 安装**

* （在新终端，或执行 `newgrp docker` 后）运行：
    ```bash
    docker info
    ```
    检查 `Server Version`, `Storage Driver` (应为 `overlay2`), `Docker Root Dir` (应为 `/mnt/d/docker-data/docker`)。
* 运行 `hello-world` 测试：
    ```bash
    docker run hello-world
    ```

---
**11. 运行 Nginx 测试 (本地开发示例)**

1.  **在 D 盘创建你的网站项目目录 (通过 WSL)：**
    ```bash
    mkdir -p /mnt/d/MyLocalSite
    echo "<h1>Nginx on Docker (ext4 loop on D:) Works!</h1>" > /mnt/d/MyLocalSite/index.html
    ```
2.  **运行 Nginx 容器：**
    ```bash
    docker run --name my-nginx-from-d -d -p 8080:80 -v /mnt/d/MyLocalSite:/usr/share/nginx/html nginx
    ```
3.  **在 Windows 浏览器中访问：** `http://localhost:8080`

---
**总结**

通过 `/etc/fstab` (配合 systemd) 或 `~/.bashrc` 脚本，你可以实现 Docker 使用的 ext4 回环镜像的自动挂载，从而**避免每次启动 WSL 后手动 `mount`**。强烈推荐启用 systemd 并使用 `/etc/fstab` 的方法，因为它更健壮和符合 Linux 标准。

请仔细检查所有路径，特别是你指定的 `/mnt/d/VHDX/docker-ext4.img` 和挂载点 `/mnt/d/docker-data`，以及 Docker `data-root` 的子目录 `/mnt/d/docker-data/docker`。