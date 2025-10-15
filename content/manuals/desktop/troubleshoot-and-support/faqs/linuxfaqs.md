---
description: Docker Desktop for Linux 常见问题
keywords: desktop, linux, faqs
title: Docker Desktop for Linux 常见问题
linkTitle: Linux
tags: [FAQ]
aliases:
- /desktop/linux/space/
- /desktop/faqs/linuxfaqs/
weight: 40
---

### 为什么 Docker Desktop for Linux 要运行虚拟机？

Docker Desktop for Linux 运行虚拟机（VM）主要基于以下原因：

1. 确保 Docker Desktop 在各平台提供一致的体验。

    在用户调研中，Linux 用户选择 Docker Desktop 最常见的原因是希望在所有主流操作系统上获得一致的功能体验。使用虚拟机可以确保 Linux 用户获得与 Windows 和 macOS 相近的 Docker Desktop 使用体验。

2. 充分利用新的内核特性。

    有时我们需要使用新的操作系统特性。由于我们可以控制虚拟机内的内核和操作系统，因此能够立即向所有用户推出这些新特性，即使是那些刻意使用 LTS 版本系统的用户也能享受到。

3. 增强安全性。

    容器镜像的漏洞会对宿主环境构成安全风险。网上存在大量未经已知漏洞验证的非官方镜像。恶意用户可以将镜像推送到公共仓库，并通过各种手段诱骗用户拉取并运行这些镜像。虚拟机方案可以缓解这一威胁，因为任何获得 root 权限的恶意软件都被限制在虚拟机环境中，无法访问宿主机。

    为什么不使用 rootless Docker？虽然这种方式表面上限制了 root 用户访问，让"top"命令看起来更安全，但它允许非特权用户在自己的用户命名空间中获得 `CAP_SYS_ADMIN` 能力，并访问那些本不应被非特权用户使用的内核 API，从而导致[安全漏洞](https://www.openwall.com/lists/oss-security/2022/01/18/7)。

4. 在保证功能一致性和安全性的同时，将对性能的影响降到最低。

    Docker Desktop for Linux 使用的虚拟机采用了 [`VirtioFS`](https://virtio-fs.gitlab.io)，这是一个共享文件系统，允许虚拟机访问宿主机上的目录树。我们的内部基准测试表明，通过为虚拟机分配合适的资源，VirtioFS 可以实现接近原生的文件系统性能。

    因此，我们调整了 Docker Desktop for Linux 中虚拟机的默认内存大小。你可以在 Docker Desktop 的 **设置** > **资源** 标签页中使用 **内存** 滑块，根据具体需求调整此设置。

### 如何启用文件共享？

Docker Desktop for Linux 使用 [VirtioFS](https://virtio-fs.gitlab.io/) 作为默认（目前也是唯一）的文件共享机制，用于在宿主机和 Docker Desktop 虚拟机之间共享文件。

{{< accordion title="Docker Desktop 4.34 及更早版本的补充信息" >}}

为了在不需要提升权限的同时，又不过度限制对共享文件的操作，Docker Desktop 在用户命名空间（参见 `user_namespaces(7)`）内运行文件共享服务（`virtiofsd`），并配置了 UID 和 GID 映射。因此，Docker Desktop 需要宿主机配置为允许当前用户使用从属 ID 委派。为此，必须存在 `/etc/subuid`（参见 `subuid(5)`）和 `/etc/subgid`（参见 `subgid(5)`）文件。Docker Desktop 仅支持通过文件配置的从属 ID 委派。Docker Desktop 将当前用户的 ID 和 GID 映射为容器中的 0。它使用 `/etc/subuid` 和 `/etc/subgid` 中与当前用户对应的第一个条目来设置容器中大于 0 的 ID 映射。

| 容器内的 ID | 宿主机上的 ID                                                                       |
| --------------- | -------------------------------------------------------------------------------- |
| 0 (root)        | 运行 Docker Desktop 的用户 ID（例如 1000）                                            |
| 1               | 0 + `/etc/subuid`/`/etc/subgid` 中指定的 ID 范围起始值（例如 100000） |
| 2               | 1 + `/etc/subuid`/`/etc/subgid` 中指定的 ID 范围起始值（例如 100001） |
| 3               | 2 + `/etc/subuid`/`/etc/subgid` 中指定的 ID 范围起始值（例如 100002） |
| ...             | ...                                                                              |

如果缺少 `/etc/subuid` 和 `/etc/subgid` 文件，则需要创建它们。两个文件都应该包含以下格式的条目：`<用户名>:<id 范围起始值>:<id 范围大小>`。例如，要允许当前用户使用从 100000 到 165535 的 ID：

```console
$ grep "$USER" /etc/subuid >> /dev/null 2&>1 || (echo "$USER:100000:65536" | sudo tee -a /etc/subuid)
$ grep "$USER" /etc/subgid >> /dev/null 2&>1 || (echo "$USER:100000:65536" | sudo tee -a /etc/subgid)
```

要验证配置是否正确创建，可以检查文件内容：

```console
$ echo $USER
exampleuser
$ cat /etc/subuid
exampleuser:100000:65536
$ cat /etc/subgid
exampleuser:100000:65536
```

在这种情况下，如果在 Docker Desktop 容器内对一个共享文件执行 `chown`，将其所有者设置为 UID 为 1000 的用户，那么在宿主机上该文件会显示为属于 UID 为 100999 的用户。这会导致一个不便之处：在宿主机上难以访问此类文件。解决方法是创建一个具有新 GID 的组并将用户添加到该组，或者为与 Docker Desktop 虚拟机共享的文件夹设置递归 ACL（参见 `setfacl(1)`）。

{{< /accordion >}}

### Docker Desktop 在哪里存储 Linux 容器？

Docker Desktop 将 Linux 容器和镜像存储在 Linux 文件系统中的一个大型"磁盘镜像"文件中。这与 Linux 上的 Docker 不同，后者通常将容器和镜像存储在宿主机文件系统的 `/var/lib/docker` 目录中。

#### 磁盘镜像文件在哪里？

要查找磁盘镜像文件的位置，请在 Docker Desktop 仪表板中选择 **设置**，然后在 **资源** 标签页中选择 **高级**。

**高级** 标签页会显示磁盘镜像的位置，以及磁盘镜像的最大大小和实际占用的空间。请注意，其他工具可能会以最大文件大小而非实际文件大小来显示文件的空间使用情况。

##### 如果文件太大怎么办？

如果磁盘镜像文件过大，你可以：

- 将其移动到更大的磁盘
- 删除不必要的容器和镜像
- 减小文件的最大允许大小

##### 如何将文件移动到更大的磁盘？

要将磁盘镜像文件移动到其他位置：

1. 选择 **设置**，然后在 **资源** 标签页中选择 **高级**。

2. 在 **磁盘镜像位置** 部分，选择 **浏览** 并为磁盘镜像选择新位置。

3. 选择 **应用** 以使更改生效。

不要直接在 Finder 中移动文件，因为这可能导致 Docker Desktop 无法跟踪该文件。

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

根据磁盘镜像文件的格式，回收宿主机上的空间可能需要几分钟：

- 如果文件名为 `Docker.raw`：宿主机上的空间应该会在几秒钟内回收。
- 如果文件名为 `Docker.qcow2`：空间将在几分钟后由后台进程释放。

只有在删除镜像时才会释放空间。在运行的容器内删除文件时，空间不会自动释放。要在任何时候触发空间回收，可以运行以下命令：

```console
$ docker run --privileged --pid=host docker/desktop-reclaim-space
```

请注意，许多工具报告的是最大文件大小，而不是实际文件大小。要从终端查询宿主机上文件的实际大小，可以运行：

```console
$ cd ~/.docker/desktop/vms/0/data
$ ls -klsh Docker.raw
2333548 -rw-r--r--@ 1 username  staff    64G Dec 13 17:42 Docker.raw
```

在这个例子中，磁盘的实际大小是 `2333548` KB，而磁盘的最大大小是 `64` GB。

##### 如何减小文件的最大大小？

要减小磁盘镜像文件的最大大小：

1. 在 Docker Desktop 仪表板中选择 **设置**，然后在 **资源** 标签页中选择 **高级**。

2. **磁盘镜像大小** 部分包含一个滑块，允许你更改磁盘镜像的最大大小。调整滑块以设置更低的限制。

3. 选择 **应用**。

当你减小最大大小时，当前的磁盘镜像文件将被删除，因此所有容器和镜像都会丢失。
