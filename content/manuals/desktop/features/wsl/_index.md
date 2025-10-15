---
description: 启用 Docker 的 WSL 2 后端，并在本指南中掌握最佳实践、GPU 支持等内容，快速投入使用。
keywords: wsl, wsl2, installing wsl2, wsl installation, docker wsl2, wsl docker, wsl2
  tech preview, wsl install docker, install docker wsl, how to install docker in wsl
title: 在 Windows 上使用 Docker Desktop 的 WSL 2 后端
linkTitle: WSL
weight: 120
aliases:
- /docker-for-windows/wsl/
- /docker-for-windows/wsl-tech-preview/
- /desktop/windows/wsl/
- /desktop/wsl/
---

Windows Subsystem for Linux（WSL）2 是由 Microsoft 构建的完整 Linux 内核，使你无需管理虚拟机即可在 Windows 上运行各类 Linux 发行版。将 Docker Desktop 运行在 WSL 2 之上后，你可以直接使用 Linux 开发环境，避免同时维护 Linux 与 Windows 的构建脚本。此外，WSL 2 在文件系统共享与启动时间方面也有显著改进。

Docker Desktop 利用 WSL 2 的动态内存分配功能来优化资源占用。这意味着 Docker Desktop 只会按需使用 CPU 与内存资源；同时，诸如构建容器等 CPU/内存密集型任务也能运行得更快。

此外，在 WSL 2 下，冷启动后启动 Docker 守护进程的耗时也显著缩短。

## 先决条件

在启用 Docker Desktop 的 WSL 2 功能之前，请确认：

- 至少使用 WSL 2.1.5，推荐更新至最新版本，以[避免 Docker Desktop 异常](best-practices.md)。
- 满足 Docker Desktop for Windows 的[系统要求](/manuals/desktop/setup/install/windows-install.md#system-requirements)。
- 在 Windows 上启用并安装 WSL 2 功能。详细步骤参见[微软文档](https://docs.microsoft.com/en-us/windows/wsl/install-win10)。

> [!TIP]
>
> 为获得更佳 WSL 体验，建议开启自 WSL 1.3.10 起提供的（实验性）
> [autoMemoryReclaim](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#experimental-settings)
> 设置。
>
> 该功能增强了 Windows 主机回收 WSL 虚拟机未使用内存的能力，使其他主机应用获得更多可用内存。对 Docker Desktop 尤其有益：在容器镜像构建期间，WSL VM 不会在 Linux 内核页缓存中长期占用大量内存（数 GB 级），在 VM 侧不再需要时可以及时归还给主机。

## 启用 Docker Desktop 的 WSL 2 后端

> [!IMPORTANT]
>
> 为避免与 WSL 2 冲突，在安装 Docker Desktop 之前，必须卸载曾通过 Linux 发行版直接安装的 Docker Engine 与 CLI。

1. 下载并安装最新版本的 [Docker Desktop for Windows](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-windows)。
2. 按常规安装向导完成安装。根据 Windows 版本的不同，安装过程中可能提示你启用 WSL 2。阅读提示并开启 WSL 2 以继续。
3. 从 **Windows Start** 菜单启动 Docker Desktop。
4. 进入 **Settings**。
5. 在 **General** 选项卡，勾选 **Use WSL 2 based engine**。

    若系统支持 WSL 2，该选项默认已启用。
6. 选择 **Apply**。

现在即可在 Windows 上基于新的 WSL 2 引擎运行 `docker` 命令。

> [!TIP]
>
> 默认情况下，Docker Desktop 会将 WSL 2 引擎的数据存储在 `C:\Users\[USERNAME]\AppData\Local\Docker\wsl`。
> 如需更改存储位置（例如迁移到其他磁盘），可在 Docker Dashboard 的 `Settings -> Resources -> Advanced` 页面进行设置。
> 更多 Windows 相关设置见[更改设置](/manuals/desktop/settings-and-maintenance/settings.md)

## 在 WSL 2 发行版中启用 Docker 支持

WSL 2 为 Windows 引入了“Linux 发行版”支持；各发行版的行为类似 VM，但共享同一个 Linux 内核。

Docker Desktop 并不要求必须安装某个特定的 Linux 发行版；即使不安装，`docker` CLI 与 UI 也可直接在 Windows 上正常使用。但为获得更佳的开发体验，建议至少安装一个发行版并启用 Docker 集成：

1. 确保发行版运行在 WSL 2 模式。WSL 支持 v1 与 v2 两种模式。

    要检查发行版的 WSL 模式，运行：

     ```console
     $ wsl.exe -l -v
     ```

    将发行版升级为 v2，运行：

    ```console
    $ wsl.exe --set-version (distribution name) 2
    ```

    将 v2 设为后续安装的默认版本，运行：

    ```console
    $ wsl.exe --set-default-version 2
    ```

2. Docker Desktop 启动后，进入 **Settings** > **Resources** > **WSL Integration**。

    Docker-WSL 集成会在默认的 WSL 发行版上启用（默认通常为 [Ubuntu](https://learn.microsoft.com/en-us/windows/wsl/install)）。如需更改默认发行版，运行：
     ```console
    $ wsl.exe --set-default <distribution name>
    ```
   若在 **Resources** 下未看到 **WSL integrations**，可能当前处于 Windows 容器模式。请在任务栏的 Docker 菜单中选择 **Switch to Linux containers**。

3. 选择 **Apply**。

> [!NOTE]
>
> 在 Docker Desktop 4.30 及更早版本中，安装会创建两个内部用途的 Linux 发行版：`docker-desktop` 与 `docker-desktop-data`。`docker-desktop` 用于运行 Docker 引擎 `dockerd`，而 `docker-desktop-data` 用于存储容器与镜像。它们均不适用于日常开发。
>
> 在全新安装的 4.30 及更高版本中，将不再创建 `docker-desktop-data`；Docker Desktop 会改为创建并管理专用的虚拟硬盘用于存储。`docker-desktop` 仍会被创建并用于运行 Docker 引擎。
>
> 注意：若从旧版本升级且未执行全新安装或恢复出厂设置，4.30 及更高版本仍会沿用已有的 `docker-desktop-data` 发行版。

## Docker Desktop 中的 WSL 2 安全性

Docker Desktop 的 WSL 2 集成遵循 WSL 既有的安全模型，不会引入超出标准 WSL 行为之外的额外安全风险。

Docker Desktop 运行在其专用的 WSL 发行版 `docker-desktop` 中，具备与其他 WSL 发行版相同的隔离属性。只有当在设置中启用 Docker Desktop 的 **WSL 集成** 时，Docker Desktop 才会与其他已安装的 WSL 发行版交互，该集成功能使集成的发行版能够便捷地访问 Docker CLI。 

WSL 的设计目标之一是促进 Windows 与 Linux 环境的互操作。其文件系统可通过 Windows 主机的 `\\wsl$` 访问，这意味着 Windows 进程可以读取并修改 WSL 内的文件。这并非 Docker Desktop 特有行为，而是 WSL 的核心特性。

如组织对 WSL 相关的安全风险有所顾虑并需要更严格的隔离与管控，可选择在 Hyper‑V 模式而非 WSL 2 下运行 Docker Desktop。或者，为容器工作负载启用[增强型容器隔离](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/_index.md)。

## 相关资源

- [查看最佳实践](best-practices.md)
- [了解如何在 WSL 2 上进行 Docker 开发](use-wsl.md)
- [了解 WSL 2 的 GPU 支持](/manuals/desktop/features/gpu.md)
- [WSL 自定义内核](custom-kernels.md)
