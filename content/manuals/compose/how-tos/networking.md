---
description: Docker Compose 如何在容器之间建立网络
keywords: documentation, docs, docker, compose, orchestration, containers, networking
title: Compose 中的网络
linkTitle: 网络
weight: 70
aliases:
- /compose/networking/
---

{{% include "compose-eol.md" %}}

默认情况下，Compose 会为你的应用创建一个
[网络](/reference/cli/docker/network/create.md)。每个服务的容器都会加入默认网络，
它既可以被同一网络中的其他容器访问，也可以通过服务名被发现。

> [!NOTE]
>
> 应用的网络名称基于“项目名”，而项目名默认取自所在目录的名称。
> 你可以使用 [`--project-name` 标志](/reference/cli/docker/compose.md)
> 或 [`COMPOSE_PROJECT_NAME` 环境变量](environment-variables/envvars.md#compose_project_name) 来覆盖该默认值。

例如，假设你的应用位于名为 `myapp` 的目录中，`compose.yaml` 如下所示：

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```

当你运行 `docker compose up` 时，将发生以下情况：

1. 创建名为 `myapp_default` 的网络。
2. 基于 `web` 的配置创建容器，并以名称 `web` 加入 `myapp_default` 网络。
3. 基于 `db` 的配置创建容器，并以名称 `db` 加入 `myapp_default` 网络。

此时，每个容器都可以通过服务名 `web` 或 `db` 进行名称解析并获取对应容器的 IP 地址。
例如，`web` 的应用代码可以连接到 `postgres://db:5432` 来使用 Postgres 数据库。

需要特别注意 `HOST_PORT` 与 `CONTAINER_PORT` 的区别。
在上述示例中，对于 `db`，宿主机端口（`HOST_PORT`）为 `8001`，容器端口为 `5432`
（Postgres 默认端口）。服务间的网络通信使用的是 `CONTAINER_PORT`。
当定义了 `HOST_PORT` 时，该服务也能从宿主机（以及 swarm 外部）进行访问。

在 `web` 容器内，连接 `db` 的连接串应为 `postgres://db:5432`；
从宿主机访问时，连接串形如 `postgres://{DOCKER_IP}:8001`，例如本地运行时可使用 `postgres://localhost:8001`。

## 更新网络中的容器

若你修改了某个服务的配置并通过 `docker compose up` 进行更新，旧容器会被移除，
新容器将以相同的名称、不同的 IP 地址加入网络。正在运行的容器仍可通过该名称解析到新地址，旧的 IP 地址将失效。

如果仍有容器保持着到旧容器的连接，这些连接会被关闭。容器需要自行检测这种情况，
重新解析名称并发起重连。

> [!TIP]
>
> 尽量通过名称而不是 IP 引用容器。否则你需要不断更新所使用的 IP 地址。

## 链接容器

链接允许你为某个服务定义额外的别名，使其可被另一个服务通过这些别名访问。
它并不是实现服务间通信的必需条件：默认情况下，任意服务都可以通过对方的服务名进行访问。
在下面的示例中，`web` 可以通过主机名 `db` 和 `database` 访问 `db`：

```yaml
services:

  web:
    build: .
    links:
      - "db:database"
  db:
    image: postgres
```

更多信息参见 [links 参考](/reference/compose-file/services.md#links)。

## 多主机网络

当在启用 [Swarm 模式](/manuals/engine/swarm/_index.md) 的 Docker Engine 上部署 Compose 应用时，
可以使用内置的 `overlay` 驱动来实现跨主机通信。

Overlay 网络始终以 `attachable` 方式创建。你也可以将 [`attachable`](/reference/compose-file/networks.md#attachable) 属性设置为 `false`。

参阅 [Swarm 模式](/manuals/engine/swarm/_index.md) 了解如何设置 Swarm 集群；
另见[多主机网络入门](/manuals/engine/network/tutorials/overlay.md)以了解多主机 overlay 网络。

## 指定自定义网络

除了仅使用默认的应用网络外，你还可以通过顶层 `networks` 键来定义自定义网络。
这有助于构建更复杂的拓扑，并指定[自定义网络驱动](/engine/extend/plugins_network/)及相关选项。
你也可以将服务连接到由外部创建、非 Compose 管理的网络。

每个服务可以通过服务级别的 `networks` 键指定要连接的网络。该键的值是名称列表，
这些名称引用顶层 `networks` 下的条目。

下面的示例展示了一个定义了两个自定义网络的 Compose 文件。
`proxy` 服务与 `db` 服务彼此隔离，因为它们不共享网络；只有 `app` 同时连接到两者。

```yaml
services:
  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    # 指定驱动选项
    driver: bridge
    driver_opts:
      com.docker.network.bridge.host_binding_ipv4: "127.0.0.1"
  backend:
    # 使用自定义驱动
    driver: custom-driver
```

可以通过为每个已连接的网络设置 [ipv4_address 和/或 ipv6_address](/reference/compose-file/services.md#ipv4_address-ipv6_address) 来配置静态 IP 地址。

你也可以为网络指定[自定义名称](/reference/compose-file/networks.md#name)：

```yaml
services:
  # ...
networks:
  frontend:
    name: custom_frontend
    driver: custom-driver-1
```

## 配置默认网络

除了定义自有网络之外，你还可以在 `networks` 下定义名为 `default` 的条目，
以调整应用范围内的默认网络设置：

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres

networks:
  default:
    # 使用自定义驱动
    driver: custom-driver-1
```

## 使用已有网络

如果你在 Compose 之外使用 `docker network create` 手动创建了一个 bridge 网络，
可以通过将该网络标记为 `external` 来让 Compose 服务连接到它。

如果希望容器加入一个已存在的网络，请使用 [`external` 选项](/reference/compose-file/networks.md#external)：
```yaml
services:
  # ...
networks:
  network1:
    name: my-pre-existing-network
    external: true
```

此时，Compose 不会尝试创建名为 `[projectname]_default` 的网络，而是查找 `my-pre-existing-network` 并将应用的容器连接到该网络。

## 更多参考信息 

有关可用网络配置选项的完整说明，请参阅：

- [顶层 `networks` 元素](/reference/compose-file/networks.md)
- [服务级 `networks` 属性](/reference/compose-file/services.md#networks)
