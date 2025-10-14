---
title: 使用 Docker Desktop CLI
linkTitle: Docker Desktop 命令行
weight: 100
description: 如何使用 Docker Desktop CLI
keywords: cli, docker desktop, macos, windows, linux
---

{{< summary-bar feature_name="Docker Desktop CLI" >}}

Docker Desktop CLI 允许你直接在命令行中执行关键操作，例如启动、停止、重启以及更新 Docker Desktop。

Docker Desktop CLI 提供：

- 面向本地开发的简化自动化：在脚本与测试中更高效地执行 Docker Desktop 操作。
- 更好的开发者体验：通过命令行重启、退出或重置 Docker Desktop，减少对 Docker Desktop 仪表板的依赖，提高灵活性与效率。

## Usage

```console
docker desktop COMMAND [OPTIONS]
```

## Commands

| Command              | Description                              |
|:---------------------|:-----------------------------------------|
| `start`              | 启动 Docker Desktop                      |
| `stop`               | 停止 Docker Desktop                      |
| `restart`            | 重启 Docker Desktop                      |
| `status`             | 显示 Docker Desktop 处于运行或停止状态   |
| `engine ls`          | 列出可用的引擎（仅限 Windows）           |
| `engine use`         | 在 Linux 与 Windows 容器间切换（仅限 Windows） |
| `update`             | 管理 Docker Desktop 更新。仅在 Mac 的 4.38 版中可用，或在 4.39 及更高版本对所有操作系统可用。 |
| `logs`               | 打印日志条目                             |
| `disable`            | 禁用某个功能                             |
| `enable`             | 启用某个功能                             | 
| `version`            | 显示 Docker Desktop CLI 插件版本信息     |
| `kubernetes`         | 列出 Docker Desktop 使用的 Kubernetes 镜像，或重启集群。自 Docker Desktop 4.44 及更高版本可用。 |

关于每个命令的详细说明，请参阅 [Docker Desktop CLI 参考](/reference/cli/docker/desktop/_index.md)。
