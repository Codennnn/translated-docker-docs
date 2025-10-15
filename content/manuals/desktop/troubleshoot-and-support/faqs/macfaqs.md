---
description: Docker Desktop for Mac 常见问题
keywords: desktop, mac, faqs
title: Docker Desktop for Mac 常见问题
linkTitle: Mac
tags: [FAQ]
aliases:
- /desktop/mac/space/
- /docker-for-mac/space/
- /desktop/faqs/macfaqs/
weight: 20
---

### 什么是 HyperKit？

HyperKit 是一个基于 macOS Hypervisor.framework 构建的虚拟化监视器。它完全在用户空间运行，没有其他依赖。

Docker 使用 HyperKit 来消除对其他虚拟机产品的需求，例如 Oracle VirtualBox 或 VMware Fusion。

### HyperKit 有什么优势？

HyperKit 比 VirtualBox 和 VMware Fusion 更加轻量，并且内置版本专门针对 Mac 上的 Docker 工作负载进行了定制优化。

### Docker Desktop 在哪里存储 Linux 容器和镜像？

Docker Desktop 将 Linux 容器和镜像存储在 Mac 文件系统中的一个大型"磁盘镜像"文件中。这与 Linux 上的 Docker 不同，后者通常将容器和镜像存储在 `/var/lib/docker` 目录中。

#### 磁盘镜像文件在哪里？

要查找磁盘镜像文件的位置，请在 Docker Desktop 仪表板中选择 **设置**，然后在 **资源** 标签页中选择 **高级**。

**高级** 标签页会显示磁盘镜像的位置，以及磁盘镜像的最大大小和实际占用的空间。请注意，其他工具可能会以最大文件大小而非实际文件大小来显示文件的空间使用情况。

#### 如果文件太大怎么办？

如果磁盘镜像文件过大，你可以：

- 将其移动到更大的磁盘
- 删除不必要的容器和镜像
- 减小文件的最大允许大小

##### 如何将文件移动到更大的磁盘？

要将磁盘镜像文件移动到其他位置：

1. 选择 **设置**，然后在 **资源** 标签页中选择 **高级**。

2. 在 **磁盘镜像位置** 部分，选择 **浏览** 并为磁盘镜像选择新位置。

3. 选择 **应用** 以使更改生效。

> [!IMPORTANT]
>
> 不要直接在 Finder 中移动文件，因为这可能导致 Docker Desktop 无法跟踪该文件。

##### 如何删除不必要的容器和镜像？

检查是否存在不必要的容器和镜像。如果你的客户端和守护进程 API 运行的是 1.25 或更高版本（在客户端使用 `docker version` 命令检查客户端和守护进程的 API 版本），你可以通过运行以下命令查看详细的空间使用信息：

```console
$ docker system df -v
```

或者，要列出镜像，可以运行：

```console
$ docker image ls
```

要列出容器，可以运行：

```console
$ docker container ls -a
```

如果存在大量冗余对象，可以运行以下命令：

```console
$ docker system prune
```

此命令会移除所有已停止的容器、未使用的网络、悬空镜像和构建缓存。

根据磁盘镜像文件的格式，回收宿主机上的空间可能需要几分钟。如果文件名为：

- `Docker.raw`：宿主机上的空间会在几秒钟内回收。
- `Docker.qcow2`：空间将在几分钟后由后台进程释放。

只有在删除镜像时才会释放空间。在运行的容器内删除文件时，空间不会自动释放。要在任何时候触发空间回收，可以运行以下命令：

```console
$ docker run --privileged --pid=host docker/desktop-reclaim-space
```

请注意，许多工具报告的是最大文件大小，而不是实际文件大小。要从终端查询宿主机上文件的实际大小，可以运行：

```console
$ cd ~/Library/Containers/com.docker.docker/Data/vms/0/data
$ ls -klsh Docker.raw
2333548 -rw-r--r--@ 1 username  staff    64G Dec 13 17:42 Docker.raw
```

