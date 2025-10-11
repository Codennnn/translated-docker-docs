---
title: Macvlan 网络驱动
description: 使用 Macvlan 让你的容器在网络中表现得像物理主机
keywords: network, macvlan, standalone
aliases:
- /config/containers/macvlan/
- /engine/userguide/networking/get-started-macvlan/
- /network/macvlan/
- /network/drivers/macvlan/
---

某些应用（尤其是遗留应用或需要监控网络流量的应用）期望直接连接到物理网络。在这种情况下，你可以使用 `macvlan` 网络驱动，为每个容器的虚拟网卡分配一个 MAC 地址，使其看起来像是直接连接到物理网络的物理网卡。
在此模式下，你需要在 Docker 主机上指定用于 Macvlan 的物理接口，以及该网络的子网与网关。你甚至可以通过不同的物理网络接口来隔离多个 Macvlan 网络。

请注意以下事项：

- 由于 IP 地址耗尽，或因“VLAN 扩散”（网络中唯一 MAC 地址数量过多）等原因，你可能会无意中降低网络性能。

- 你的网络设备需要支持“混杂模式”（promiscuous mode），即一个物理接口可被分配多个 MAC 地址。

- 如果你的应用可以使用 bridge（单主机）或 overlay（跨多主机通信），从长期看这些方案可能更合适。

## 选项

下表列出了使用 `macvlan` 驱动创建网络时，可通过 `--opt` 传入的驱动特定选项：

| 选项           | 默认值   | 说明                                                                                 |
| -------------- | -------- | ------------------------------------------------------------------------------------ |
| `macvlan_mode` | `bridge` | 设置 Macvlan 的模式，可选：`bridge`、`vepa`、`passthru`、`private`                 |
| `parent`       |          | 指定要使用的父接口。                                                                 |

## 创建 Macvlan 网络

创建 Macvlan 网络时，可以使用 bridge 模式或 802.1Q trunk bridge 模式。

- 在 bridge 模式下，Macvlan 流量通过主机上的物理设备转发。

- 在 802.1Q trunk bridge 模式下，流量通过 Docker 动态创建的 802.1Q 子接口转发。这样可以更细粒度地控制路由与过滤。

### Bridge 模式

要创建与指定物理网卡桥接的 `macvlan` 网络，请在 `docker network create` 中使用 `--driver macvlan`，并指定 `parent`（即在 Docker 主机上实际承载流量的接口）。

```console
$ docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 pub_net
```

如果需要在 `macvlan` 网络中排除某些 IP 地址（例如该地址已被占用），可以使用 `--aux-addresses`：

```console
$ docker network create -d macvlan \
  --subnet=192.168.32.0/24 \
  --ip-range=192.168.32.128/25 \
  --gateway=192.168.32.254 \
  --aux-address="my-router=192.168.32.129" \
  -o parent=eth0 macnet32
```

### 802.1Q trunk bridge 模式

如果在 `parent` 中指定带点的接口名（如 `eth0.50`），Docker 会将其视为 `eth0` 的子接口并自动创建。

```console
$ docker network create -d macvlan \
    --subnet=192.168.50.0/24 \
    --gateway=192.168.50.1 \
    -o parent=eth0.50 macvlan50
```

### 使用 IPvlan 替代 Macvlan

在上述示例中，你仍在使用三层 bridge。也可以改用 `ipvlan` 实现二层 bridge：指定 `-o ipvlan_mode=l2`。

```console
$ docker network create -d ipvlan \
    --subnet=192.168.210.0/24 \
    --subnet=192.168.212.0/24 \
    --gateway=192.168.210.254 \
    --gateway=192.168.212.254 \
     -o ipvlan_mode=l2 -o parent=eth0 ipvlan210
```

## 使用 IPv6

如果你已[将 Docker 守护进程配置为允许 IPv6](/manuals/engine/daemon/ipv6.md)，即可使用双栈 IPv4/IPv6 的 `macvlan` 网络。

```console
$ docker network create -d macvlan \
    --subnet=192.168.216.0/24 --subnet=192.168.218.0/24 \
    --gateway=192.168.216.1 --gateway=192.168.218.1 \
    --subnet=2001:db8:abc8::/64 --gateway=2001:db8:abc8::10 \
     -o parent=eth0.218 \
     -o macvlan_mode=bridge macvlan216
```

## 进一步阅读

在[Macvlan 网络教程](/manuals/engine/network/tutorials/macvlan.md)中学习如何使用 Macvlan 驱动。
