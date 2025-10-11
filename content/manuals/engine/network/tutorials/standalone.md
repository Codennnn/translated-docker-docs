---
title: 使用独立容器进行联网
description: 独立容器的联网教程
keywords: networking, bridge, routing, ports, overlay
aliases:
  - /network/network-tutorial-standalone/
---

本系列教程介绍独立 Docker 容器的网络配置。
若需了解 Swarm 服务的联网方法，请参见[Swarm 服务联网](/manuals/engine/network/tutorials/overlay.md)。如需了解 Docker 网络的通用概念，请参见[概览](/manuals/engine/network/_index.md)。

本主题包含两个教程。除最后一个需要额外一台 Docker 主机外，其余均可在 Linux、Windows 或 macOS 上运行。

- [使用默认 bridge 网络](#use-the-default-bridge-network)：演示使用 Docker 自动提供的默认 `bridge` 网络。该网络不适用于生产环境。

- [使用用户自定义 bridge 网络](#use-user-defined-bridge-networks)：演示在同一 Docker 主机上创建并使用自定义 bridge 网络连接容器。建议生产环境中的独立容器使用。

尽管[overlay 网络](/manuals/engine/network/drivers/overlay.md)通常用于 Swarm 服务，也可以用于独立容器，详见[overlay 网络教程](/manuals/engine/network/tutorials/overlay.md#use-an-overlay-network-for-standalone-containers)。

## 使用默认 bridge 网络

本示例会在同一主机上启动两个 `alpine` 容器，并通过测试理解它们如何相互通信。需要事先安装并运行 Docker。

1.  打开终端。先列出当前网络。如果从未在该 Docker 守护进程上添加过网络或初始化过 swarm，你应至少看到如下网络（ID 会不同）：

    ```console
    $ docker network ls

    NETWORK ID          NAME                DRIVER              SCOPE
    17e324f45964        bridge              bridge              local
    6ed54d316334        host                host                local
    7092879f2cc8        none                null                local
    ```

    默认的 `bridge` 网络与 `host`、`none` 一并列出。后两者并非完整网络，分别用于让容器直接使用主机网络栈，或让容器无网络设备。本教程将把两个容器连接到 `bridge` 网络。

2.  启动两个运行 `ash`（Alpine 默认 shell，而非 `bash`）的 `alpine` 容器。`-dit` 表示后台启动、可交互并分配 TTY。由于后台启动，你不会立刻附着到容器，而是看到容器 ID。未指定 `--network` 时，容器会连接到默认 `bridge` 网络。

    ```console
    $ docker run -dit --name alpine1 alpine ash

    $ docker run -dit --name alpine2 alpine ash
    ```

    确认两个容器均已启动：

    ```console
    $ docker container ls

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    602dbf1edc81        alpine              "ash"               4 seconds ago       Up 3 seconds                            alpine2
    da33b7aa74b0        alpine              "ash"               17 seconds ago      Up 16 seconds                           alpine1
    ```

3.  检查 `bridge` 网络，查看连接的容器：

    ```console
    $ docker network inspect bridge

    [
        {
            "Name": "bridge",
            "Id": "17e324f459648a9baaea32b248d3884da102dde19396c25b30ec800068ce6b10",
            "Created": "2017-06-22T20:27:43.826654485Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": [
                    {
                        "Subnet": "172.17.0.0/16",
                        "Gateway": "172.17.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {
                "602dbf1edc81813304b6cf0a647e65333dc6fe6ee6ed572dc0f686a3307c6a2c": {
                    "Name": "alpine2",
                    "EndpointID": "03b6aafb7ca4d7e531e292901b43719c0e34cc7eef565b38a6bf84acf50f38cd",
                    "MacAddress": "02:42:ac:11:00:03",
                    "IPv4Address": "172.17.0.3/16",
                    "IPv6Address": ""
                },
                "da33b7aa74b0bf3bda3ebd502d404320ca112a268aafe05b4851d1e3312ed168": {
                    "Name": "alpine1",
                    "EndpointID": "46c044a645d6afc42ddd7857d19e9dcfb89ad790afb5c239a35ac0af5e8a5bc5",
                    "MacAddress": "02:42:ac:11:00:02",
                    "IPv4Address": "172.17.0.2/16",
                    "IPv6Address": ""
                }
            },
            "Options": {
                "com.docker.network.bridge.default_bridge": "true",
                "com.docker.network.bridge.enable_icc": "true",
                "com.docker.network.bridge.enable_ip_masquerade": "true",
                "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
                "com.docker.network.bridge.name": "docker0",
                "com.docker.network.driver.mtu": "1500"
            },
            "Labels": {}
        }
    ]
    ```

    顶部包含 `bridge` 网络信息，其中包括主机与 `bridge` 网络之间的网关 IP（`172.17.0.1`）。在 `Containers` 下，可见每个已连接容器及其 IP（如 `alpine1` 为 `172.17.0.2`，`alpine2` 为 `172.17.0.3`）。

4.  容器在后台运行。使用 `docker attach` 连接到 `alpine1`：

    ```console
    $ docker attach alpine1

    / #
    ```

    提示符变为 `#`，表示你在容器内是 `root`。使用 `ip addr show` 查看容器内的网络接口：

    ```console
    # ip addr show

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    27: eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
        link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.2/16 scope global eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::42:acff:fe11:2/64 scope link
           valid_lft forever preferred_lft forever
    ```

    第一块是回环接口，可忽略。第二块网卡的 IP `172.17.0.2` 与前一步中 `alpine1` 的地址一致。

5.  在 `alpine1` 内 ping `google.com` 验证外网连通性；`-c 2` 表示发送两次：

    ```console
    # ping -c 2 google.com

    PING google.com (172.217.3.174): 56 data bytes
    64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.841 ms
    64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.897 ms

    --- google.com ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 9.841/9.869/9.897 ms
    ```

6.  试着 ping 第二个容器。先按 IP `172.17.0.3`：

    ```console
    # ping -c 2 172.17.0.3

    PING 172.17.0.3 (172.17.0.3): 56 data bytes
    64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.086 ms
    64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.094 ms

    --- 172.17.0.3 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.086/0.090/0.094 ms
    ```

    该操作应成功。接着尝试按容器名 `alpine2` 进行 ping，会失败：

    ```console
    # ping -c 2 alpine2

    ping: bad address 'alpine2'
    ```

7.  使用分离快捷键 `CTRL`+`p`、`CTRL`+`q` 从 `alpine1` 脱离（容器不停止）。可选择附着到 `alpine2` 重复第 4、5、6 步。

8.  停止并删除两个容器：

    ```console
    $ docker container stop alpine1 alpine2
    $ docker container rm alpine1 alpine2
    ```

请注意：默认 `bridge` 网络不建议用于生产。要了解用户自定义 bridge 网络，请继续[下一节](#use-user-defined-bridge-networks)。

## 使用用户自定义 bridge 网络

本示例再次启动两个 `alpine` 容器，并将它们连接到已创建的用户自定义网络 `alpine-net`。这些容器不会连接到默认 `bridge`。随后我们再启动一个仅连到 `bridge` 的第三个容器，以及一个同时连接两种网络的第四个容器。

1.  创建 `alpine-net` 网络。`--driver bridge` 非必需（默认即为 bridge），此处仅作展示：

    ```console
    $ docker network create --driver bridge alpine-net
    ```

2.  列出 Docker 网络：

    ```console
    $ docker network ls

    NETWORK ID          NAME                DRIVER              SCOPE
    e9261a8c9a19        alpine-net          bridge              local
    17e324f45964        bridge              bridge              local
    6ed54d316334        host                host                local
    7092879f2cc8        none                null                local
    ```

    检查 `alpine-net` 网络，查看其 IP 信息并确认当前无容器连接：

    ```console
    $ docker network inspect alpine-net

    [
        {
            "Name": "alpine-net",
            "Id": "e9261a8c9a19eabf2bf1488bf5f208b99b1608f330cff585c273d39481c9b0ec",
            "Created": "2017-09-25T21:38:12.620046142Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]
    ```

    注意该网络的网关为 `172.18.0.1`，不同于默认 `bridge` 的 `172.17.0.1`。你的系统上数值可能不同。

3.  创建四个容器，注意 `--network` 标志。`docker run` 期间只能连接一个网络，因此需要随后使用 `docker network connect` 将 `alpine4` 也连接到 `bridge`：

    ```console
    $ docker run -dit --name alpine1 --network alpine-net alpine ash

    $ docker run -dit --name alpine2 --network alpine-net alpine ash

    $ docker run -dit --name alpine3 alpine ash

    $ docker run -dit --name alpine4 --network alpine-net alpine ash

    $ docker network connect bridge alpine4
    ```

    确认所有容器正在运行：

    ```console
    $ docker container ls

    CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
    156849ccd902        alpine              "ash"               41 seconds ago       Up 41 seconds                           alpine4
    fa1340b8d83e        alpine              "ash"               51 seconds ago       Up 51 seconds                           alpine3
    a535d969081e        alpine              "ash"               About a minute ago   Up About a minute                       alpine2
    0a02c449a6e9        alpine              "ash"               About a minute ago   Up About a minute                       alpine1
    ```

4.  再次检查 `bridge` 与 `alpine-net`：

    ```console
    $ docker network inspect bridge

    [
        {
            "Name": "bridge",
            "Id": "17e324f459648a9baaea32b248d3884da102dde19396c25b30ec800068ce6b10",
            "Created": "2017-06-22T20:27:43.826654485Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": [
                    {
                        "Subnet": "172.17.0.0/16",
                        "Gateway": "172.17.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {
                "156849ccd902b812b7d17f05d2d81532ccebe5bf788c9a79de63e12bb92fc621": {
                    "Name": "alpine4",
                    "EndpointID": "7277c5183f0da5148b33d05f329371fce7befc5282d2619cfb23690b2adf467d",
                    "MacAddress": "02:42:ac:11:00:03",
                    "IPv4Address": "172.17.0.3/16",
                    "IPv6Address": ""
                },
                "fa1340b8d83eef5497166951184ad3691eb48678a3664608ec448a687b047c53": {
                    "Name": "alpine3",
                    "EndpointID": "5ae767367dcbebc712c02d49556285e888819d4da6b69d88cd1b0d52a83af95f",
                    "MacAddress": "02:42:ac:11:00:02",
                    "IPv4Address": "172.17.0.2/16",
                    "IPv6Address": ""
                }
            },
            "Options": {
                "com.docker.network.bridge.default_bridge": "true",
                "com.docker.network.bridge.enable_icc": "true",
                "com.docker.network.bridge.enable_ip_masquerade": "true",
                "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
                "com.docker.network.bridge.name": "docker0",
                "com.docker.network.driver.mtu": "1500"
            },
            "Labels": {}
        }
    ]
    ```

    容器 `alpine3` 与 `alpine4` 已连接到 `bridge`。

    ```console
    $ docker network inspect alpine-net

    [
        {
            "Name": "alpine-net",
            "Id": "e9261a8c9a19eabf2bf1488bf5f208b99b1608f330cff585c273d39481c9b0ec",
            "Created": "2017-09-25T21:38:12.620046142Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {
                "0a02c449a6e9a15113c51ab2681d72749548fb9f78fae4493e3b2e4e74199c4a": {
                    "Name": "alpine1",
                    "EndpointID": "c83621678eff9628f4e2d52baf82c49f974c36c05cba152db4c131e8e7a64673",
                    "MacAddress": "02:42:ac:12:00:02",
                    "IPv4Address": "172.18.0.2/16",
                    "IPv6Address": ""
                },
                "156849ccd902b812b7d17f05d2d81532ccebe5bf788c9a79de63e12bb92fc621": {
                    "Name": "alpine4",
                    "EndpointID": "058bc6a5e9272b532ef9a6ea6d7f3db4c37527ae2625d1cd1421580fd0731954",
                    "MacAddress": "02:42:ac:12:00:04",
                    "IPv4Address": "172.18.0.4/16",
                    "IPv6Address": ""
                },
                "a535d969081e003a149be8917631215616d9401edcb4d35d53f00e75ea1db653": {
                    "Name": "alpine2",
                    "EndpointID": "198f3141ccf2e7dba67bce358d7b71a07c5488e3867d8b7ad55a4c695ebb8740",
                    "MacAddress": "02:42:ac:12:00:03",
                    "IPv4Address": "172.18.0.3/16",
                    "IPv6Address": ""
                }
            },
            "Options": {},
            "Labels": {}
        }
    ]
    ```

    容器 `alpine1`、`alpine2` 与 `alpine4` 已连接到 `alpine-net`。

5.  在 `alpine-net` 这类用户自定义网络上，容器不仅能按 IP 通信，还可以解析容器名到 IP，这称为自动服务发现。连接到 `alpine1` 做个测试：`alpine1` 应能解析 `alpine2`、`alpine4`（以及自身 `alpine1`）到 IP。
    
    > [!NOTE]
    > 
    > 自动服务发现只能解析自定义容器名，不能解析默认自动生成的容器名。

    ```console
    $ docker container attach alpine1

    # ping -c 2 alpine2

    PING alpine2 (172.18.0.3): 56 data bytes
    64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.085 ms
    64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.090 ms

    --- alpine2 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.085/0.087/0.090 ms

    # ping -c 2 alpine4

    PING alpine4 (172.18.0.4): 56 data bytes
    64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.076 ms
    64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.091 ms

    --- alpine4 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.076/0.083/0.091 ms

    # ping -c 2 alpine1

    PING alpine1 (172.18.0.2): 56 data bytes
    64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.026 ms
    64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.054 ms

    --- alpine1 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.026/0.040/0.054 ms
    ```

6.  在 `alpine1` 内，不应能连接到 `alpine3`，因为它不在 `alpine-net` 上。

    ```console
    # ping -c 2 alpine3

    ping: bad address 'alpine3'
    ```

    同时，你也无法按 IP 连接 `alpine3`。回看 `bridge` 的 `docker network inspect` 输出，找到 `alpine3` 的 IP `172.17.0.2` 并尝试 ping：

    ```console
    # ping -c 2 172.17.0.2

    PING 172.17.0.2 (172.17.0.2): 56 data bytes

    --- 172.17.0.2 ping statistics ---
    2 packets transmitted, 0 packets received, 100% packet loss
    ```

    使用 `CTRL`+`p`、`CTRL`+`q` 从 `alpine1` 脱离。

7.  注意：`alpine4` 同时连接了默认 `bridge` 与 `alpine-net`，因此它应该能访问其他所有容器；但访问 `alpine3` 需按其 IP。附着并测试：

    ```console
    $ docker container attach alpine4

    # ping -c 2 alpine1

    PING alpine1 (172.18.0.2): 56 data bytes
    64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.074 ms
    64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.082 ms

    --- alpine1 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.074/0.078/0.082 ms

    # ping -c 2 alpine2

    PING alpine2 (172.18.0.3): 56 data bytes
    64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.075 ms
    64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.080 ms

    --- alpine2 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.075/0.077/0.080 ms

    # ping -c 2 alpine3
    ping: bad address 'alpine3'

    # ping -c 2 172.17.0.2

    PING 172.17.0.2 (172.17.0.2): 56 data bytes
    64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.089 ms
    64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.075 ms

    --- 172.17.0.2 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.075/0.082/0.089 ms

    # ping -c 2 alpine4

    PING alpine4 (172.18.0.4): 56 data bytes
    64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.033 ms
    64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.064 ms

    --- alpine4 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.033/0.048/0.064 ms
    ```

8.  最后测试：确认所有容器都能访问互联网，依次在 `alpine4`、`alpine3`（仅连接 `bridge`）与 `alpine1`（仅连接 `alpine-net`）中 ping `google.com`。

    ```console
    # ping -c 2 google.com

    PING google.com (172.217.3.174): 56 data bytes
    64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.778 ms
    64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.634 ms

    --- google.com ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 9.634/9.706/9.778 ms

    CTRL+p CTRL+q

    $ docker container attach alpine3

    # ping -c 2 google.com

    PING google.com (172.217.3.174): 56 data bytes
    64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.706 ms
    64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.851 ms

    --- google.com ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 9.706/9.778/9.851 ms

    CTRL+p CTRL+q

    $ docker container attach alpine1

    # ping -c 2 google.com

    PING google.com (172.217.3.174): 56 data bytes
    64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.606 ms
    64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.603 ms

    --- google.com ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 9.603/9.604/9.606 ms

    CTRL+p CTRL+q
    ```

9.  停止并删除所有容器及 `alpine-net` 网络：

    ```console
    $ docker container stop alpine1 alpine2 alpine3 alpine4

    $ docker container rm alpine1 alpine2 alpine3 alpine4

    $ docker network rm alpine-net
    ```


## 其他网络教程

- [Host 网络教程](/manuals/engine/network/tutorials/host.md)
- [Overlay 网络教程](/manuals/engine/network/tutorials/overlay.md)
- [Macvlan 网络教程](/manuals/engine/network/tutorials/macvlan.md)
