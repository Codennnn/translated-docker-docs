---
title: Host 网络驱动
description: 在 Docker 主机网络上暴露容器的相关说明
keywords: network, host, standalone, host mode networking
aliases:
- /network/host/
- /network/drivers/host/
---

如果你为容器使用 `host` 网络模式，该容器的网络栈将不再与 Docker 主机隔离（容器与主机共享网络命名空间），并且容器不会分配到独立的 IP 地址。
例如，当你运行一个绑定到端口 80 的容器并启用 `host` 网络时，该容器内的应用将通过主机 IP 的 80 端口对外提供服务。

> [!NOTE]
>
> 由于在 `host` 网络模式下容器没有独立的 IP 地址，[端口映射](overlay.md#publish-ports)不会生效；`-p`、`--publish`、`-P` 与 `--publish-all` 选项将被忽略，并给出警告：
>
> ```console
> WARNING: Published ports are discarded when using host network mode
> ```

Host 模式在以下场景中非常有用：

- 需要优化网络性能
- 容器需要处理大量端口范围的场景

原因是该模式不需要进行网络地址转换（NAT），且无需为每个端口创建“用户态代理”（userland-proxy）。

Host 网络驱动在 Docker Engine（仅限 Linux）以及 Docker Desktop 4.34 及以上版本中受支持。

你也可以在 Swarm 服务中使用 `host` 网络方式：在 `docker service create` 命令中传入 `--network host`。此时，与 Swarm 及服务管理相关的控制流量仍通过 overlay 网络传输，但各个服务容器将使用守护进程主机的网络与端口发送数据。这样会带来额外限制，例如：如果某个服务容器绑定了 80 端口，那么在同一 Swarm 节点上只能运行一个该服务容器。

## Docker Desktop

Docker Desktop 自 4.34 起支持 Host 网络。启用方法：

1. 在 Docker Desktop 中登录你的 Docker 账户。
2. 打开 **Settings**。
3. 在 **Resources** 标签页选择 **Network**。
4. 勾选 **Enable host networking** 选项。
5. 点击 **Apply and restart**。

该功能支持双向访问：你既可以从主机访问容器中的服务器，也可以从启用了 host 网络的任意容器访问主机上的服务。通信协议同时支持 TCP 与 UDP。

### 示例

下面的命令在容器中启动 netcat，使其监听 `8000` 端口：

```console
$ docker run --rm -it --net=host nicolaka/netshoot nc -lkv 0.0.0.0 8000
```

此时主机上的 `8000` 端口可用，你可以在另一个终端通过如下命令连接：

```console
$ nc localhost 8000
```

你在此处输入的内容将会出现在运行该容器的终端上。

要从容器访问主机上的服务，可以使用下述命令启动一个启用了 host 网络的容器：

```console
$ docker run --rm -it --net=host nicolaka/netshoot
```

随后，若要在容器中访问主机上的服务（例如运行在 `80` 端口的 Web 服务器），可以这样：

```console
$ nc localhost 80
```

### 限制

- 容器内的进程无法绑定主机的 IP 地址，因为容器无法直接访问主机网卡接口。
- Docker Desktop 的 host 网络特性工作在第 4 层。这意味着与 Linux 上的 Docker 不同，不支持 TCP/UDP 以下的网络协议。
- 当启用“增强容器隔离”（Enhanced Container Isolation）时，该功能无法使用；因为将容器与主机隔离与允许容器访问主机网络是相互矛盾的。
- 仅支持 Linux 容器；Windows 容器不支持 host 网络。

## 进一步阅读

- 学习[Host 网络教程](/manuals/engine/network/tutorials/host.md)
- 了解[从容器视角的网络](../_index.md)
- 了解[bridge 网络](./bridge.md)
- 了解[overlay 网络](./overlay.md)
- 了解[Macvlan 网络](./macvlan.md)
