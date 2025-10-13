---
title: 覆盖容器默认设置
weight: 2
keywords: concepts, build, images, container, docker desktop
description: 本文将演示如何通过 `docker run` 覆盖容器的默认设置。
aliases: 
 - /guides/docker-concepts/running-containers/overriding-container-defaults/
---

{{< youtube-embed PFszWK3BB8I >}}

## 概念解析

当 Docker 容器启动时，会执行某个应用或命令。这个可执行脚本或文件来自镜像的配置。镜像通常带有一组默认设置，足以满足多数场景；但在需要时你可以对其进行覆盖，以便让容器内的程序按你的期望运行。

例如，你已有一个监听标准端口的数据库容器，现在又想启动一个相同镜像的新实例，就可能需要更改新容器的监听端口，以避免与现有容器冲突。有时你也会希望增加容器可用内存，以应对高负载；或通过设置环境变量向程序传递必要的配置。

`docker run` 提供了一组强大的参数，允许你在启动时覆盖默认行为，并按需定制容器。下面是几种常用方式。

### 覆盖网络端口

在开发与测试中，可能需要运行多套数据库实例。若它们都使用同一个端口，就会发生冲突。可以通过 `docker run` 的 `-p` 参数映射容器端口到宿主机端口，从而让多个实例同时运行而不冲突：

```console
$ docker run -d -p HOST_PORT:CONTAINER_PORT postgres
```

### 设置环境变量

下面示例会在容器内设置名为 `foo`、值为 `bar` 的环境变量：

```console
$ docker run -e foo=bar postgres env
```

你将看到类似如下输出：

```console
HOSTNAME=2042f2e6ebe4
foo=bar
```

> [!TIP]
>
> `.env` 文件是为容器批量设置环境变量的便捷方式，避免命令行堆叠大量 `-e`。可通过 `--env-file` 与 `docker run` 配合使用：
> ```console
> $ docker run --env-file .env postgres env
> ```

### 限制容器资源占用

可以在 `docker run` 中使用 `--memory` 与 `--cpus` 来限制容器可用的内存与 CPU。例如，为 Python API 设置内存上限，避免容器过度消耗宿主机资源：

```console
$ docker run -e POSTGRES_PASSWORD=secret --memory="512m" --cpus="0.5" postgres
```

上述命令将内存限制为 512 MB，并将 CPU 配额设为 0.5（半个核心）。

> **实时监控资源使用**
>
> 可使用 `docker stats` 监控正在运行容器的实时资源使用情况，以便判断配额是否需要调整。

通过合理使用这些 `docker run` 参数，你可以让容器化应用的行为更契合你的具体需求。

## 动手试试

本实践将演示如何使用 `docker run` 覆盖容器默认设置。

1. [下载并安装](/get-started/get-docker/) Docker Desktop。

### 运行多个 Postgres 实例

