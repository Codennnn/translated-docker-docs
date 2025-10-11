---
title: None 网络驱动
description: 使用 none 驱动完全隔离容器的网络栈
keywords: network, none, standalone
aliases:
- /network/none/
- /network/drivers/none/
---

如果你想完全隔离某个容器的网络栈，可以在启动容器时使用 `--network none` 标志。此时容器内部只会创建回环设备（loopback）。

下面的示例展示了在使用 `none` 网络驱动的 `alpine` 容器中执行 `ip link show` 的输出：

```console
$ docker run --rm --network none alpine:latest ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

使用 `none` 驱动的容器不会配置 IPv6 回环地址。

```console
$ docker run --rm --network none --name no-net-alpine alpine:latest ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

## 进一步阅读

- 学习[Host 网络教程](/manuals/engine/network/tutorials/host.md)
- 了解[从容器视角的网络](../_index.md)
- 了解[bridge 网络](bridge.md)
- 了解[overlay 网络](overlay.md)
- 了解[Macvlan 网络](macvlan.md)
