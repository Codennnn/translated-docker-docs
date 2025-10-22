---
title: 从扩展后端使用 Docker 套接字
linkTitle: 使用 Docker 套接字
description: Docker 扩展元数据
keywords: Docker, 扩展, sdk, 元数据
aliases: 
 - /desktop/extensions-sdk/guides/use-docker-socket-from-backend/
---

扩展可以通过 SDK 直接从前端调用 Docker 命令。

在某些情况下，从后端与 Docker Engine 交互也很有用。

扩展后端容器可以挂载 Docker 套接字，并使用它从扩展后端逻辑与 Docker Engine 交互。了解更多关于 [Docker Engine 套接字](/reference/cli/dockerd/#examples) 的信息。

但是，当从位于 Desktop 虚拟机中的扩展容器挂载 Docker 套接字时，您需要挂载虚拟机内部的 Docker 套接字，而不是从主机文件系统挂载 `/var/run/docker.sock`（使用主机的 Docker 套接字可能会导致容器中的权限问题）。

为此，您可以使用 `/var/run/docker.sock.raw`。Docker Desktop 挂载的是位于 Desktop 虚拟机中的套接字，而不是来自主机的套接字。

```yaml
services:
  myExtension:
    image: ${DESKTOP_PLUGIN_IMAGE}
    volumes:
      - /var/run/docker.sock.raw:/var/run/docker.sock
```
