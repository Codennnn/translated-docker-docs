---
description: 查阅推荐的 Docker Engine 安装后步骤（Linux），包括以非 root 用户运行 Docker 等。
keywords: run docker without sudo, docker running as root, docker post install, docker
  post installation, run docker as non root, docker non root user, how to run docker
  in linux, how to run docker linux, how to start docker in linux, run docker on linux
title: Docker Engine 的 Linux 安装后步骤
linkTitle: 安装后步骤
weight: 90
aliases:
- /engine/installation/linux/docker-ee/linux-postinstall/
- /engine/installation/linux/linux-postinstall/
- /install/linux/linux-postinstall/
---

以下可选的安装后步骤，介绍如何配置你的 Linux 主机以更好地配合 Docker 使用。

## 以非 root 用户管理 Docker

Docker 守护进程绑定的是 Unix 套接字，而非 TCP 端口。默认情况下，该套接字归 `root` 用户所有，其他用户只能通过 `sudo` 访问。
Docker 守护进程始终以 `root` 用户运行。

如果不想在每次运行 `docker` 命令时都加上 `sudo`，可以创建名为 `docker` 的 Unix 组，并将用户加入该组。
当 Docker 守护进程启动时，它会创建一个仅 `docker` 组成员可访问的 Unix 套接字。
在某些 Linux 发行版上，通过包管理器安装 Docker Engine 时系统会自动创建该组，此时无需手动创建。

<!-- prettier-ignore -->
> [!WARNING]
>
> `docker` 组会赋予用户接近 root 级别的权限。有关对系统安全的影响，参见
> [Docker 守护进程的攻击面](../security/_index.md#docker-daemon-attack-surface)。

> [!NOTE]
>
> 若要在无 root 权限下运行 Docker，参见
> [以非 root 用户运行 Docker 守护进程（Rootless 模式）](../security/rootless.md)。

创建 `docker` 组并添加你的用户：

1. 创建 `docker` 组。

   ```console
   $ sudo groupadd docker
   ```

2. 将你的用户加入 `docker` 组。

   ```console
   $ sudo usermod -aG docker $USER
   ```

3. 注销并重新登录，使你的组成员身份生效。

   > 如果你在虚拟机中运行 Linux，可能需要重启虚拟机以使更改生效。

   你也可以运行以下命令以立即应用组更改：

   ```console
   $ newgrp docker
   ```

4. 验证你可以在不使用 `sudo` 的情况下运行 `docker` 命令。

   ```console
   $ docker run hello-world
   ```

   该命令会拉取测试镜像并在容器中运行。容器启动后会打印一条消息，然后退出。

   如果在将用户加入 `docker` 组之前，曾使用 `sudo` 运行过 Docker CLI 命令，可能会看到如下错误：

   ```text
   WARNING: Error loading config file: /home/user/.docker/config.json -
   stat /home/user/.docker/config.json: permission denied
   ```

   该错误表明 `~/.docker/` 目录的权限设置不正确，原因是之前使用了 `sudo`。

   要修复此问题，可以删除 `~/.docker/` 目录（它会自动重建，但自定义设置会丢失），
   或使用以下命令更改其所有权与权限：

   ```console
   $ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
   $ sudo chmod g+rwx "$HOME/.docker" -R
   ```

## 使用 systemd 配置开机自启动

许多现代 Linux 发行版使用 [systemd](https://systemd.io/) 管理系统启动时的服务。
在 Debian 与 Ubuntu 上，Docker 服务默认随开机启动。对于其他使用 systemd 的发行版，
如需在开机时自动启动 Docker 与 containerd，请运行：

```console
$ sudo systemctl enable docker.service
$ sudo systemctl enable containerd.service
```

如需取消该行为，可改用 `disable`：

```console
$ sudo systemctl disable docker.service
$ sudo systemctl disable containerd.service
```

你可以通过 systemd 单元文件在启动时配置 Docker 服务，例如添加 HTTP 代理、
为 Docker 运行时文件设置不同的目录或分区，或进行其他自定义。
示例参见：[配置守护进程使用代理](/manuals/engine/daemon/proxy.md#systemd-unit-file)。

## 配置默认日志驱动

Docker 提供了[日志驱动](/manuals/engine/logging/_index.md)，用于收集与查看主机上所有容器的日志数据。
默认日志驱动 `json-file` 会将日志写入主机文件系统中的 JSON 文件。随着时间推移，日志文件可能不断增大，
导致磁盘空间被耗尽。

为避免日志数据占用过多磁盘空间，可考虑以下方案：

- 为 `json-file` 日志驱动开启[日志轮转](/manuals/engine/logging/drivers/json-file.md)。
- 使用[其他日志驱动](/manuals/engine/logging/configure.md#configure-the-default-logging-driver)，
  例如默认启用日志轮转的[“local” 日志驱动](/manuals/engine/logging/drivers/local.md)。
- 使用可将日志发送到远程聚合器的日志驱动。

## 进一步阅读

- 前往 [Docker workshop](/get-started/workshop/_index.md)，学习如何构建镜像并将其作为容器化应用运行。
