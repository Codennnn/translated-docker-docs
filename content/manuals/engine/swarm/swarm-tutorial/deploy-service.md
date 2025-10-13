---
description: 将服务部署到 swarm
keywords: tutorial, cluster management, swarm mode, get started
title: 向 swarm 部署服务
weight: 30
notoc: true
---

完成[创建 swarm](create-swarm.md) 后，你可以将服务部署到该集群中。本教程中你还[添加了工作节点](add-nodes.md)，但这并不是部署服务的前置条件。

1.  打开终端，使用 ssh 登录到运行管理节点的那台机器。例如，本教程使用名为 `manager1` 的机器。

2.  运行以下命令：

    ```console
    $ docker service create --replicas 1 --name helloworld alpine ping docker.com

    9uk4639qpg7npwf3fn2aasksr
    ```

    * `docker service create`：创建服务。
    * `--name`：将服务命名为 `helloworld`。
    * `--replicas`：声明期望状态为 1 个运行中的实例。
    * `alpine ping docker.com`：定义该服务为一个 Alpine Linux 容器，并在容器内执行 `ping docker.com`。

3.  执行 `docker service ls` 查看正在运行的服务列表：

    ```console
    $ docker service ls

    ID            NAME        SCALE  IMAGE   COMMAND
    9uk4639qpg7n  helloworld  1/1    alpine  ping docker.com
    ```

## 下一步

现在可以继续查看服务详情。

{{< button text="检查服务" url="inspect-service.md" >}}
