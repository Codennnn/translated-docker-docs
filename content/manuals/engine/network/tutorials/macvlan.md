title: 使用 macvlan 网络进行联网
description: 使用 macvlan 桥接网络与 802.1Q trunk 桥接网络的联网教程
keywords: networking, macvlan, 802.1Q, standalone
aliases:
  - /network/network-tutorial-macvlan/
---

本系列教程介绍如何将独立容器连接到 `macvlan` 网络进行联网。在此类网络中，Docker 主机会在自身 IP 下接受多个 MAC 地址的请求，并将这些请求路由到相应的容器。更多网络主题请参见[概览](/manuals/engine/network/_index.md)。

## 目标

本教程目标是：先创建一个桥接模式的 `macvlan` 网络并将容器连接其上；随后创建一个 802.1Q trunk 的 `macvlan` 网络并将容器连接其上。

## 前提条件

- 大多数云服务商会屏蔽 `macvlan` 网络。你可能需要对网络设备具备物理访问能力。

- `macvlan` 网络驱动仅适用于 Linux 主机；Windows 上的 Docker Desktop 与 Docker Engine 不支持。

- 至少需要 Linux 内核 3.9，推荐 4.0 及以上版本。

- 示例默认你的以太网接口为 `eth0`；若设备名称不同，请相应替换。

- rootless 模式不支持 `macvlan` 驱动。

## Bridge 示例

在简单的 bridge 示例中，流量通过 `eth0`，Docker 根据容器的 MAC 地址将流量路由到容器。在你的网络设备看来，容器就像物理接入了网络。

1.  创建名为 `my-macvlan-net` 的 `macvlan` 网络。根据你的环境修改 `subnet`、`gateway` 与 `parent` 的取值。

    ```console
    $ docker network create -d macvlan \
      --subnet=172.16.86.0/24 \
      --gateway=172.16.86.1 \
      -o parent=eth0 \
      my-macvlan-net
    ```

    你可以使用 `docker network ls` 与 `docker network inspect my-macvlan-net` 验证该网络已创建且类型为 `macvlan`。

2.  启动一个 `alpine` 容器并将其连接到 `my-macvlan-net` 网络。`-dit` 标志表示在后台启动容器但仍可附着；`--rm` 表示容器停止后自动删除。

    ```console
    $ docker run --rm -dit \
      --network my-macvlan-net \
      --name my-macvlan-alpine \
      alpine:latest \
      ash
    ```

3.  检查 `my-macvlan-alpine` 容器，留意 `Networks` 下的 `MacAddress` 字段：

    ```console
    $ docker container inspect my-macvlan-alpine

    ...truncated...
    "Networks": {
      "my-macvlan-net": {
          "IPAMConfig": null,
          "Links": null,
          "Aliases": [
              "bec64291cd4c"
          ],
          "NetworkID": "5e3ec79625d388dbcc03dcf4a6dc4548644eb99d58864cf8eee2252dcfc0cc9f",
          "EndpointID": "8caf93c862b22f379b60515975acf96f7b54b7cf0ba0fb4a33cf18ae9e5c1d89",
          "Gateway": "172.16.86.1",
          "IPAddress": "172.16.86.2",
          "IPPrefixLen": 24,
          "IPv6Gateway": "",
          "GlobalIPv6Address": "",
          "GlobalIPv6PrefixLen": 0,
          "MacAddress": "02:42:ac:10:56:02",
          "DriverOpts": null
      }
    }
    ...truncated
    ```

4.  通过执行几条 `docker exec` 命令，查看容器自身的网络接口与路由：

    ```console
    $ docker exec my-macvlan-alpine ip addr show eth0

    9: eth0@tunl0: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:10:56:02 brd ff:ff:ff:ff:ff:ff
    inet 172.16.86.2/24 brd 172.16.86.255 scope global eth0
       valid_lft forever preferred_lft forever
    ```

    ```console
    $ docker exec my-macvlan-alpine ip route

    default via 172.16.86.1 dev eth0
    172.16.86.0/24 dev eth0 scope link  src 172.16.86.2
    ```