1. 使用 [Postgres 镜像](https://hub.docker.com/_/postgres) 启动一个容器：
    
    ```console
    $ docker run -d -e POSTGRES_PASSWORD=secret -p 5432:5432 postgres
    ```

    该命令会在后台启动一个 Postgres 数据库，容器内部监听标准端口 `5432`，并映射到宿主机的 `5432`。

2. 启动第二个 Postgres 容器，并映射到不同的宿主机端口： 

    ```console
    $ docker run -d -e POSTGRES_PASSWORD=secret -p 5433:5432 postgres
    ```

    该命令会在容器内仍监听 `5432`，但映射到宿主机的 `5433`。通过覆盖宿主机端口，避免与已运行实例发生冲突。

3. 打开 Docker Desktop 控制面板的 **Containers** 视图，验证两个容器均在运行。

    ![Docker Desktop Dashboard 截图，显示正在运行的多个 Postgres 实例](images/running-postgres-containers.webp?border=true)

### 在自定义网络中运行 Postgres 容器

默认情况下，容器会自动连接到一个名为 bridge 的特殊网络。bridge 网络好比一个虚拟交换桥，允许同一宿主机上的容器互相通信，同时与外部世界和其他宿主机保持隔离。这对多数场景很方便。但在某些情况下，你可能需要对网络进行更精细的控制。

这时就可以创建自定义网络。通过在 `docker run` 中传入 `--network` 参数来指定网络；未显式指定的容器会加入默认的 bridge 网络。

按照以下步骤，将 Postgres 容器连接到自定义网络：

1. 创建一个名为 `mynetwork` 的自定义网络：

    ```console
    $ docker network create mynetwork
    ```

2. 通过以下命令查看网络是否创建成功：

    ```console
    $ docker network ls
    ```

    你会在列表中看到新创建的 "mynetwork"。

3. 将 Postgres 连接到该自定义网络：

    ```console
    $ docker run -d -e POSTGRES_PASSWORD=secret -p 5434:5432 --network mynetwork postgres
    ```

    该命令会在后台启动一个 Postgres 容器，映射宿主机端口 5434，并连接到 `mynetwork`。通过 `--network` 覆盖默认网络设置，使容器接入自定义网络，以便获得更好的隔离与与其他容器的可控通信。可使用 `docker network inspect` 检查容器是否已加入该网络。

    > **默认 bridge 与自定义网络的关键差异**
    >
    > 1. DNS 解析：连接到默认 bridge 网络的容器彼此通信通常需使用 IP（`--link` 已属过时特性且不建议在生产使用，原因见[差异说明](/engine/network/drivers/bridge/#differences-between-user-defined-bridges-and-the-default-bridge)）。接入自定义网络后，容器可通过名称或别名解析。
    > 2. 隔离性：未显式指定 `--network` 的容器都会加入默认 bridge 网络，彼此可通信，存在一定风险。自定义网络提供了作用域隔离，仅同属该网络的容器可互通，从而带来更好的隔离。

### 管理资源配额

默认情况下，容器不会被限制资源使用。在多用户或共享系统上，合理的资源治理十分关键，不应允许某个容器占用过多内存或 CPU。

再次用到 `docker run` 的 `--memory` 与 `--cpus` 参数：

```console
$ docker run -d -e POSTGRES_PASSWORD=secret --memory="512m" --cpus=".5" postgres
```

其中 `--cpus` 指定 CPU 配额（此处为 0.5 个核心），`--memory` 指定内存上限（此处为 512 MB）。

### 在 Docker Compose 中覆盖默认的 CMD 与 ENTRYPOINT

在使用 Docker Compose 时，你也可能需要覆盖镜像中定义的默认命令（`CMD`）或入口点（`ENTRYPOINT`）。

1. 新建 `compose.yml`，内容如下：

    ```yaml
    services:
      postgres:
        image: postgres
        entrypoint: ["docker-entrypoint.sh", "postgres"]
        command: ["-h", "localhost", "-p", "5432"]
        environment:
          POSTGRES_PASSWORD: secret 
    ```

    上述 Compose 文件定义了一个名为 `postgres` 的服务，使用官方 Postgres 镜像，设置了入口点脚本，并以密码认证方式启动容器。

2. 运行以下命令启动服务：

    ```console
    $ docker compose up -d
    ```

    该命令会启动 Compose 文件中定义的 Postgres 服务。

3. 在 Docker Desktop 控制面板中验证认证情况。

    打开 Docker Desktop，选中 **Postgres** 容器，点击 **Exec** 进入容器 Shell，执行以下命令连接数据库：

    ```console
    # psql -U postgres
    ```

    ![Docker Desktop Dashboard 截图：选择 Postgres 容器并通过 EXEC 进入其 Shell](images/exec-into-postgres-container.webp?border=true)

    > [!NOTE]
    > 
    > PostgreSQL 镜像会在本地使用 trust 认证，因此在同一容器内（localhost）连接时可能无需密码。但从其他主机/容器连接时需要密码。

### 使用 `docker run` 覆盖默认 CMD 与 ENTRYPOINT

也可以直接通过 `docker run` 覆盖默认设置：

```console 
$ docker run -e POSTGRES_PASSWORD=secret postgres docker-entrypoint.sh -h localhost -p 5432
```

该命令会运行一个 Postgres 容器，设置密码认证所需的环境变量，并覆盖默认的启动命令以配置主机名与端口。

## 延伸阅读

* [在 Compose 中设置环境变量的多种方式](/compose/how-tos/environment-variables/set-environment-variables/)
* [什么是容器](/get-started/docker-concepts/the-basics/what-is-a-container/)

## 下一步

现在你已经了解如何覆盖容器默认设置，接下来学习如何持久化容器数据。

{{< button text="持久化容器数据" url="persisting-container-data" >}}

