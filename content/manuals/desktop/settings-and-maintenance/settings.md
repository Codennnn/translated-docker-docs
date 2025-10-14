---
description: 了解并更改 Docker Desktop 的设置
keywords: settings, preferences, proxy, file sharing, resources, kubernetes, Docker
  Desktop, Linux, Mac, Windows
title: 更改 Docker Desktop 设置
linkTitle: 更改设置
aliases:
 - /desktop/settings/mac/
 - /desktop/settings/windows/
 - /desktop/settings/linux/
 - /desktop/settings/
weight: 10
---

进入 **Settings（设置）** 的方式：

- 在 Docker 菜单 {{< inline-image src="../images/whale-x.svg" alt="whale menu" >}} 中选择 **Settings**
- 在 Docker Desktop 仪表板中选择 **Settings** 图标

你也可以在以下位置找到 `settings-store.json` 文件（Docker Desktop 4.34 及更早版本为 `settings.json`）：
 - Mac: `~/Library/Group\ Containers/group.com.docker/settings-store.json`
 - Windows: `C:\Users\[USERNAME]\AppData\Roaming\Docker\settings-store.json`
 - Linux: `~/.docker/desktop/settings-store.json`

## 常规

在 **General** 选项卡中，你可以设置 Docker 的启动时机并配置其他常用选项：

- **Start Docker Desktop when you sign in to your computer**：登录电脑时自动启动 Docker Desktop。

- **Open Docker Dashboard when Docker Desktop starts**：启动 Docker Desktop 时自动打开仪表板（Dashboard）。

- **Choose theme for Docker Desktop**：选择 **Light** 或 **Dark** 主题，或设置为 **Use system settings**（跟随系统）。

- **Configure shell completions**：自动修改你的 shell 配置，提供命令、参数和 Docker 对象（如容器与卷名）的 Tab 补全。详见 [Completion](/manuals/engine/cli/completion.md)。

- **Choose container terminal**：选择从容器打开终端时启动的终端类型。若选择集成终端，可直接在 Docker Desktop 仪表板中对运行中的容器执行命令。详见 [探索容器](/manuals/desktop/use-desktop/container.md)。

- **Enable Docker terminal**：在 Docker Desktop 中直接与宿主机交互并执行命令。

