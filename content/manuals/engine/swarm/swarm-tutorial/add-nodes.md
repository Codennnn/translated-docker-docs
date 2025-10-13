---
description: 向 Swarm 添加节点
keywords: tutorial, cluster management, swarm, get started
title: 向 Swarm 添加节点
weight: 20
notoc: true
---

当你已经通过[创建 swarm](create-swarm.md) 获得了一个包含管理节点的集群后，就可以开始添加工作节点了。

1.  打开终端，使用 ssh 登录到你希望运行工作节点的那台机器。
    本教程将其命名为 `worker1`。

2.  在该机器上执行 `docker swarm init` 输出中给出的加入命令（来自
    「[创建 swarm](create-swarm.md)」步骤），将其作为工作节点加入现有的 swarm：

    ```console
    $ docker swarm join \
      --token  SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
      192.168.99.100:2377

    This node joined a swarm as a worker.
    ```

    如果你手头没有该加入命令，可以在任一管理节点上运行下面的命令，重新获取用于加入为工作节点的命令：

    ```console
    $ docker swarm join-token worker

    To add a worker to this swarm, run the following command:

        docker swarm join \
        --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
        192.168.99.100:2377
    ```

3.  打开终端，使用 ssh 登录到第二台将运行工作节点的机器。
    本教程将其命名为 `worker2`。

4.  在该机器上执行「[创建 swarm](create-swarm.md)」步骤的 `docker swarm init` 输出中给出的加入命令，
    将其作为第二个工作节点加入现有的 swarm：

    ```console
    $ docker swarm join \
      --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
      192.168.99.100:2377

    This node joined a swarm as a worker.
    ```

5.  在运行管理节点的那台机器上打开终端，执行 `docker node ls` 查看工作节点是否加入：

    ```console
    $ docker node ls
    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    03g1y59jwfg7cf99w4lt0f662    worker2   Ready   Active
    9j68exjopxe7wfl6yuxml7a7j    worker1   Ready   Active
    dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader
    ```

    `MANAGER STATUS` 列用于标识 swarm 中的管理节点。`worker1` 与 `worker2` 在该列为空，说明它们是工作节点。

    `docker node ls` 等 Swarm 管理命令只能在管理节点上运行。


## 下一步

现在你的 swarm 由 1 个管理节点与 2 个工作节点组成。接下来，你将部署一个服务。

{{< button text="部署服务" url="deploy-service.md" >}}
