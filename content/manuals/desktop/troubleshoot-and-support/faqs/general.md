---
description: 适用于所有平台的 Docker Desktop 常见问题
keywords: desktop, mac, windows, faqs
title: Desktop 通用常见问题
linkTitle: 通用
tags: [FAQ]
aliases:
- /mackit/faqs/
- /docker-for-mac/faqs/
- /docker-for-windows/faqs/
- /desktop/faqs/
- /desktop/faqs/general/
weight: 10
---

### 可以离线使用 Docker Desktop 吗？

可以。你可以在离线环境下使用 Docker Desktop，但无法访问需要联网的功能。此外，在离线或网络隔离环境中使用时，所有需要登录的功能都将无法使用。

这些功能包括：

- [学习中心](/manuals/desktop/use-desktop/_index.md)的资源
- 从 Docker Hub 拉取或推送镜像
- [镜像访问管理](/manuals/security/access-tokens.md)
- [静态漏洞扫描](/manuals/docker-hub/repos/manage/vulnerability-scanning.md)
- 在 Docker Dashboard 中查看远程镜像
- 使用 [BuildKit](/manuals/build/buildkit/_index.md#getting-started) 时的 Docker Build。
  你可以通过禁用 BuildKit 来绕过此限制，运行 `DOCKER_BUILDKIT=0 docker build .` 即可。
- [Kubernetes](/manuals/desktop/features/kubernetes.md)（首次启用 Kubernetes 时需要下载镜像）
- 检查更新
- [应用内诊断](/manuals/desktop/troubleshoot-and-support/troubleshoot/_index.md#diagnose-from-the-app)（包括[自我诊断工具](/manuals/desktop/troubleshoot-and-support/troubleshoot/_index.md#diagnose-from-the-app)）
- 发送使用统计信息
- 当 `networkMode` 设置为 `mirrored` 时

### 如何连接到远程 Docker Engine API？

要连接到远程 Engine API，你可能需要为 Docker 客户端和开发工具提供 Engine API 的位置。

Mac 和 Windows WSL 2 用户可以通过 Unix 套接字连接到 Docker Engine：`unix:///var/run/docker.sock`。

如果你使用的应用程序（如 [Apache Maven](https://maven.apache.org/)）需要配置 `DOCKER_HOST` 和 `DOCKER_CERT_PATH` 环境变量，请设置这些变量以通过 Unix 套接字连接到 Docker 实例。

例如：

```console
$ export DOCKER_HOST=unix:///var/run/docker.sock
```

Windows 版 Docker Desktop 用户可以通过**命名管道**连接到 Docker Engine：`npipe:////./pipe/docker_engine`，或通过 **TCP 套接字**：`tcp://localhost:2375`。

详细信息请参阅 [Docker Engine API](/reference/api/engine/_index.md)。

### 如何从容器连接到宿主机上的服务？

宿主机的 IP 地址可能会变化，或者在没有网络访问时根本没有 IP。建议你使用特殊的 DNS 名称 `host.docker.internal` 进行连接，它会解析为宿主机使用的内部 IP 地址。

更多信息和示例，请参阅[如何从容器连接到宿主机上的服务](/manuals/desktop/features/networking.md#i-want-to-connect-from-a-container-to-a-service-on-the-host)。

### 可以将 USB 设备直通到容器吗？

Docker Desktop 不支持直接的 USB 设备直通。不过，你可以使用 USB over IP 技术将常见的 USB 设备连接到 Docker Desktop 虚拟机，然后转发给容器。详细信息请参阅[在 Docker Desktop 中使用 USB/IP](/manuals/desktop/features/usbip.md)。

### 如何在没有管理员权限的情况下运行 Docker Desktop？

Docker Desktop 只在安装时需要管理员权限。安装完成后，运行时不需要管理员权限。但是，为了让非管理员用户能够运行 Docker Desktop，必须在安装时使用特定的安装标志，并满足一些因平台而异的前置条件。

{{< tabs >}}
{{< tab name="Mac" >}}

要在 Mac 上以非管理员权限运行 Docker Desktop，需通过命令行安装并传递 `—user=<userid>` 安装标志：

```console
$ /Applications/Docker.app/Contents/MacOS/install --user=<userid>
```

然后，使用指定的用户 ID 登录到你的机器，并启动 Docker Desktop。

> [!NOTE]
> 
> 在启动 Docker Desktop 之前，如果 `~/Library/Group Containers/group.com.docker/` 目录中已经存在 `settings-store.json` 文件（或者对于 Docker Desktop 4.34 及更早版本为 `settings.json`），你会看到一个**完成 Docker Desktop 设置**窗口，在你选择**完成**时会提示需要管理员权限。为了避免这种情况，请确保在启动应用程序之前删除之前安装遗留的 `settings-store.json` 文件（或者对于 Docker Desktop 4.34 及更早版本为 `settings.json`）。

{{< /tab >}}
{{< tab name="Windows" >}}

> [!NOTE]
>
> 如果你使用的是 WSL 2 后端，请先确保满足 WSL 2 的[最低版本要求](/manuals/desktop/features/wsl/best-practices.md)。否则，请先更新 WSL 2。

要在 Windows 上以非管理员权限运行 Docker Desktop，需通过命令行安装并传递 `—always-run-service` 安装标志。

```console
$ "Docker Desktop Installer.exe" install —always-run-service
```

{{< /tab >}}
{{< /tabs >}}


