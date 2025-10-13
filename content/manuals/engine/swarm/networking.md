---
description: 在 Swarm 模式下使用 Overlay 网络功能
keywords: swarm, networking, ingress, overlay, service discovery
title: 管理 Swarm 服务网络
toc_max: 3
---

本文介绍 Swarm 服务的网络相关内容。

## Swarm 与流量类型

Docker Swarm 涉及两类不同的网络流量：

- 控制与管理平面流量：包括 Swarm 的管理消息，例如加入/退出集群的请求等。该类流量始终加密。

- 应用数据平面流量：包括容器之间的通信，以及与外部客户端之间的流量。

## 关键网络概念

以下三个网络概念对 Swarm 服务尤为重要：

- Overlay 网络用于管理参与 Swarm 的各个 Docker 守护进程之间的通信。你可以像为独立容器创建自定义网络一样创建 overlay 网络。你也可以将某个服务连接到一个或多个现有 overlay 网络，从而实现服务间通信。Overlay 网络本质上是使用 `overlay` 网络驱动的 Docker 网络。

- `ingress` 网络是一个特殊的 overlay 网络，用于在某个服务的多个节点间进行负载均衡。当任一 Swarm 节点在已发布端口上收到请求时，会将该请求交给名为 `IPVS` 的模块处理。`IPVS` 维护该服务内所有参与容器的 IP 列表，选择其中一个目标，并通过 `ingress` 网络将请求路由过去。

  在初始化或加入 Swarm 时，`ingress` 网络会自动创建。大多数场景无需自定义其配置，但 Docker 也允许你进行自定义。

- `docker_gwbridge` 是一个桥接网络，用于将 overlay 网络（包括 `ingress`）连接到单个 Docker 守护进程所在主机的物理网络。默认情况下，服务运行的每个容器都会连接到其所在宿主机的 `docker_gwbridge` 网络。

  当你初始化或加入 Swarm 时，`docker_gwbridge` 会自动创建。通常也无需自定义其配置，但 Docker 提供了可选的自定义能力。

> [!TIP]
>
> 想要了解更多 Swarm 网络整体信息，请参阅《[网络概览](/manuals/engine/network/_index.md)》。

## 防火墙注意事项

参与 Swarm 的各个 Docker 守护进程需要能够通过以下端口相互通信：

* 端口 `7946` TCP/UDP：用于容器网络发现。
* 端口 `4789` UDP（可配置）：用于 overlay 网络（包括 ingress）的数据路径。

