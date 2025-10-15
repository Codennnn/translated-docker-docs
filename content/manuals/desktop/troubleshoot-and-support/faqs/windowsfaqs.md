---
description: Docker Desktop for Windows 常见问题
keywords: desktop, windows, faqs
title: Docker Desktop for Windows 常见问题
linkTitle: Windows
tags: [FAQ]
weight: 30
---

### 可以同时使用 VirtualBox 和 Docker Desktop 吗？

可以。如果你在机器上启用了 [Windows Hypervisor Platform](https://docs.microsoft.com/en-us/virtualization/api/) 功能，就可以同时运行 VirtualBox 和 Docker Desktop。

### 为什么需要 Windows 10 或 Windows 11？

Docker Desktop 使用了 Windows Hyper-V 功能。虽然较旧的 Windows 版本也有 Hyper-V，但它们的 Hyper-V 实现缺少 Docker Desktop 运行所必需的关键功能。

### 可以在 Windows Server 上运行 Docker Desktop 吗？

不可以，不支持在 Windows Server 上运行 Docker Desktop。

### 符号链接在 Windows 上如何工作？

Docker Desktop 支持两种类型的符号链接：Windows 原生符号链接和容器内创建的符号链接。

Windows 原生符号链接在容器内会显示为符号链接，而容器内创建的符号链接则表示为 [mfsymlinks](https://wiki.samba.org/index.php/UNIX_Extensions#Minshall.2BFrench_symlinks)。这些是带有特殊元数据的常规 Windows 文件。因此，在容器内创建的符号链接在容器内显示为符号链接，但在宿主机上则不是。

### Kubernetes 和 WSL 2 的文件共享

Docker Desktop 将 Windows 宿主机文件系统挂载到运行 Kubernetes 的容器内的 `/run/desktop` 下。有关如何配置 Kubernetes 持久卷以表示宿主机上的目录的示例，请参阅 [Stack Overflow 帖子](https://stackoverflow.com/questions/67746843/clear-persistent-volume-from-a-kubernetes-cluster-running-on-docker-desktop/69273405#69273)。

### 如何添加自定义 CA 证书？

你可以向 Docker 守护进程添加受信任的证书颁发机构（CA），用于验证仓库服务器证书和客户端证书，以便向仓库进行身份验证。

Docker Desktop 支持所有受信任的证书颁发机构（CA）（根证书或中间证书）。Docker 会识别存储在受信任的根证书颁发机构或中间证书颁发机构下的证书。

Docker Desktop 会根据 Windows 证书存储创建一个包含所有用户受信任 CA 的证书包，并将其附加到 Moby 的受信任证书中。因此，如果宿主机上的用户信任某个企业 SSL 证书，Docker Desktop 也会信任该证书。

要了解更多关于如何为仓库安装 CA 根证书的信息，请参阅 Docker Engine 主题中的[使用证书验证仓库客户端](/manuals/engine/security/certificates.md)。

### 如何添加客户端证书？

你可以在 `~/.docker/certs.d/<MyRegistry><Port>/client.cert` 和 `~/.docker/certs.d/<MyRegistry><Port>/client.key` 中添加客户端证书。你不需要使用 `git` 命令推送证书。

当 Docker Desktop 应用程序启动时，它会将 Windows 系统上的 `~/.docker/certs.d` 文件夹复制到 Moby（运行在 Hyper-V 上的 Docker Desktop 虚拟机）上的 `/etc/docker/certs.d` 目录。

在对密钥链或 `~/.docker/certs.d` 目录进行任何更改后，你需要重启 Docker Desktop 才能使更改生效。

仓库不能被列为不安全仓库（参见 [Docker 守护进程](/manuals/desktop/settings-and-maintenance/settings.md#docker-engine)）。Docker Desktop 会忽略不安全仓库下列出的证书，并且不会发送客户端证书。诸如 `docker run` 之类试图从仓库拉取镜像的命令会在命令行和仓库上产生错误消息。

要了解更多关于如何设置客户端 TLS 证书进行验证的信息，请参阅 Docker Engine 主题中的[使用证书验证仓库客户端](/manuals/engine/security/certificates.md)。

