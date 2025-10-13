---
description: 初始化一个 swarm
keywords: tutorial, cluster management, swarm mode, get started, docker engine
title: 创建一个 swarm
weight: 10
notoc: true
---

完成[教程环境准备](index.md)后，你就可以创建一个 swarm。请确保宿主机上的 Docker Engine 守护进程已启动。

1.  打开终端，使用 ssh 登录到你想要运行管理节点的那台机器。本教程中该机器命名为 `manager1`。

2.  运行以下命令创建一个新的 swarm：

    ```console
    $ docker swarm init --advertise-addr <MANAGER-IP>
    ```

    在本教程中，下面的命令会在 `manager1` 这台机器上创建一个 swarm：

    ```console
    $ docker swarm init --advertise-addr 192.168.99.100
    Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join \
        --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
        192.168.99.100:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

    `--advertise-addr` 用于将管理节点对外通告为 `192.168.99.100`。swarm 中的其他节点必须能够通过该 IP 访问该管理节点。

    命令输出中包含用于将新节点加入到 swarm 的相关命令。节点以管理节点或工作节点加入，取决于 `--token` 的取值。

3.  执行 `docker info` 查看当前 swarm 状态：

    ```console
    $ docker info

    Containers: 2
    Running: 0
    Paused: 0
    Stopped: 2
      ...snip...
    Swarm: active
      NodeID: dxn1zf6l61qsb1josjja83ngz
      Is Manager: true
      Managers: 1
      Nodes: 1
      ...snip...
    ```

4.  执行 `docker node ls` 查看节点信息：

    ```console
    $ docker node ls

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader

    ```

    节点 ID 旁的 `*` 表示你当前连接的是此节点。

    Docker Engine 的 Swarm 模式会自动使用机器的主机名作为节点名称。关于输出中的其他列，本教程后续步骤会进行介绍。

## 下一步

接下来，你将向集群中再添加两个节点。

{{< button text="添加两个节点" url="add-nodes.md" >}}
