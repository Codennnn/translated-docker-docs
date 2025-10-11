---
title: 使用 overlay 网络进行联网
description: 在多台 Docker 守护进程上，针对 Swarm 服务与独立容器的联网教程
keywords: networking, bridge, routing, ports, swarm, overlay
aliases:
- /engine/userguide/networking/get-started-overlay/
- /network/network-tutorial-overlay/
---

本系列教程主要介绍 Swarm 服务的网络配置。
若需了解独立容器的联网方法，请参见[独立容器联网](/manuals/engine/network/tutorials/standalone.md)。
如需了解 Docker 网络的通用概念，请参见[概览](/manuals/engine/network/_index.md)。

本页包含以下教程。除最后一个示例需要额外一台 Docker 主机外，其余均可在 Linux、Windows 或 macOS 上运行。

- [使用默认 overlay 网络](#use-the-default-overlay-network)：演示初始化或加入 swarm 时 Docker 自动创建的默认 overlay 网络的用法。该网络不适用于生产环境。

- [使用用户自定义 overlay 网络](#use-a-user-defined-overlay-network)：演示如何创建并使用自定义 overlay 网络连接服务。推荐在生产环境使用。

- [为独立容器使用 overlay 网络](#use-an-overlay-network-for-standalone-containers)：演示如何让运行在不同 Docker 守护进程上的独立容器通过 overlay 网络通信。

## 前提条件

需要至少一个单节点 swarm，即在主机上启动 Docker 并执行 `docker swarm init`。这些示例同样适用于多节点 swarm。

## 使用默认 overlay 网络

本示例将启动一个 `alpine` 服务，并从服务容器视角观察网络特性。

本教程不涉及各操作系统对 overlay 的实现细节，而是关注其在服务视角下的行为。

### 前提条件

本示例需要三台能够互相通信的物理或虚拟 Docker 主机。假设三台主机处于同一网络且无防火墙干扰。

本文将它们称为 `manager`、`worker-1` 与 `worker-2`。`manager` 同时承担管理与工作节点角色（既可运行任务也可管理 swarm），`worker-1` 与 `worker-2` 仅作为工作节点。

如果手头没有三台主机，可以在云服务（如 Amazon EC2）上准备三台 Ubuntu，置于同一网络并允许相互通信（例如通过安全组放通），然后按照[在 Ubuntu 上安装 Docker Engine - Community](/manuals/engine/install/ubuntu.md) 的说明进行配置。

### 操作演练

#### 创建 swarm

完成以下步骤后，三台主机都将加入 swarm，并通过名为 `ingress` 的 overlay 网络互联。

1.  在 `manager` 上初始化 swarm。若主机只有一个网络接口，可省略 `--advertise-addr`：

    ```console
    $ docker swarm init --advertise-addr=<IP-ADDRESS-OF-MANAGER>
    ```

    记录输出信息，其中包含将 `worker-1` 与 `worker-2` 加入 swarm 所需的 token。建议将 token 保存在密码管理器中。

2.  在 `worker-1` 上加入 swarm。若主机只有一个网络接口，可省略 `--advertise-addr`：

    ```console
    $ docker swarm join --token <TOKEN> \
      --advertise-addr <IP-ADDRESS-OF-WORKER-1> \
      <IP-ADDRESS-OF-MANAGER>:2377
    ```

3.  在 `worker-2` 上加入 swarm。若主机只有一个网络接口，可省略 `--advertise-addr`：

    ```console
    $ docker swarm join --token <TOKEN> \
      --advertise-addr <IP-ADDRESS-OF-WORKER-2> \
      <IP-ADDRESS-OF-MANAGER>:2377
    ```

4.  在 `manager` 上列出全部节点（仅管理节点可执行）：

    ```console
    $ docker node ls

    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
    d68ace5iraw6whp7llvgjpu48 *   ip-172-31-34-146    Ready               Active              Leader
    nvp5rwavvb8lhdggo8fcf7plg     ip-172-31-35-151    Ready               Active
    ouvx2l7qfcxisoyms8mtkgahw     ip-172-31-36-89     Ready               Active
    ```

    也可使用 `--filter` 按角色过滤：

    ```console
    $ docker node ls --filter role=manager

    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
    d68ace5iraw6whp7llvgjpu48 *   ip-172-31-34-146    Ready               Active              Leader

    $ docker node ls --filter role=worker

    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
    nvp5rwavvb8lhdggo8fcf7plg     ip-172-31-35-151    Ready               Active
    ouvx2l7qfcxisoyms8mtkgahw     ip-172-31-36-89     Ready               Active
    ```

5.  分别在 `manager`、`worker-1` 与 `worker-2` 上列出 Docker 网络，注意每台主机上都有名为 `ingress` 的 overlay 网络与名为 `docker_gwbridge` 的 bridge 网络。下方仅示例 `manager` 的输出：

    ```console
    $ docker network ls

    NETWORK ID          NAME                DRIVER              SCOPE
    495c570066be        bridge              bridge              local
    961c6cae9945        docker_gwbridge     bridge              local
    ff35ceda3643        host                host                local
    trtnl4tqnc3n        ingress             overlay             swarm
    c8357deec9cb        none                null                local
    ```

`docker_gwbridge` 将 `ingress` 网络与主机网络接口连接起来，使管理节点与工作节点之间的流量可达。若创建服务时未显式指定网络，服务将连接到 `ingress`。建议为每个应用或应用组使用独立的 overlay 网络。接下来我们将创建两个 overlay 网络，并将一个服务依次连接到它们。

#### 创建服务

1.  在 `manager` 上创建名为 `nginx-net` 的 overlay 网络：

    ```console
    $ docker network create -d overlay nginx-net
    ```

    无需在其他节点手动创建；当节点运行到需要该网络的任务时，会自动创建。

2.  在 `manager` 上创建一个连接到 `nginx-net` 的 Nginx 服务，副本数为 5，并向外发布 80 端口。所有任务容器无需额外开放端口即可互相通信。

    > [!NOTE]
    >
    > 只能在管理节点上创建服务。

    ```console
    $ docker service create \
      --name my-nginx \
      --publish target=80,published=80 \
      --replicas=5 \
      --network nginx-net \
      nginx
      ```

      当未为 `--publish` 指定 `mode` 时，默认使用 `ingress` 模式。这意味着无论你访问 `manager`、`worker-1` 还是 `worker-2` 的 80 端口，都会被转发到 5 个任务中的某一个，即使该节点当前并未运行任务。如需使用 `host` 模式发布端口，可在 `--publish` 中添加 `mode=host`；但此时应改用 `--mode global` 而非 `--replicas=5`，因为在每个节点上一个端口只能被一个任务占用。

3.  运行 `docker service ls` 观察服务启动进度（可能需要几秒）。

4.  分别在 `manager`、`worker-1` 与 `worker-2` 上检查 `nginx-net`。无需在 `worker-1` 与 `worker-2` 手动创建，它们会按需自动创建。输出较长，关注 `Containers` 与 `Peers` 小节：`Containers` 列出该主机上连接到该 overlay 的所有任务（或独立容器）。

5.  在 `manager` 上使用 `docker service inspect my-nginx` 查看服务详情，关注服务使用的端口与端点信息。

6.  创建新网络 `nginx-net-2`，并将服务从 `nginx-net` 切换到该网络：

    ```console
    $ docker network create -d overlay nginx-net-2
    ```

    ```console
    $ docker service update \
      --network-add nginx-net-2 \
      --network-rm nginx-net \
      my-nginx
    ```

7.  通过 `docker service ls` 确认服务已更新且任务完成重部署。执行 `docker network inspect nginx-net` 确认该网络已无容器连接；对 `nginx-net-2` 执行同样命令，可见所有任务容器均已连接其上。

    > [!NOTE]
    >
    > 虽然 overlay 网络会在工作节点上按需自动创建，但不会自动移除。

8.  清理服务与网络：在 `manager` 上运行以下命令，管理节点会指挥工作节点自动移除网络。

    ```console
    $ docker service rm my-nginx
    $ docker network rm nginx-net nginx-net-2
    ```

## 使用用户自定义 overlay 网络

### 前提条件

假设 swarm 已就绪，且当前处于管理节点上。

### 操作演练

1.  创建用户自定义 overlay 网络：

    ```console
    $ docker network create -d overlay my-overlay
    ```

2.  启动一个使用该 overlay 网络的服务，并将容器 80 端口发布到主机 8080 端口：

    ```console
    $ docker service create \
      --name my-nginx \
      --network my-overlay \
      --replicas 1 \
      --publish published=8080,target=80 \
      nginx:latest
    ```

3.  执行 `docker network inspect my-overlay`，在 `Containers` 小节确认 `my-nginx` 任务已连接其上。

4.  清理服务与网络：

    ```console
    $ docker service rm my-nginx

    $ docker network rm my-overlay
    ```

## 为独立容器使用 overlay 网络

本示例演示 DNS 容器发现，具体为：如何让位于不同 Docker 守护进程上的独立容器通过 overlay 网络通信。步骤如下：

- 在 `host1` 上初始化 swarm（管理节点）。
- 在 `host2` 上加入该 swarm（工作节点）。
- 在 `host1` 上创建可附着的 overlay 网络（`test-net`）。
- 在 `host1` 上运行一个连接到 `test-net` 的交互式 [alpine](https://hub.docker.com/_/alpine/) 容器（`alpine1`）。
- 在 `host2` 上运行一个连接到 `test-net` 的交互式（且后台运行） [alpine](https://hub.docker.com/_/alpine/) 容器（`alpine2`）。
- 在 `host1` 上，从 `alpine1` 的交互会话中 ping `alpine2`。

### 前提条件

该示例需要两台能够相互通信的 Docker 主机。两台主机之间需要放通以下端口：

- TCP 2377
- TCP 与 UDP 7946
- UDP 4789

一个简单的方式是准备两台虚拟机（本地或云上，如 AWS），均安装并运行 Docker。若使用 AWS 或类似平台，可通过安全组在两台主机之间放通所有入站端口，并仅对你的客户端 IP 开放 SSH。

本文将 swarm 的两台节点称为 `host1` 与 `host2`。示例使用 Linux 主机，但相同命令同样适用于 Windows。

### 操作演练

1.  准备 swarm。

    a.  在 `host1` 上初始化 swarm（如有提示，使用 `--advertise-addr` 指定用于与其他主机通信的接口 IP，例如 AWS 的私网 IP）：


    ```console
    $ docker swarm init
    Swarm initialized: current node (vz1mm9am11qcmo979tlrlox42) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-5g90q48weqrtqryq4kj6ow0e8xm9wmv9o6vgqc5j320ymybd5c-8ex8j0bc40s6hgvy5ui5gl4gy 172.31.47.252:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

    b.  在 `host2` 上按上述提示加入 swarm：

    ```console
    $ docker swarm join --token <your_token> <your_ip_address>:2377
    This node joined a swarm as a worker.
    ```

    如果加入失败导致 `docker swarm join` 超时，请在 `host2` 上执行 `docker swarm leave --force`，检查网络与防火墙设置后重试。

2.  在 `host1` 上创建名为 `test-net` 的可附着 overlay 网络：

    ```console
    $ docker network create --driver=overlay --attachable test-net
    uqsof8phj3ak0rq9k86zta6ht
    ```

    > 请注意返回的 **NETWORK ID** —— 在 `host2` 连接该网络时会再次看到它。

3.  在 `host1` 上启动连接到 `test-net` 的交互式（`-it`）容器（`alpine1`）：

    ```console
    $ docker run -it --name alpine1 --network test-net alpine
    / #
    ```

4.  在 `host2` 上列出可用网络——此时 `test-net` 尚不存在：

    ```console
    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    ec299350b504        bridge              bridge              local
    66e77d0d0e9a        docker_gwbridge     bridge              local
    9f6ae26ccb82        host                host                local
    omvdxqrda80z        ingress             overlay             swarm
    b65c952a4b2b        none                null                local
    ```

5.  在 `host2` 上启动一个连接 `test-net` 的后台（`-d`）交互式（`-it`）容器（`alpine2`）：

    ```console
    $ docker run -dit --name alpine2 --network test-net alpine
    fb635f5ece59563e7b8b99556f816d24e6949a5f6a5b1fbd92ca244db17a4342
    ```

    > [!NOTE]
    >
    > 自动 DNS 容器发现仅在容器名称唯一时有效。

6. 在 `host2` 上验证 `test-net` 已创建（且 NETWORK ID 与 `host1` 上的 `test-net` 相同）：

    ```console
    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    ...
    uqsof8phj3ak        test-net            overlay             swarm
    ```

7.  在 `host1` 上，从 `alpine1` 的交互终端内 ping `alpine2`：

    ```console
    / # ping -c 2 alpine2
    PING alpine2 (10.0.0.5): 56 data bytes
    64 bytes from 10.0.0.5: seq=0 ttl=64 time=0.600 ms
    64 bytes from 10.0.0.5: seq=1 ttl=64 time=0.555 ms

    --- alpine2 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.555/0.577/0.600 ms
    ```

    两个容器通过连接两台主机的 overlay 网络进行通信。若在 `host2` 上再运行一个“非后台”的 alpine 容器，则可在 `host2` 上 ping `alpine1`（此处添加了[自动清理容器](/reference/cli/docker/container/run/#rm) 的 `--rm` 选项）：

    ```sh
    $ docker run -it --rm --name alpine3 --network test-net alpine
    / # ping -c 2 alpine1
    / # exit
    ```

8.  在 `host1` 上关闭 `alpine1` 会话（容器也将停止）：

    ```console
    / # exit
    ```

9.  清理容器与网络：

    由于各宿主机的 Docker 守护进程彼此独立，且这里是独立容器，你需要在每台主机上分别停止并删除容器。只需在 `host1` 上删除网络即可，因为当你在 `host2` 上停止 `alpine2` 后，`test-net` 会自动消失。

    a.  在 `host2` 上停止 `alpine2`，确认 `test-net` 已移除，然后删除 `alpine2`：

    ```console
    $ docker container stop alpine2
    $ docker network ls
    $ docker container rm alpine2
    ```

    a.  在 `host1` 上删除 `alpine1` 与 `test-net`：

    ```console
    $ docker container rm alpine1
    $ docker network rm test-net
    ```

## 其他网络教程

- [Host 网络教程](/manuals/engine/network/tutorials/host.md)
- [独立网络教程](/manuals/engine/network/tutorials/standalone.md)
- [Macvlan 网络教程](/manuals/engine/network/tutorials/macvlan.md)
