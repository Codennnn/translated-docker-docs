---
description: 了解如何在守护进程不可用时保持容器继续运行
keywords: docker, upgrade, daemon, dockerd, live-restore, daemonless container
title: 实时恢复（Live restore）
weight: 40
aliases:
  - /config/containers/live-restore/
  - /engine/admin/live-restore/
  - /engine/containers/live-restore/
---

默认情况下，Docker 守护进程退出时会停止正在运行的容器。
你可以配置守护进程，使其在不可用时容器仍保持运行。这项功能称为 _live restore_。
启用后，可在守护进程崩溃、计划停机或升级期间，显著减少容器停机时间。

> [!NOTE]
>
> Windows 容器不支持 Live restore。
> 但在 Docker Desktop for Windows 中运行的 Linux 容器支持该功能。

## 启用 Live restore

当守护进程不可用时，要让容器保持运行，可以通过以下两种方式启用 Live restore。
**二选一，不要同时使用。**

- 在守护进程配置文件中添加配置。Linux 上默认路径为 `/etc/docker/daemon.json`。
  在 Docker Desktop for Mac / Windows 中，点击任务栏 Docker 图标，依次进入
  **Settings** -> **Docker Engine**。

  - 使用以下 JSON 启用 `live-restore`：

    ```json
    {
      "live-restore": true
    }
    ```

  - 重启 Docker 守护进程。在 Linux 上，你也可以通过重新加载守护进程来避免重启
    （从而避免容器停机）。若使用 `systemd`，执行 `systemctl reload docker`；
    否则，向 `dockerd` 进程发送 `SIGHUP` 信号。

- 也可以手动以 `--live-restore` 标志启动 `dockerd` 进程。
  但这种方式不建议使用，因为它不会像 `systemd` 或其他进程管理器那样设置运行环境，
  可能导致意外行为。

## 升级期间的 Live restore

Live restore 支持守护进程更新期间保持容器不间断运行，
但仅适用于安装补丁版本（`YY.MM.x`），不支持跨主版本（`YY.MM`）升级。

如果升级时跳过了多个版本，守护进程可能无法恢复与容器的连接。
一旦无法恢复连接，守护进程将无法管理运行中的容器，你需要手动停止它们。

## 重启后的 Live restore

仅当守护进程的关键参数未改变（例如网桥 IP、存储驱动等）时，Live restore 才能成功恢复容器。
如果这些守护进程级别的配置发生变化，Live restore 可能失效，你可能需要手动停止容器。

## 对运行中容器的影响

如果守护进程长时间不可用，运行中的容器可能会填满守护进程平时读取的 FIFO 日志。
一旦缓冲区被占满，容器将无法继续写入日志。默认缓冲区大小为 64K。
若缓冲区已满，你必须重启 Docker 守护进程以清空。

在 Linux 上，可以通过修改 `/proc/sys/fs/pipe-max-size` 来调整内核缓冲区大小。
在 Docker Desktop for Mac / Windows 上无法调整该缓冲区大小。

## 与 Swarm 模式的关系

Live restore 仅适用于独立容器，不适用于 Swarm 服务。Swarm 服务由管理节点负责调度与管理。
当管理节点不可用时，服务仍会在工作节点上继续运行，但直到管理节点恢复至可用法定数量前，
它们无法被管理。
