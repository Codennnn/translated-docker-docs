---
title: Docker 网络驱动插件
description: "网络驱动插件。"
keywords: "Examples, Usage, plugins, docker, documentation, user guide"
---

本文介绍 Docker Engine 中普遍可用的网络驱动插件。若需了解由 Docker Engine 托管的插件，请参阅《[Docker Engine 插件系统](_index.md)》。

Docker Engine 的网络插件可将 Engine 部署扩展为支持多种网络技术，例如 VXLAN、IPVLAN、MACVLAN，或完全不同的实现。网络驱动插件由 LibNetwork 项目提供支持。每个插件都作为 LibNetwork 的“远程驱动（remote driver）”实现，并与 Engine 共享同一套插件基础设施。实际上，网络驱动插件与其他插件的激活方式与使用的协议相同。

## 网络插件与 Swarm 模式

[传统插件](legacy_plugins.md)在 Swarm 模式下不可用。但使用[v2 插件系统](_index.md)编写的插件可以在 Swarm 模式下使用，前提是它们已安装在每个 Swarm 工作节点上。

## 使用网络驱动插件

如何安装与运行网络驱动插件取决于具体插件。因此，请务必按照插件作者提供的说明进行安装。

插件运行后，使用方式与内置网络驱动相同：在与网络相关的 Docker 命令中将其指定为驱动。例如：

```console
$ docker network create --driver weave mynet
```

部分网络驱动插件列在[插件列表](legacy_plugins.md)中。

`mynet` 网络现在由 `weave` 驱动管理，因此后续引用该网络的命令都会发送给该插件：

```console
$ docker run --network=mynet busybox top
```

## 查找网络插件

网络插件由第三方编写并发布，你可以在[Docker Hub](https://hub.docker.com/search?q=&type=plugin)或第三方站点上找到它们。

## 编写网络插件

网络插件需实现[Docker 插件 API](plugin_api.md)以及网络插件协议。

## 网络插件协议

除插件激活调用外，网络驱动协议作为 libnetwork 的一部分进行说明：
[https://github.com/moby/moby/blob/master/libnetwork/docs/remote.md](https://github.com/moby/moby/blob/master/libnetwork/docs/remote.md)。

## 相关信息

若要与 Docker 维护者与其它感兴趣的用户交流，请加入 IRC 频道 `#docker-network`。

- [Docker 网络功能概览](https://docs.docker.com/engine/userguide/networking/)
- [LibNetwork](https://github.com/docker/libnetwork) 项目