在这个例子中，磁盘的实际大小是 `2333548` KB，而磁盘的最大大小是 `64` GB。

##### 如何减小文件的最大大小？

要减小磁盘镜像文件的最大大小：

1. 选择 **设置**，然后在 **资源** 标签页中选择 **高级**。

2. **磁盘镜像大小** 部分包含一个滑块，允许你更改磁盘镜像的最大大小。调整滑块以设置更低的限制。

3. 选择 **应用**。

当你减小最大大小时，当前的磁盘镜像文件将被删除，因此所有容器和镜像都会丢失。

### 如何添加 TLS 证书？

你可以向 Docker 守护进程添加受信任的证书颁发机构（CA）（用于验证仓库服务器证书）和客户端证书（用于向仓库进行身份验证）。

#### 添加自定义 CA 证书（服务器端）

支持所有受信任的 CA（根证书或中间证书）。Docker Desktop 会根据 Mac 钥匙串创建一个包含所有用户受信任 CA 的证书包，并将其附加到 Moby 的受信任证书中。因此，如果宿主机上的用户信任某个企业 SSL 证书，Docker Desktop 也会信任该证书。

要手动添加自定义的自签名证书，首先需要将证书添加到 macOS 钥匙串中，Docker Desktop 会自动获取。以下是一个示例：

```console
$ sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ca.crt
```

或者，如果你只想将证书添加到自己的本地钥匙串（而不是所有用户），可以运行以下命令：

```console
$ security add-trusted-cert -d -r trustRoot -k ~/Library/Keychains/login.keychain ca.crt
```

另请参阅[证书的目录结构](#directory-structures-for-certificates)。

> [!NOTE]
>
> 在对钥匙串或 `~/.docker/certs.d` 目录进行任何更改后，你需要重启 Docker Desktop 才能使更改生效。

有关如何执行此操作的完整说明，请参阅博客文章 [Adding Self-signed Registry Certs to Docker & Docker Desktop for Mac](https://blog.container-solutions.com/adding-self-signed-registry-certs-docker-mac)。

#### 添加客户端证书

你可以将客户端证书放在 `~/.docker/certs.d/<MyRegistry>:<Port>/client.cert` 和 `~/.docker/certs.d/<MyRegistry>:<Port>/client.key` 中。

当 Docker Desktop 应用程序启动时，它会将 Mac 上的 `~/.docker/certs.d` 文件夹复制到 Moby（Docker Desktop 的 `xhyve` 虚拟机）上的 `/etc/docker/certs.d` 目录。

> [!NOTE]
>
> * 在对钥匙串或 `~/.docker/certs.d` 目录进行任何更改后，你需要重启 Docker Desktop 才能使更改生效。
>
> * 仓库不能被列为_不安全仓库_。Docker Desktop 会忽略不安全仓库下列出的证书，并且不会发送客户端证书。诸如 `docker run` 之类试图从仓库拉取镜像的命令会在命令行和仓库上产生错误消息。

#### 证书的目录结构

如果你有以下目录结构，则无需手动将 CA 证书添加到 Mac OS 系统登录中：

```text
/Users/<user>/.docker/certs.d/
└── <MyRegistry>:<Port>
   ├── ca.crt
   ├── client.cert
   └── client.key
```

以下进一步展示和解释了使用自定义证书的配置：

```text
/etc/docker/certs.d/        <-- 证书目录
└── localhost:5000          <-- 主机名:端口
   ├── client.cert          <-- 客户端证书
   ├── client.key           <-- 客户端密钥
   └── ca.crt               <-- 签署仓库证书的证书颁发机构
```

只要 CA 证书也在你的钥匙串中，你也可以使用以下目录结构：

```text
/Users/<user>/.docker/certs.d/
└── <MyRegistry>:<Port>
    ├── client.cert
    └── client.key
```

要了解更多关于如何为仓库安装 CA 根证书以及如何设置客户端 TLS 证书进行验证的信息，请参阅 Docker Engine 主题中的[使用证书验证仓库客户端](/manuals/engine/security/certificates.md)。
