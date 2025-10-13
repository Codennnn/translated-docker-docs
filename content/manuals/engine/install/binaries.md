---
description: 了解如何通过二进制方式安装 Docker。以下说明更适合测试用途。
keywords: binaries, installation, docker, documentation, linux, install docker engine
title: 通过二进制安装 Docker Engine
linkTitle: 二进制
weight: 80
aliases:
- /engine/installation/binaries/
- /engine/installation/linux/docker-ce/binaries/
- /install/linux/docker-ce/binaries/
- /installation/binaries/
---

> [!IMPORTANT]
>
> 本页介绍如何通过二进制方式安装 Docker。这些说明主要适用于测试场景。
> 我们不建议在生产环境中使用二进制方式安装 Docker，因为该方式不具备自动安全更新能力。
> 本页所述的 Linux 二进制为静态链接，这意味着构建时依赖中的漏洞
> 不会随着发行版的安全更新而自动修复。
>
> 相比包管理器或 Docker Desktop 的方式，二进制的更新也更繁琐：
> 每次 Docker 发布新版本时，你都需要手动更新本地安装版本。
>
> 另外，静态二进制可能不包含动态包所提供的全部功能。
>
> 在 Windows 与 macOS 上，我们推荐使用 [Docker Desktop](/manuals/desktop/_index.md)。
> 对于 Linux，请优先参考各发行版的专用安装说明。

如果你希望在不受支持的平台上试用 Docker 或用于测试环境，可以考虑使用静态二进制安装。
如有可能，优先使用操作系统提供的官方软件包，并通过系统的包管理器管理 Docker 的安装与升级。

Docker 守护进程的静态二进制仅适用于 Linux（`dockerd`）与 Windows（`dockerd.exe`）。
Docker 客户端的静态二进制适用于 Linux、Windows 与 macOS（均为 `docker`）。

本文讨论在 Linux、Windows 与 macOS 上使用二进制安装：

