> ⚠️ **注意:** 本项目使用了 [FRP (Fast Reverse Proxy)](https://github.com/fatedier/frp) 项目。这是一个用于快速部署 FRP 的 Docker 解决方案。

# 🚀 FRP Docker部署项目 (服务端与客户端)

本项目提供了一套使用 Docker 和 Docker Compose 快速部署 FRP (Fast Reverse Proxy) 服务端和客户端的解决方案。你可以轻松地在 VPS ☁️ 上部署 FRP 服务端，并在本地机器 💻 上部署客户端，实现内网穿透 🌐。

## 📚 目录

*   [✨ 功能特性](#-功能特性)
*   [📁 目录结构](#-目录结构)
*   [🛠️ 准备工作](#️-准备工作)
    *   [环境要求](#环境要求)
    *   [配置详解](#配置详解)
*   [🚀 部署](#-部署)
    *   [部署服务端 (VPS)](#1-部署服务端-在-vps-️-上)
    *   [部署客户端 (本地)](#2-部署客户端-在本地机器--上)
*   [⚙️ 日常管理](#️-日常管理)
    *   [查看日志](#查看日志)
    *   [更新配置](#更新配置-1)
    *   [停止与移除](#停止与移除-1)

## ✨ 功能特性

*   🐳 使用 Docker 容器化部署，环境隔离，方便管理。
*   แยก 服务端 (`frps`) 和客户端 (`frpc`) 分离部署。
*   ⚙️ 使用 `docker-compose` 简化构建和运行流程。
*   📄 支持通过 TOML 配置文件 (`frps.toml`, `frpc.toml`) 进行参数配置。
*   🔗 容器使用 `host` 网络模式，方便端口映射和访问本地服务。
*   📜 容器日志可通过 `docker logs` 查看。

## 📁 目录结构

```
frp/
├── client/             # 客户端配置与部署文件
│   ├── Dockerfile        # 客户端 Docker 镜像构建文件
│   ├── docker-compose.yml # 客户端 Docker Compose 部署文件
│   └── frpc.toml         # 客户端 FRP 配置文件
├── server/             # 服务端配置与部署文件
│   ├── Dockerfile        # 服务端 Docker 镜像构建文件
│   ├── docker-compose.yml # 服务端 Docker Compose 部署文件
│   └── frps.toml         # 服务端 FRP 配置文件
└── README.md           # 📝 本说明文件
```

## 🛠️ 准备工作

### 环境要求

确保你的 VPS 和本地机器上都安装了以下软件：

*   🐳 [Docker](https://docs.docker.com/engine/install/)
*   ⚙️ [Docker Compose](https://docs.docker.com/compose/install/) (通常随 Docker Desktop 一起安装)

### 配置详解

在部署之前，请务必仔细检查并修改以下配置文件：

#### 1. ☁️ 服务端配置 (`server/frps.toml`)

*   `bindPort`: FRP 服务端监听的端口，客户端将连接此端口。默认为 `7000`。
*   `webServer.port`: 服务端 Dashboard 📊 的访问端口。默认为 `7500`。
*   `webServer.user`: Dashboard 登录用户名。默认为 `admin`。
*   `webServer.password`: **(重要)** Dashboard 登录密码。**请务必修改为一个强密码！** 🔑
*   `auth.token`: **(重要)** 用于客户端连接认证的令牌。**请务必修改为一个安全的随机字符串！** 🔒 客户端配置中的 `auth.token` 必须与此值完全一致。

#### 2. 💻 客户端配置 (`client/frpc.toml`)

*   `serverAddr`: **(重要)** 你的 **VPS 的公网 IP 地址**。将其替换为实际 IP。📍
*   `serverPort`: FRP 服务端的监听端口，必须与 `server/frps.toml` 中的 `bindPort` 一致。默认为 `7000`。
*   `auth.token`: **(重要)** 连接服务端的认证令牌。**必须与 `server/frps.toml` 中设置的 `auth.token` 完全一致！** 🔒
*   `[[proxies]]`: 定义端口映射规则 🔄。
    *   `name`: 代理规则的名称，必须唯一。
    *   `type`: 代理类型，常用 `tcp` 或 `udp`。
    *   `localIP`: **(重要 - 平台差异 🐧/🍎/🪟)**
        *   **Linux (如 Ubuntu):** 通常使用 `127.0.0.1` 来访问运行在宿主机上的服务。
        *   **Windows / macOS:** 如果要访问**直接运行在 Windows/macOS 宿主机上的服务**，请使用 `host.docker.internal`。如果服务也运行在 Docker 容器中，则根据情况使用 `127.0.0.1` 或服务容器名。
    *   `localPort`: 你本地需要被穿透的服务的端口。
    *   `remotePort`: 你希望在 VPS 上暴露的公网端口。访问 `VPS公网IP:remotePort` 即可访问到本地服务。
    *   你可以添加多个 `[[proxies]]` 段来映射不同的服务。

#### 3. 🔩 CPU 架构 (`client/docker-compose.yml` 和 `server/docker-compose.yml`)

*   Dockerfile 默认使用 `amd64` 架构的 FRP。如果你的机器 (特别是客户端本地机器) 使用不同的架构 (如 Apple Silicon Mac 的 `arm64`)，请在对应的 `docker-compose.yml` 文件中取消注释并修改 `args` 下的 `ARCH` 参数。例如：
    ```yaml
    services:
      frpc: # 或 frps
        build:
          context: .
          args:
            ARCH: arm64 # 修改为你的架构
    ```

## 🚀 部署

### 1. 部署服务端 (在 VPS ☁️ 上)

1.  📂 将 `server` 文件夹完整复制到你的 VPS 上。
2.  ➡️ 进入 VPS 上的 `server` 目录: `cd /path/to/server`
3.  🏗️ 构建并以后台模式启动服务端容器:
    ```bash
    docker-compose up -d --build
    ```
4.  ✅ 部署完成！你可以通过 [查看日志](#查看日志) 或 [访问 Dashboard](#访问-dashboard) 来确认运行状态。

### 2. 部署客户端 (在本地机器 💻 上)

1.  📂 将 `client` 文件夹放在你的本地机器上。
2.  ➡️ 进入本地机器上的 `client` 目录: `cd /path/to/client`
3.  🏗️ 构建并以后台模式启动客户端容器:
    ```bash
    docker-compose up -d --build
    ```
4.  ✅ 部署完成！你可以通过 [查看日志](#查看日志) 来确认连接和服务状态。

## ⚙️ 日常管理

### 查看日志

*   **服务端日志:** 在 VPS 的 `server` 目录下运行：
    ```bash
    docker logs frps_server
    # 或持续查看:
    # docker logs -f frps_server
    ```
*   **客户端日志:** 在本地机器的 `client` 目录下运行：
    ```bash
    docker logs frpc_client
    # 或持续查看:
    # docker logs -f frpc_client
    ```
    客户端日志会显示连接状态和代理启动情况。

### 访问 Dashboard

在浏览器中打开 `http://<你的VPS公网IP>:7500` (使用你在 `frps.toml` 中配置的用户名和密码登录)。

### 更新配置

当你想修改 FRP 的设置 (例如添加/删除端口映射规则、更改认证令牌等) 时，请按照以下步骤操作：

1.  📝 **修改配置文件:** 直接编辑你想要更改的配置文件 (`server/frps.toml` 或 `client/frpc.toml`) 并保存。
2.  🚀 **重新构建并启动容器:** 在对应的目录 (`server/` 或 `client/`) 下，再次运行以下命令：
    ```bash
    docker-compose up -d --build
    ```
    这个命令会使用修改后的配置文件重新构建 Docker 镜像，并用新镜像启动（或替换）容器。FRP 服务会加载新的配置。

**⚠️ 注意 (替代方法):**
*   如果你在 `docker-compose.yml` 文件中取消注释了 `volumes` 部分，将宿主机的配置文件挂载到了容器内部 (例如 `- ./frps.toml:/frp/frps.toml`)，那么更新配置会更简单：
    1.  📝 直接编辑宿主机上的 `.toml` 文件并保存。
    2.  🔄 在对应的目录下运行 `docker-compose restart frps` (或 `frpc`) 来重启容器，使其加载新的配置。**无需** `--build`。

### 停止与移除

*   **停止容器:** 在对应的 `server` 或 `client` 目录下运行 `docker-compose down`。
*   **停止并移除容器和网络:** `docker-compose down -v` (会移除容器但不移除镜像)。
*   **移除镜像:** 使用 `docker rmi <image_name_or_id>` 🗑️。你可以使用 `docker images` 查看镜像列表。

---

🎉 现在你可以根据这些说明配置和部署你的 FRP 服务了。
