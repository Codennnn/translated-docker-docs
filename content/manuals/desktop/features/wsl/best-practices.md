---
title: 最佳实践
description: 将 Docker Desktop 与 WSL 2 搭配使用时的最佳实践
keywords: wsl, docker desktop, best practices
tags: [Best practices]
aliases:
- /desktop/wsl/best-practices/
---

- 始终使用最新版本的 WSL。至少需要 WSL 2.1.5；否则 Docker Desktop 可能无法正常工作。官方的测试、开发与文档均基于最新内核版本。较旧的 WSL 版本可能导致：
    - Docker Desktop 间歇性卡顿或在升级时挂起
    - 通过 SCCM 部署失败
    - `vmmem.exe` 占用全部内存
    - 网络过滤策略被全局应用，而非按对象生效
    - 容器中的 GPU 使用失败

- 为获得更佳的绑定挂载（bind‑mount）文件系统性能，建议将源代码和需要挂载到 Linux 容器中的数据存放在 Linux 文件系统中。例如，应在 Linux 文件系统中执行形如 `docker run -v <host-path>:<container-path>` 的挂载，而非从 Windows 文件系统挂载。也可参考 Microsoft 的[官方建议](https://learn.microsoft.com/en-us/windows/wsl/compare-versions)。
    - 仅当原始文件位于 Linux 文件系统中时，Linux 容器才能接收到文件更改事件（inotify 事件）。例如，一些 Web 开发流程依赖 inotify 事件在文件变更时自动重载。
    - 与从 Windows 主机远程挂载相比，从 Linux 文件系统进行绑定挂载的性能更高。因此应避免 `docker run -v /mnt/c/users:/users` 这类路径（其中 `/mnt/c` 来自 Windows 挂载）。
    - 更佳做法是在 Linux shell 中使用类似命令：`docker run -v ~/my-project:/sources <my-image>`，其中 `~` 会由 Linux shell 展开为 `$HOME`。

- 若担心 `docker-desktop-data` 发行版占用空间过大，可查看 Windows 内置的 [WSL 管理工具](https://learn.microsoft.com/en-us/windows/wsl/disk-space)。 
    - 自 4.30 起，全新安装的 Docker Desktop 不再依赖 `docker-desktop-data` 发行版；Docker Desktop 会创建并管理自身的虚拟硬盘（VHDX）用于存储。（但如果早期版本已创建过 `docker-desktop-data`，且未全新安装或恢复出厂设置，仍将沿用该发行版。）
    - 自 4.34 起，Docker Desktop 会自动管理该 VHDX 的大小，并将未使用空间回收并归还给操作系统。

- 若担心 CPU 或内存占用，可为 [WSL 2 实用 VM（utility VM）](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#global-configuration-options-with-wslconfig) 配置内存、CPU 与交换分区（swap）的限制。