- [在 Linux 上安装守护进程与客户端二进制](#install-daemon-and-client-binaries-on-linux)
- [在 macOS 上安装客户端二进制](#install-client-binaries-on-macos)
- [在 Windows 上安装服务端与客户端二进制](#install-server-and-client-binaries-on-windows)

## 在 Linux 上安装守护进程与客户端二进制

### 先决条件

在尝试通过二进制安装 Docker 之前，请确保你的主机满足以下条件：

- 64 位操作系统
- Linux 内核版本 3.10 或更高。建议使用适用于你平台的最新内核版本。
- `iptables` 版本不低于 1.4
- `git` 版本不低于 1.7
- 可用的 `ps` 可执行文件，通常由 `procps` 或类似软件包提供
- [XZ Utils](https://tukaani.org/xz/) 版本不低于 4.9
- 正确挂载的 [`cgroupfs`](https://github.com/tianon/cgroupfs-mount/blob/master/cgroupfs-mount) 层级；
  单一的汇总式 `cgroup` 挂载点并不足够。参见 GitHub 议题
  [#2683](https://github.com/moby/moby/issues/2683)、
  [#3485](https://github.com/moby/moby/issues/3485)、
  [#4568](https://github.com/moby/moby/issues/4568)。

#### 尽可能加固你的环境

##### 操作系统注意事项

如条件允许，请启用 SELinux 或 AppArmor。

如果你的发行版支持 AppArmor 或 SELinux，建议启用其中之一。
这有助于提升安全性并阻断某些类型的攻击。请参考你的发行版文档以了解如何启用与配置 AppArmor 或 SELinux。

> **安全警告**
>
> 如果已启用上述任一安全机制，请不要为了让 Docker 或容器运行而禁用它。
> 正确的做法是调整相应配置以解决问题。

##### Docker 守护进程注意事项

- 如有可能，启用 `seccomp` 安全配置文件。参见
  [为 Docker 启用 `seccomp`](../security/seccomp.md)。

- 如有可能，启用用户命名空间。参见
  [守护进程的用户命名空间选项](/reference/cli/dockerd/#daemon-user-namespace-options)。

### 安装静态二进制

1.  下载静态二进制归档。访问
    [https://download.docker.com/linux/static/stable/](https://download.docker.com/linux/static/stable/)，
    选择你的硬件平台，并下载目标 Docker Engine 版本对应的 `.tgz` 文件。

2.  使用 `tar` 解压归档，得到 `dockerd` 与 `docker` 二进制。

    ```console
    $ tar xzvf /path/to/<FILE>.tar.gz
    ```

3.  （可选）将二进制拷贝到可执行路径（例如 `/usr/bin/`）。
    如果跳过该步骤，后续调用 `docker` 或 `dockerd` 时需要提供完整路径。

    ```console
    $ sudo cp docker/* /usr/bin/
    ```

4.  启动 Docker 守护进程：

    ```console
    $ sudo dockerd &
    ```

    如需使用额外选项启动守护进程，可按需修改以上命令，或创建并编辑 `/etc/docker/daemon.json` 添加自定义配置。

5.  通过运行 `hello-world` 镜像验证安装是否成功。

    ```console
    $ sudo docker run hello-world
    ```

    该命令会下载一个测试镜像并在容器中运行。容器运行后会打印一条消息，然后退出。

至此，你已经成功安装并启动了 Docker Engine。

{{% include "root-errors.md" %}}

## 在 macOS 上安装客户端二进制

> [!NOTE]
>
> 以下说明主要适用于测试场景。macOS 提供的二进制仅包含 Docker 客户端，
> 不包含用于运行容器的 `dockerd` 守护进程。因此，我们推荐安装
> [Docker Desktop](/manuals/desktop/_index.md)。

Mac 的二进制还不包含：

- 运行时环境。你需要在虚拟机或远程 Linux 主机上准备一个可用的引擎。
- `buildx` 与 `docker compose` 等 Docker 组件。

安装客户端二进制：

1.  下载静态二进制归档。访问
    [https://download.docker.com/mac/static/stable/](https://download.docker.com/mac/static/stable/)，选择 `x86_64`（Intel 芯片）或 `aarch64`（Apple silicon），
    然后下载目标 Docker Engine 版本对应的 `.tgz` 文件。

2.  使用 `tar` 解压归档，得到 `docker` 二进制。

    ```console
    $ tar xzvf /path/to/<FILE>.tar.gz
    ```

3.  清理扩展属性以允许执行：

    ```console
    $ sudo xattr -rc docker
    ```

    现在运行以下命令，你应能看到 Docker CLI 的使用说明：

    ```console
    $ docker/docker
    ```

4.  （可选）将二进制移动到可执行路径（如 `/usr/local/bin/`）。如果跳过该步骤，调用 `docker` 或 `dockerd` 时需要提供完整路径。

    ```console
    $ sudo cp docker/docker /usr/local/bin/
    ```

5.  通过运行 `hello-world` 镜像验证安装是否成功。`<hostname>` 的值应为可被客户端访问、且正在运行 Docker 守护进程的主机名或 IP 地址。

    ```console
    $ sudo docker -H <hostname> run hello-world
    ```

    该命令会下载测试镜像并在容器中运行。容器运行后会打印消息并退出。

## 在 Windows 上安装服务端与客户端二进制

> [!NOTE]
>
> 本节介绍如何在 Windows Server 上安装 Docker 守护进程（仅支持运行 Windows 容器）。
> 在 Windows Server 上安装的守护进程不包含 `buildx` 与 `compose` 等 Docker 组件。
> 若你使用的是 Windows 10 或 11，推荐安装 [Docker Desktop](/manuals/desktop/_index.md)。

Windows 的二进制包包含 `dockerd.exe` 与 `docker.exe`。
在 Windows 上，这些二进制仅支持运行原生 Windows 容器（不支持 Linux 容器）。

安装服务端与客户端二进制：

1. Download the static binary archive. Go to
    [https://download.docker.com/win/static/stable/x86_64](https://download.docker.com/win/static/stable/x86_64) and select the latest version from the list.

2. 运行以下 PowerShell 命令，将归档解压并安装到程序目录：

    ```powershell
    PS C:\> Expand-Archive /path/to/<FILE>.zip -DestinationPath $Env:ProgramFiles
    ```

3. 注册服务并启动 Docker Engine：

    ```powershell
    PS C:\> &$Env:ProgramFiles\Docker\dockerd --register-service
    PS C:\> Start-Service docker
    ```

4.  通过运行 `hello-world` 镜像验证安装是否成功。

    ```powershell
    PS C:\> &$Env:ProgramFiles\Docker\docker run hello-world:nanoserver
    ```

    该命令会下载测试镜像并在容器中运行。容器运行后会打印消息并退出。

## 升级静态二进制

要升级通过二进制手动安装的 Docker Engine，请先停止本地正在运行的 `dockerd` 或 `dockerd.exe` 进程，
然后按照常规安装步骤在原有版本之上安装新版本。

## 进一步阅读

- 继续阅读 [Linux 安装后的步骤](linux-postinstall.md)。