在搭建 Swarm 网络时，需要特别注意上述端口设置。可以参考《[教程](swarm-tutorial/_index.md#open-protocols-and-ports-between-the-hosts)》了解概要说明。

## Overlay 网络

当你初始化 Swarm 或将某台 Docker 主机加入已有的 Swarm 时，该主机会自动创建两个网络：

- 一个名为 `ingress` 的 overlay 网络，用于处理与 Swarm 服务相关的控制与数据流量。当你创建某个 Swarm 服务且未显式连接到自定义 overlay 网络时，它会默认连接到 `ingress`。
- 一个名为 `docker_gwbridge` 的桥接网络，用于将该 Docker 守护进程与 Swarm 中其他守护进程连接起来。

### 创建 overlay 网络

要创建 overlay 网络，请在 `docker network create` 中指定 `overlay` 驱动：

```console
$ docker network create \
  --driver overlay \
  my-network
```

上述命令未指定任何自定义选项，因此 Docker 会自动分配子网并使用默认配置。可通过 `docker network inspect` 查看网络详情。

当 overlay 网络上尚无容器连接时，网络配置看起来较为“空”：

```console
$ docker network inspect my-network
[
    {
        "Name": "my-network",
        "Id": "fsf1dmx3i9q75an49z36jycxd",
        "Created": "0001-01-01T00:00:00Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "Containers": null,
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": null
    }
]
```

在以上输出中，可以看到网络驱动为 `overlay`，作用域为 `swarm`（而非 `local`、`host` 或 `global`）。这表示仅参与该 Swarm 的主机才能访问此网络。

当第一个服务连接到该网络时，网络的子网与网关会被动态配置。下面示例展示了同一个网络在连接了 `redis` 服务的三个容器之后的情况：

```console
$ docker network inspect my-network
[
    {
        "Name": "my-network",
        "Id": "fsf1dmx3i9q75an49z36jycxd",
        "Created": "2017-05-31T18:35:58.877628262Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "Containers": {
            "0e08442918814c2275c31321f877a47569ba3447498db10e25d234e47773756d": {
                "Name": "my-redis.1.ka6oo5cfmxbe6mq8qat2djgyj",
                "EndpointID": "950ce63a3ace13fe7ef40724afbdb297a50642b6d47f83a5ca8636d44039e1dd",
                "MacAddress": "02:42:0a:00:00:03",
                "IPv4Address": "10.0.0.3/24",
                "IPv6Address": ""
            },
            "88d55505c2a02632c1e0e42930bcde7e2fa6e3cce074507908dc4b827016b833": {
                "Name": "my-redis.2.s7vlybipal9xlmjfqnt6qwz5e",
                "EndpointID": "dd822cb68bcd4ae172e29c321ced70b731b9994eee5a4ad1d807d9ae80ecc365",
                "MacAddress": "02:42:0a:00:00:05",
                "IPv4Address": "10.0.0.5/24",
                "IPv6Address": ""
            },
            "9ed165407384f1276e5cfb0e065e7914adbf2658794fd861cfb9b991eddca754": {
                "Name": "my-redis.3.hbz3uk3hi5gb61xhxol27hl7d",
                "EndpointID": "f62c686a34c9f4d70a47b869576c37dffe5200732e1dd6609b488581634cf5d2",
                "MacAddress": "02:42:0a:00:00:04",
                "IPv4Address": "10.0.0.4/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": {},
        "Peers": [
            {
                "Name": "moby-e57c567e25e2",
                "IP": "192.168.65.2"
            }
        ]
    }
]
```

### 自定义 overlay 网络

在某些场景中，你可能不希望使用 overlay 网络的默认配置。可运行 `docker network create --help` 查看全部可配置项。以下列出一些常见的自定义选项。

#### 配置子网与网关

默认情况下，当第一个服务连接到网络时，网络的子网与网关会自动配置。你也可以在创建网络时使用 `--subnet` 与 `--gateway` 明确指定。如下示例在前文基础上补充了子网与网关配置：

```console
$ docker network create \
  --driver overlay \
  --subnet 10.0.9.0/24 \
  --gateway 10.0.9.99 \
  my-network
```

##### 使用自定义默认地址池

要自定义 Swarm 网络的子网分配，可在执行 `swarm init` 时进行[可选配置](swarm-mode.md)。

例如，初始化 Swarm 时可执行：

```console
$ docker swarm init --default-addr-pool 10.20.0.0/16 --default-addr-pool-mask-length 26
```

之后，当用户创建网络且未使用 `--subnet` 指定子网时，系统会从地址池中按顺序分配下一个可用子网；若该子网已被占用，则不会用于 Swarm。

如需非连续的地址空间，可配置多个地址池。但暂不支持从特定地址池定向分配。子网会按顺序从 IP 地址池中分配，并在网络被删除后回收再用。

默认子网掩码长度可配置，且对所有网络一致。默认值为 `/24`；可通过 `--default-addr-pool-mask-length` 修改。

> [!NOTE]
>
> 默认地址池只能在 `swarm init` 时配置，集群创建后无法修改。

##### Overlay 网络规模限制

Docker 建议将 overlay 网络创建为 `/24` 网段。`/24` 网段最多提供 256 个 IP 地址。

该建议是基于 Swarm 模式的[已知限制](https://github.com/moby/moby/issues/30820)。
如果你需要超过 256 个 IP 地址，不要扩大网段大小。可以考虑使用带外部负载均衡器的 `dnsrr` 端点模式，或拆分为多个较小的 overlay 网络。参见「[配置服务发现](#configure-service-discovery)」了解不同端点模式。

#### 配置应用数据加密 {#encryption}

与 Swarm 相关的管理与控制平面数据始终会被加密。关于加密机制的细节，请参阅《[Swarm 模式 Overlay 网络安全模型](/manuals/engine/network/drivers/overlay.md)》。

Swarm 节点之间的应用数据默认不加密。若要对某个 overlay 网络上的此类流量进行加密，可在 `docker network create` 时添加 `--opt encrypted`。这会在 VXLAN 层启用 IPsec 加密。请注意，加密会带来一定性能开销，建议在生产前充分评估。

> [!NOTE]
>
> 若希望对 `ingress` 网络启用加密，你需要先[自定义自动创建的 ingress](#customize-ingress)。默认情况下，`ingress` 上的流量不加密，因为加密属于网络级别的可选项。

## 将服务连接到 overlay 网络

要将服务连接到现有 overlay 网络，可在 `docker service create` 中添加 `--network`，或在 `docker service update` 中使用 `--network-add`。

```console
$ docker service create \
  --replicas 3 \
  --name my-web \
  --network my-network \
  nginx
```

连接到同一 overlay 网络的服务容器可以彼此通信。

要查看某个服务连接了哪些网络，可以先用 `docker service ls` 找到服务名，再用 `docker service ps <service-name>` 列出网络。反过来，若要查看某个网络连接了哪些服务容器，可使用 `docker network inspect <network-name>`。这些命令可在任意已加入且处于 `running` 状态的 Swarm 节点上执行。

### 配置服务发现

服务发现是 Docker 用来把外部客户端的请求路由到某个具体 Swarm 节点的机制，客户端无需关心服务后端参与的节点数量、IP 地址或端口。同一网络内的服务间通信无需对外发布端口。例如，若你有一个[将数据存储在 MySQL 服务中的 WordPress 服务](https://training.play-with-docker.com/swarm-service-discovery/)，且它们连接到同一 overlay 网络，则无需向客户端发布 MySQL 端口，只需发布 WordPress 的 HTTP 端口。

服务发现可以有两种方式：
— 基于内置 DNS 与虚拟 IP（VIP）的三/四层连接级负载均衡；
— 基于 DNS 轮询（DNSRR）的七层请求级负载均衡（通常配合外部负载均衡器）。
你可以按服务粒度进行配置。

- 默认情况下，当你将服务连接到网络且发布了一个或多个端口，Docker 会为该服务分配一个虚拟 IP（VIP），作为客户端访问该服务的“前端”。Docker 维护该服务的所有工作节点列表，并在客户端与其中某个节点之间进行请求转发。来自客户端的每个请求可能被路由到不同的节点。

- 如果将服务配置为使用 DNS 轮询（DNSRR）进行服务发现，则不会存在唯一的虚拟 IP。Docker 会为服务创建 DNS 记录，使对服务名的查询返回一组 IP，客户端会直接连到其中之一。

  当你打算使用自有负载均衡器（如 HAProxy）时，DNS 轮询非常有用。要启用 DNSRR，可在创建或更新服务时添加 `--endpoint-mode dnsrr`。

## 自定义 ingress 网络 {#customize-ingress}

多数用户无需配置 `ingress` 网络，但 Docker 允许这样做。当自动选择的子网与你现有网络冲突、或你需要自定义 MTU 等底层网络设置，或你想要[启用加密](#encryption)时，这会很有用。

自定义 `ingress` 网络需要先删除再重建。通常应在创建任何服务之前进行此操作。若已有对外发布端口的服务，必须先删除这些服务后才能移除 `ingress` 网络。

在 `ingress` 网络不存在的期间，未发布端口的现有服务可继续运行，但不会进行负载均衡。这会影响那些发布端口的服务，例如对外发布 80 端口的 WordPress 服务。

1.  使用 `docker network inspect ingress` 检查 `ingress` 网络，并删除所有连接到它的服务容器。通常这些都是发布了端口的服务，如发布 80 端口的 WordPress 服务。若这些服务未完全停止，下一步将失败。

2.  删除现有的 `ingress` 网络：

    ```console
    $ docker network rm ingress

    WARNING! 在移除 routing-mesh 网络之前，请确保集群中所有节点运行相同版本的 Docker Engine。否则，移除可能不会生效，并影响新创建的 ingress 网络功能。
    Are you sure you want to continue? [y/N]
    ```

3.  使用 `--ingress` 标志创建新的 overlay 网络，并按需设置自定义选项。以下示例将 MTU 设为 1200，子网设为 `10.11.0.0/16`，网关设为 `10.11.0.2`：

    ```console
    $ docker network create \
      --driver overlay \
      --ingress \
      --subnet=10.11.0.0/16 \
      --gateway=10.11.0.2 \
      --opt com.docker.network.driver.mtu=1200 \
      my-ingress
    ```

    > [!NOTE]
    >
    > 你可以把 `ingress` 网络命名为其他名称，但集群中同时只能有一个。尝试创建第二个将失败。

4.  重启第一步中停止的那些服务。

## 自定义 docker_gwbridge

`docker_gwbridge` 是一个虚拟网桥，用于将 overlay 网络（包括 `ingress`）连接到单个 Docker 守护进程的物理网络。当你初始化或将宿主机加入 Swarm 时，它会被自动创建。它并非 Docker 设备，而是存在于宿主机内核中的网桥。若需自定义其配置，必须在主机加入 Swarm 之前，或将主机临时移出 Swarm 之后进行。

删除已有网桥需要操作系统安装 `brctl` 工具（软件包名为 `bridge-utils`）。

1.  停止 Docker。

2.  使用 `brctl show docker_gwbridge` 检查是否存在名为 `docker_gwbridge` 的网桥；若存在，使用 `brctl delbr docker_gwbridge` 将其删除。

3.  启动 Docker，但先不要加入或初始化 Swarm。

4.  使用你的自定义设置创建或重建 `docker_gwbridge` 网桥。以下示例使用子网 `10.11.0.0/16`。更多可自定义选项参见《[Bridge 驱动选项](/reference/cli/docker/network/create.md#bridge-driver-options)》。

    ```console
    $ docker network create \
    --subnet 10.11.0.0/16 \
    --opt com.docker.network.bridge.name=docker_gwbridge \
    --opt com.docker.network.bridge.enable_icc=false \
    --opt com.docker.network.bridge.enable_ip_masquerade=true \
    docker_gwbridge
    ```

5.  初始化或加入 Swarm。

## 为控制与数据流量使用独立网卡

默认情况下，所有 Swarm 流量都通过同一网卡发送，包括用于维持集群的控制/管理流量，以及服务容器的进出数据流量。

你可以在初始化或加入 Swarm 时通过 `--data-path-addr` 来实现控制/数据流量分离。若存在多块网卡，需要显式指定 `--advertise-addr`；若未指定，`--data-path-addr` 将默认与 `--advertise-addr` 相同。关于加入、退出与管理 Swarm 的流量会走 `--advertise-addr` 指定的接口，服务容器之间的流量会走 `--data-path-addr` 指定的接口。这两个参数都可以是 IP 地址或网卡名（如 `eth0`）。

下面示例展示在初始化 Swarm 时设置独立的 `--data-path-addr`。假定你的宿主机有两块网卡：10.0.0.1 用于控制/管理流量，192.168.0.1 用于服务相关流量。

```console
$ docker swarm init --advertise-addr 10.0.0.1 --data-path-addr 192.168.0.1
```

下面示例演示加入由 `192.168.99.100:2377` 管理的 Swarm，同时将 `--advertise-addr` 设为 `eth0`，`--data-path-addr` 设为 `eth1`：

```console
$ docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2d7c \
  --advertise-addr eth0 \
  --data-path-addr eth1 \
  192.168.99.100:2377
```

## 在 overlay 网络上发布端口

连接到同一 overlay 网络的 Swarm 服务彼此之间等价于端口全通。若某个端口需要对集群外部可达，必须在 `docker service create` 或 `docker service update` 中使用 `-p` 或 `--publish` 进行“发布”。支持旧的冒号分隔语法与新的逗号分隔语法。推荐使用较长的新语法，因为表达更加自描述。

<table>
<thead>
<tr>
<th>参数值</th>
<th>说明</th>
</tr>
</thead>
<tr>
<td><tt>-p 8080:80</tt> 或<br /><tt>-p published=8080,target=80</tt></td>
<td>将服务的 TCP 80 端口映射到路由网格的 8080 端口。</td>
</tr>
<tr>
<td><tt>-p 8080:80/udp</tt> 或<br /><tt>-p published=8080,target=80,protocol=udp</tt></td>
<td>将服务的 UDP 80 端口映射到路由网格的 8080 端口。</td>
</tr>
<tr>
<td><tt>-p 8080:80/tcp -p 8080:80/udp</tt> 或 <br /><tt>-p published=8080,target=80,protocol=tcp -p published=8080,target=80,protocol=udp</tt></td>
<td>将服务的 TCP 80 端口映射到路由网格的 TCP 8080 端口，同时将服务的 UDP 80 端口映射到路由网格的 UDP 8080 端口。</td>
</tr>
</table>

## 延伸阅读

* [将服务部署到 Swarm](services.md)
* [Swarm 管理指南](admin_guide.md)
* [Swarm 模式入门教程](swarm-tutorial/_index.md)
* [网络概览](/manuals/engine/network/_index.md)
* [Docker CLI 参考](/reference/cli/docker/)