- **Enable Docker Debug by default**：默认在集成终端中启用 Docker Debug。详见[探索容器](/manuals/desktop/use-desktop/container.md#integrated-terminal)。

- {{< badge color=blue text="Mac only" >}}**Include VM in Time Machine backups**：将 Docker Desktop 虚拟机纳入 Time Machine 备份（默认关闭）。

- **Use containerd for pulling and storing images**：启用 containerd 镜像存储。可通过懒加载加速容器启动，并支持在 Docker 中运行 Wasm 应用。详见 [containerd 镜像存储](/manuals/desktop/features/containerd.md)。

- {{< badge color=blue text="Windows only" >}}**Expose daemon on tcp://localhost:2375 without TLS**：在无 TLS 的情况下暴露 Docker 守护进程，便于旧版客户端连接。请谨慎使用，此设置可能带来远程代码执行风险。

- {{< badge color=blue text="Windows only" >}}**Use the WSL 2 based engine**：使用 WSL 2 引擎，性能优于 Hyper-V。详见 [WSL 2 后端](/manuals/desktop/features/wsl/_index.md)。

- {{< badge color=blue text="Windows only" >}}**Add the `*.docker.internal` names to the host's `/etc/hosts` file (Password required)**：将 `*.docker.internal` 名称写入宿主机 `/etc/hosts`，便于宿主机与容器解析这些 DNS 名称。

- {{< badge color=blue text="Mac only" >}} **Choose Virtual Machine Manager (VMM)**：选择用于创建与管理 Docker Desktop Linux VM 的虚拟机管理器。
  - 选择 **Docker VMM** 可获得最新且性能最佳的管理器（仅适用于运行 macOS 12.5 及以上版本的 Apple 芯片 Mac，当前为 Beta）。
    > [!TIP]
    >
    > 开启该设置可使 Docker Desktop 跑得更快。
  - 也可选择 **Apple Virtualization framework**、**QEMU**（Apple 芯片，Docker Desktop 4.43 及更早版本）或 **HyperKit**（Intel Mac）。在 macOS 12.5 及以上，默认使用 Apple Virtualization framework。

   详情参见 [Virtual Machine Manager](/manuals/desktop/features/vmm.md)。

- {{< badge color=blue text="Mac only" >}}**Choose file sharing implementation for your containers**：选择文件共享实现（**VirtioFS**、**gRPC FUSE** 或 **osxfs（Legacy）**）。VirtioFS 仅适用于 macOS 12.5 及以上，且默认启用。
    > [!TIP]
    >
    > 优先使用 VirtioFS 以加速文件共享。它可将文件系统操作时间缩短[最高达 98%](https://github.com/docker/roadmap/issues/7#issuecomment-1044452206)。同时，它是 Docker VMM 唯一支持的文件共享实现。

- {{< badge color=blue text="Mac only" >}}**Use Rosetta for x86_64/amd64 emulation on Apple Silicon**：在 Apple 芯片上启用 Rosetta 加速 x86/AMD64 二进制仿真。仅当 VMM 选择 **Apple Virtualization framework** 且系统为 macOS 13 或更高时可用。

- **Send usage statistics**：允许 Docker Desktop 发送诊断、崩溃报告与使用数据，用于改进和故障排查。取消勾选即可退出。Docker 可能会不定期请求更多信息。

- **Use Enhanced Container Isolation**：启用增强容器隔离，防止容器突破 Linux VM 边界以提升安全性。详见[增强容器隔离](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/_index.md)。
    > [!NOTE]
    >
    > 仅当你已登录 Docker Desktop 且订阅了 Docker Business 时可用。

- **Show CLI hints**：在命令行运行 Docker 命令时显示提示（默认开启）。也可通过设置环境变量 `DOCKER_CLI_HINTS=true|false` 开关。

- **Enable Scout image analysis**：开启后，在 Docker Desktop 中查看镜像时会显示 **Start analysis** 按钮，点击即可用 Docker Scout 分析镜像。

- **Enable background SBOM indexing**：开启后，Docker Scout 会自动分析你构建或拉取的镜像。

- {{< badge color=blue text="Mac only" >}}**Automatically check configuration**：定期检查你的配置，避免被其他应用意外修改。

  Docker Desktop 会检测安装时的配置是否被诸如 Orbstack 等外部应用更改，包括：
    - Docker 二进制到 `/usr/local/bin` 的符号链接
    - 默认 Docker 套接字的符号链接 
  另外，Docker Desktop 会在启动时确保上下文切换到 `desktop-linux`。
  
  如果检测到变更，你会收到通知并可一键恢复配置。详见 [常见问题](/manuals/desktop/troubleshoot-and-support/faqs/macfaqs.md#why-do-i-keep-getting-a-notification-telling-me-an-application-has-changed-my-desktop-configurations)。

## 资源

在 **Resources** 选项卡中，你可以配置 CPU、内存、磁盘、代理、网络等资源。

### 高级

> [!NOTE]
>
> 在 Windows 上，**Advanced** 里的 **Resource allocation**（资源分配）仅在 Hyper-V 模式可用，因为在 WSL 2 模式和 Windows 容器模式下，资源由 Windows 管理。在 WSL 2 模式下，你可以为 [WSL 2 实用 VM](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configure-global-options-with-wslconfig) 配置内存、CPU 与交换空间上限。

在 **Advanced** 中，你可以限制分配给 Docker Linux VM 的资源。

可配置项包括：

- **CPU limit**：设定 Docker Desktop 可用的 CPU 最大数量。默认使用宿主机的全部可用处理器。

- **Memory limit**：默认最多使用宿主机内存的 50%。调大即增加可用内存，调小则减少。

- **Swap**：根据需要配置交换文件大小，默认 1 GB。

- **Disk usage limit**：限制引擎可用的磁盘空间上限。

- **Disk image location**：指定用于存放容器与镜像的 Linux 卷位置。

  你也可以将磁盘镜像移动到其他位置。若目标位置已有镜像，系统会提示你选择使用现有镜像或进行替换。

>[!TIP]
>
> 如果你感觉 Docker Desktop 变慢，或需要运行多容器工作负载，可适当增加内存与磁盘镜像空间配额。

- **Resource Saver**：启用或禁用[资源节省模式](/manuals/desktop/use-desktop/resource-saver.md)。该模式在 Docker Desktop 空闲（即无容器运行）时自动关闭 Linux VM，从而显著降低宿主机的 CPU 与内存占用。

  你可以配置资源节省模式的超时时间，即 Docker Desktop 空闲多久开始生效。默认 5 分钟。

  > [!NOTE]
  >
  > 当有容器运行时会自动退出资源节省模式。退出过程可能需要几秒（约 3–10 秒），因为 Docker Desktop 需要重启 Linux VM。


### 文件共享

> [!NOTE]
>
> 在 Windows 上，**File sharing** 仅在 Hyper-V 模式可用；在 WSL 2 与 Windows 容器模式下，文件会自动共享。

通过文件共享，你可以将本机目录共享给 Linux 容器。这对“在宿主机 IDE 中编辑源码、并在容器中运行与测试”尤为有用。

#### 同步文件共享 

同步文件共享是一种替代的文件共享机制，通过同步的文件系统缓存提升主机到 VM 的文件共享与绑定挂载性能。该功能适用于 Pro、Team 与 Business 订阅。

了解更多，参见 [Synchronized file share](/manuals/desktop/features/synchronized-file-sharing.md)。

#### 虚拟文件共享

默认会共享 `/Users`、`/Volumes`、`/private`、`/tmp` 与 `/var/folders` 目录。
若项目不在上述目录内，需要手动添加到共享列表；否则运行时可能出现 `Mounts denied` 或 `cannot start service` 错误。

文件共享相关设置包括：

- **Add a Directory**：点击 `+` 并选择要添加的目录。

- **Remove a Directory**：点击要移除目录旁的 `-`。

- **Apply**：应用后，该目录即可通过 Docker 绑定挂载（`-v`）供容器使用。

> [!TIP]
>
> * 仅共享容器所需目录。文件共享会带来额外开销：宿主机文件的每次变更都需同步到 Linux VM。共享过多文件可能导致高 CPU 占用与文件系统性能下降。
> * 共享文件夹旨在支持“在宿主机编辑应用代码，在容器中运行”。对于缓存目录、数据库等非代码数据，建议存放在 Linux VM 中，并使用[数据卷](/manuals/engine/storage/volumes.md)（命名卷）或[数据容器](/manuals/engine/storage/volumes.md)以获得更佳性能。
> * 若将整个用户主目录共享进容器，macOS 可能会提示授予对提醒事项、下载等个人区域的访问权限。
> * 默认情况下，Mac 文件系统不区分大小写，而 Linux 区分。在 Linux 中可以存在 `test` 与 `Test` 两个文件，但在 Mac 上它们会指向同一底层文件。这可能导致应用在开发者机器（共享文件内容）上正常，而在 Linux 生产环境（文件内容区分）失败。为避免此问题，Docker Desktop 要求以原始大小写访问共享文件。例如创建了 `test`，必须以 `test` 打开；以 `Test` 打开会报 “No such file or directory”。同理，已有 `test` 时创建 `Test` 也会失败。
>
> 详见[将项目目录挂载为卷时，如位于 `/Users` 之外需开启文件共享](/manuals/desktop/troubleshoot-and-support/troubleshoot/topics.md)

#### 按需共享文件夹

在 Windows 上，当某个容器首次使用特定文件夹时，你可以“按需”共享该文件夹。

当你在 Shell 中执行带卷挂载的 Docker 命令（如下例）或启动包含卷挂载的 Compose 文件时，系统会弹窗询问是否共享该文件夹。

选择 **Share it** 后，该文件夹会加入 Docker Desktop 的共享列表并供容器使用。也可以选择 **Cancel** 拒绝共享。

![Shared folder on demand](../images/shared-folder-on-demand.png)

### 代理

Docker Desktop 支持使用 HTTP/HTTPS 与 [SOCKS5 代理](/manuals/desktop/features/networking.md#socks5-proxy-support)。

HTTP/HTTPS 代理适用于：

- 登录 Docker
- 拉取或推送镜像
- 构建镜像期间获取制品
- 容器访问外部网络
- 扫描镜像

若宿主机已配置 HTTP/HTTPS 代理（静态配置或 PAC），Docker Desktop 会自动读取并用于登录、拉取/推送镜像以及容器联网。若代理需要认证，Docker Desktop 会动态提示输入用户名和密码，且将密码安全保存在操作系统的凭据存储中。当前仅支持 `Basic` 认证方式，建议使用 `https://` 代理地址以保护传输中的密码。Docker Desktop 在与代理通信时支持 TLS 1.3。

如需为 Docker Desktop 设置不同代理，开启 **Manual proxy configuration**，并填写一个上游代理 URL，格式为 `http://proxy:port` 或 `https://proxy:port`。

为防止代理设置被意外修改，参见[设置管理](/manuals/enterprise/security/hardened-desktop/settings-management/_index.md#what-features-can-i-configure-with-settings-management)。

镜像扫描使用的 HTTPS 代理可通过环境变量 `HTTPS_PROXY` 配置。

> [!NOTE]
>
> 若使用托管在 Web 服务器上的 PAC 文件，请确保为 `.pac` 扩展名配置 MIME 类型 `application/x-ns-proxy-autoconfig`，否则可能无法正确解析。关于 PAC 与 Docker Desktop 的更多细节，参见 [Hardened Docker Desktop](/manuals/enterprise/security/hardened-desktop/air-gapped-containers.md#proxy-auto-configuration-files)。

> [!IMPORTANT]
> 不能通过 Docker 守护进程配置文件（`daemon.json`）配置代理，也不建议通过 Docker CLI 配置文件（`config.json`）配置代理。
>
> 如需管理 Docker Desktop 的代理配置，请在应用中设置，或使用[设置管理](/manuals/enterprise/security/hardened-desktop/settings-management/_index.md)。

#### 代理认证

##### 基本认证

如果代理使用 Basic 认证，Docker Desktop 会提示输入用户名和密码，并缓存凭据。密码会安全地存储在操作系统的凭据存储中；若缓存被清除，系统会再次请求认证。

建议使用 `https://` 代理地址以保护密码在网络传输过程中的安全。Docker Desktop 与代理通信时也支持 TLS 1.3。

##### Kerberos 与 NTLM 认证

> [!NOTE]
>
> 适用于 Docker Business 订阅用户，且需 Windows 版 Docker Desktop 4.30 及以上。

由于认证集中化，开发者不再被代理凭据弹窗打断；这也降低了因错误登录尝试导致账号被锁的风险。

如果代理在 407（Proxy Authentication Required）响应中提供多种认证方案，Docker Desktop 默认选择 Basic 方案。

适用于 Docker Desktop 4.30–4.31：

启用 Kerberos 或 NTLM 代理认证除指定代理 IP 与端口外，无需额外配置。

适用于 Docker Desktop 4.32 及以上：

需在命令行安装时传入安装器参数 `--proxy-enable-kerberosntlm`，并确保代理服务器已正确配置 Kerberos 或 NTLM 认证。

### 网络

> [!NOTE]
>
> 在 Windows 容器模式下，**Network** 不可用，因为网络由 Windows 管理。

Docker Desktop 为内部服务（如 DNS 与 HTTP 代理）使用私有 IPv4 网络。若默认子网与环境中的 IP 冲突，你可以在 **Network** 中自定义子网。

在 Windows 与 Mac 上，你还可以设置默认网络模式与 DNS 解析行为。详见[网络](/manuals/desktop/features/networking.md#networking-mode-and-dns-behaviour-for-mac-and-windows)。

在 Mac 上，你可以启用 **Use kernel networking for UDP**，以使用更高效的 UDP 内核网络路径；但可能与部分 VPN 软件不兼容。

### WSL 集成

在 Windows 的 WSL 2 模式下，你可以选择为哪些 WSL 2 发行版启用 Docker 集成。

默认会对当前默认的 WSL 发行版启用集成。要更改默认发行版，运行 `wsl --set-default <distribution name>`（例如：`wsl --set-default ubuntu`）。

你也可以为其他发行版单独开启 WSL 2 集成。

有关在 Docker Desktop 中使用 WSL 2 的更多信息，参见 [WSL 2 后端](/manuals/desktop/features/wsl/_index.md)。

## Docker 引擎

**Docker Engine** 选项卡用于配置 Docker Desktop 运行容器所使用的 Docker 守护进程。

通过 JSON 配置文件来配置守护进程。示例如下：

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false
}
```

该文件位于 `$HOME/.docker/daemon.json`。你可以直接在 Docker Desktop 的仪表板中编辑该 JSON，或使用文本编辑器打开并修改。

完整配置项请参考 [dockerd 命令参考](/reference/cli/dockerd/)。

选择 **Apply** 以保存更改。

## 构建器

如果你开启了 [Docker Desktop 构建视图](/manuals/desktop/use-desktop/builds.md)，即可在 **Builders** 选项卡中查看并管理构建器。

### 查看

要查看构建器，找到目标构建器并点击展开图标。仅可查看处于活动状态的构建器。

查看结果包括：

- BuildKit version
- Status
- Driver type
- Supported capabilities and platforms
- Disk usage
- Endpoint address

### 选择其他构建器

**Selected builder** 区域显示当前选中的构建器。若要切换：

1. 在 **Available builders** 中找到目标构建器
2. 打开其名称旁的下拉菜单
3. 选择 **Use** 切换至该构建器

之后构建命令将默认使用该构建器。

### 创建构建器

使用 Docker CLI 创建构建器。参见[创建新构建器](/build/builders/manage/#create-a-new-builder)。

### 移除构建器

满足以下条件时可移除构建器：

- 该构建器不是当前的[已选构建器](/build/builders/#selected-builder)
- 该构建器未[关联到 Docker 上下文](/build/builders/#default-builder)

  如需移除与上下文关联的构建器，先使用 `docker context rm` 删除相关上下文。

移除步骤：

1. 在 **Available builders** 中找到目标构建器
2. 打开下拉菜单
3. 选择 **Remove**

如果该构建器使用 `docker-container` 或 `kubernetes` 驱动，将连同构建器一并删除其构建缓存。

### 停止与启动构建器

使用 [`docker-container` 驱动](/build/builders/drivers/docker-container/) 的构建器会在容器中运行 BuildKit 守护进程。你可以通过下拉菜单启动或停止该容器。

若容器已停止，运行构建会自动将其拉起。

仅 `docker-container` 驱动的构建器支持启动/停止操作。

## Kubernetes

> [!NOTE]
>
> 在 Windows 容器模式下，**Kubernetes** 选项卡不可用。

Docker Desktop 内置独立的 Kubernetes 服务器，便于测试将 Docker 工作负载部署到 Kubernetes。要启用 Kubernetes 支持，并以 Docker 容器形式安装独立 Kubernetes 实例，请勾选 **Enable Kubernetes**。

从 Docker Desktop 4.38 起，你可以选择集群的初始化方式：
 - **Kubeadm**：创建单节点集群，版本由 Docker Desktop 设定
 - **kind**：创建多节点集群，你可自行设定版本与节点数量 

选择 **Show system containers (advanced)** 可在使用 Docker 命令时查看内部容器。

选择 **Reset Kubernetes cluster** 可删除所有 Stack 与 Kubernetes 资源。

关于在 Docker Desktop 中使用 Kubernetes 集成，参见[在 Kubernetes 上部署](/manuals/desktop/features/kubernetes.md)。

## 软件更新

在 **Software updates** 中可管理 Docker Desktop 的版本更新。当有新版本时，你可以立即下载，或点击 **Release Notes** 查看更新内容。

**Automatically check for updates** 会在 Docker 菜单与仪表板底部提示可用更新（默认开启）。 

如需在后台自动下载更新，开启 **Always download updates**。下载完成后，选择 **Apply and restart** 安装更新。可在 Docker 菜单或仪表板的 **Updates** 区域执行。

**Automatically update components** 会检测 Docker Desktop 组件（如 Docker Compose、Docker Scout、Docker CLI）是否可独立更新而无需完整重启（默认开启）。 

## 扩展

在 **Extensions** 中可以：

- **Enable Docker Extensions**
- **Allow only extensions distributed through the Docker Marketplace**
- **Show Docker Extensions system containers**

关于扩展的更多信息，参见 [Extensions](/manuals/extensions/_index.md)。

## Beta 功能

Beta 功能用于抢先体验后续版本可能提供的能力，仅供测试与反馈之用；它们可能在版本间发生变化或被移除。请勿在生产环境使用，Docker 不为 Beta 功能提供支持。

你也可以在 **Beta features** 中加入 [Developer Preview 计划](https://www.docker.com/community/get-involved/developer-preview/)。

Docker CLI 当前实验特性的列表，参见 [Docker CLI Experimental features](https://github.com/docker/cli/blob/master/experimental/README.md)。

> [!IMPORTANT]
>
> 在 Docker Desktop 4.41 及更早版本，**Features in development** 页面下还有 **Experimental features**。
>
> 与 Beta 功能一致，实验功能也不应在生产环境中使用，Docker 不提供支持。

## 通知

在 **Notifications** 中可以开启或关闭以下事件的通知：

- **Status updates on tasks and processes**
- **Recommendations from Docker**
- **Docker announcements**
- **Docker surveys**

默认情况下，通用通知均为开启。无论设置如何，你始终会收到错误通知与新版本/更新通知。

你也可以[配置与 Docker Scout 相关问题的通知设置](/manuals/scout/explore/dashboard.md#notification-settings)。 

通知会短暂显示在 Docker Desktop 仪表板的右下角，随后进入右上角可访问的 **Notifications** 抽屉。

## 高级

在 Mac 上，你可以在 **Advanced** 中重新配置初始安装设置：

- **Choose how to configure the installation of Docker's CLI tools**：
  - **System**：将 Docker CLI 工具安装到系统目录 `/usr/local/bin`
  - **User**：将 Docker CLI 工具安装到用户目录 `$HOME/.docker/bin`，并将 `$HOME/.docker/bin` 加入 PATH。操作步骤：
      1. 打开你的 shell 配置文件：bash 为 `~/.bashrc`，zsh 为 `~/.zshrc`
      2. 复制并粘贴：
            ```console
            $ export PATH=$PATH:~/.docker/bin
            ```
     3. 保存并关闭文件；重启终端使 PATH 变更生效。

- **Allow the default Docker socket to be used (Requires password)**：创建 `/var/run/docker.sock`，供部分第三方客户端与 Docker Desktop 通信。详见 [macOS 权限要求](/manuals/desktop/setup/install/mac-permission-requirements.md#installing-symlinks)。

- **Allow privileged port mapping (Requires password)**：启动特权助手进程，用于绑定 1–1024 端口。详见 [macOS 权限要求](/manuals/desktop/setup/install/mac-permission-requirements.md#binding-privileged-ports)。

关于各项配置与使用场景，参见[权限要求](/manuals/desktop/setup/install/mac-permission-requirements.md)。
