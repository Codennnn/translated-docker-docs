---
description: 扩缩在 swarm 中运行的服务
keywords: tutorial, cluster management, swarm mode, scale, get started
title: 在 swarm 中扩缩服务
weight: 50
notoc: true
---

当你已将[服务部署](deploy-service.md)到 swarm 后，就可以使用 Docker CLI 调整该服务中的容器数量。服务中的运行单元称为任务（task）。

1.  如果尚未操作，请打开终端并通过 ssh 登录到运行管理节点的那台机器。例如，本教程使用名为 `manager1` 的机器。

2.  运行以下命令以更改该服务在 swarm 中的期望状态：

    ```console
    $ docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>
    ```

    例如：

    ```console
    $ docker service scale helloworld=5

    helloworld scaled to 5
    ```

3.  执行 `docker service ps <SERVICE-ID>` 查看更新后的任务列表：

    ```console
    $ docker service ps helloworld

    NAME                                    IMAGE   NODE      DESIRED STATE  CURRENT STATE
    helloworld.1.8p1vev3fq5zm0mi8g0as41w35  alpine  worker2   Running        Running 7 minutes
    helloworld.2.c7a7tcdq5s0uk3qr88mf8xco6  alpine  worker1   Running        Running 24 seconds
    helloworld.3.6crl09vdcalvtfehfh69ogfb1  alpine  worker1   Running        Running 24 seconds
    helloworld.4.auky6trawmdlcne8ad8phb0f1  alpine  manager1  Running        Running 24 seconds
    helloworld.5.ba19kca06l18zujfwxyc5lkyn  alpine  worker2   Running        Running 24 seconds
    ```

    可以看到，swarm 新创建了 4 个任务，使总运行实例数达到 5 个（Alpine Linux）。这些任务分布在集群的 3 个节点上，其中一个运行在 `manager1`。

4.  运行 `docker ps` 查看当前所连节点上运行的容器。下面示例展示了运行在 `manager1` 上的任务：

    ```console
    $ docker ps

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    528d68040f95        alpine:latest       "ping docker.com"   About a minute ago   Up About a minute                       helloworld.4.auky6trawmdlcne8ad8phb0f1
    ```

    如需查看其他节点上的容器，请 ssh 到对应节点后执行 `docker ps`。

## 下一步

至此，本教程中的 `helloworld` 服务已完成演示。接下来，你将删除该服务。

{{< button text="删除该服务" url="delete-service.md" >}}
