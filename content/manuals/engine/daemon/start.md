---
title: 启动守护进程
weight: 10
description: 手动启动 Docker 守护进程
keywords: docker, daemon, configuration, troubleshooting
aliases:
  - /config/daemon/start/
---

本页介绍如何启动守护进程，可以手动启动，也可以通过操作系统的服务管理器启动。

## 使用操作系统的服务管理器启动

在典型安装中，Docker 守护进程由系统服务管理器启动，而不是由用户手动启动。这样可以在机器重启时自动启动 Docker。

启动命令因操作系统而异。请在 [安装 Docker](/manuals/engine/install/_index.md) 文档中查找对应页面。

### 使用 systemd 启动

在 Ubuntu、Debian 等操作系统上，Docker 守护进程服务通常会自动启动。你也可以手动执行：

```console
$ sudo systemctl start docker
```

如果希望开机时自动启动，请参见
[使用 systemd 配置开机自启](/manuals/engine/install/linux-postinstall.md#configure-docker-to-start-on-boot-with-systemd)。

## 手动启动守护进程

如果你不想使用系统服务管理器来管理 Docker 守护进程，或仅用于测试，可以直接使用 `dockerd` 命令手动运行。是否需要 `sudo` 取决于你的操作系统配置。

以这种方式启动时，Docker 以前台方式运行，并将日志直接输出到终端：

```console
$ dockerd

INFO[0000] +job init_networkdriver()
INFO[0000] +job serveapi(unix:///var/run/docker.sock)
INFO[0000] Listening for HTTP on unix (/var/run/docker.sock)
```

若以手动方式启动，需要停止 Docker 时，请在终端中按 `Ctrl+C`。