5.  停止容器（由于 `--rm` 标志容器会被自动移除），并删除该网络。

    ```console
    $ docker container stop my-macvlan-alpine

    $ docker network rm my-macvlan-net
    ```

## 802.1Q trunk bridge 示例

在 802.1Q trunk bridge 示例中，流量通过 `eth0` 的一个子接口（如 `eth0.10`），Docker 依然基于容器的 MAC 地址进行路由。在你的网络设备看来，容器同样像物理接入在网络中。

1.  创建名为 `my-8021q-macvlan-net` 的 `macvlan` 网络。根据你的环境修改 `subnet`、`gateway` 与 `parent` 的取值。

    ```console
    $ docker network create -d macvlan \
      --subnet=172.16.86.0/24 \
      --gateway=172.16.86.1 \
      -o parent=eth0.10 \
      my-8021q-macvlan-net
    ```

    你可以使用 `docker network ls` 与 `docker network inspect my-8021q-macvlan-net` 验证该网络存在、类型为 `macvlan` 且父接口为 `eth0.10`。在 Docker 主机上使用 `ip addr show` 可进一步确认 `eth0.10` 已存在且拥有独立 IP 地址。

2.  启动一个 `alpine` 容器并将其连接到 `my-8021q-macvlan-net` 网络。`-dit` 表示在后台启动但可以附着，`--rm` 表示容器停止后即被移除。

    ```console
    $ docker run --rm -itd \
      --network my-8021q-macvlan-net \
      --name my-second-macvlan-alpine \
      alpine:latest \
      ash
    ```

3.  检查 `my-second-macvlan-alpine` 容器，留意 `Networks` 下的 `MacAddress` 字段：

    ```console
    $ docker container inspect my-second-macvlan-alpine

    ...truncated...
    "Networks": {
      "my-8021q-macvlan-net": {
          "IPAMConfig": null,
          "Links": null,
          "Aliases": [
              "12f5c3c9ba5c"
          ],
          "NetworkID": "c6203997842e654dd5086abb1133b7e6df627784fec063afcbee5893b2bb64db",
          "EndpointID": "aa08d9aa2353c68e8d2ae0bf0e11ed426ea31ed0dd71c868d22ed0dcf9fc8ae6",
          "Gateway": "172.16.86.1",
          "IPAddress": "172.16.86.2",
          "IPPrefixLen": 24,
          "IPv6Gateway": "",
          "GlobalIPv6Address": "",
          "GlobalIPv6PrefixLen": 0,
          "MacAddress": "02:42:ac:10:56:02",
          "DriverOpts": null
      }
    }
    ...truncated
    ```

4.  通过执行几条 `docker exec` 命令，查看容器的网卡与路由：

    ```console
    $ docker exec my-second-macvlan-alpine ip addr show eth0

    11: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:10:56:02 brd ff:ff:ff:ff:ff:ff
    inet 172.16.86.2/24 brd 172.16.86.255 scope global eth0
       valid_lft forever preferred_lft forever
    ```

    ```console
    $ docker exec my-second-macvlan-alpine ip route

    default via 172.16.86.1 dev eth0
    172.16.86.0/24 dev eth0 scope link  src 172.16.86.2
    ```

5.  停止容器（由于 `--rm` 标志容器会被自动移除），并删除该网络。

    ```console
    $ docker container stop my-second-macvlan-alpine

    $ docker network rm my-8021q-macvlan-net
    ```

## 其他网络教程

- [独立网络教程](/manuals/engine/network/tutorials/standalone.md)
- [Overlay 网络教程](/manuals/engine/network/tutorials/overlay.md)
- [Host 网络教程](/manuals/engine/network/tutorials/host.md)
