---
description: 将 swarm 中的节点设为 Drain
keywords: tutorial, cluster management, swarm, service, drain, get started
title: 将 swarm 中的某个节点设为 Drain
weight: 80
notoc: true
---

在本教程的前面步骤中，所有节点的可用性均为 `Active`。swarm 管理节点可以把任务分配给任意 `Active` 节点，因此到目前为止所有节点都可接收任务。

在某些场景（例如计划性维护）下，你需要将某个节点的可用性设置为 `Drain`。`Drain` 会阻止管理节点向该节点分配新任务；同时，管理节点会停止该节点上正在运行的任务，并在处于 `Active` 状态的节点上启动等价副本任务。

> [!IMPORTANT]
>
> 将节点设为 `Drain` 不会移除该节点上的独立容器，例如通过 `docker run`、`docker compose up` 或 Docker Engine API 创建的容器。节点状态（包括 `Drain`）仅影响该节点调度 Swarm 服务工作负载的能力。

1.  如果尚未操作，请打开终端并通过 ssh 登录到运行管理节点的那台机器。例如，本教程使用名为 `manager1` 的机器。

2.  验证所有节点当前均为 `Active`：

    ```console
    $ docker node ls

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    1bcef6utixb0l0ca7gxuivsj0    worker2   Ready   Active
    38ciaotwjuritcdtn9npbnkuz    worker1   Ready   Active
    e216jshn25ckzbvmwlnh5jr3g *  manager1  Ready   Active        Leader
    ```

3.  如果你没有继续运行来自[滚动更新](rolling-update.md) 教程的 `redis` 服务，请现在启动它：

    ```console
    $ docker service create --replicas 3 --name redis --update-delay 10s redis:7.4.0

    c5uo6kdmzpon37mgj9mwglcfw
    ```

4.  运行 `docker service ps redis` 查看管理节点如何将任务分配到不同节点：

    ```console
    $ docker service ps redis

    NAME                               IMAGE        NODE     DESIRED STATE  CURRENT STATE
    redis.1.7q92v0nr1hcgts2amcjyqg3pq  redis:7.4.0  manager1 Running        Running 26 seconds
    redis.2.7h2l8h3q3wqy5f66hlv9ddmi6  redis:7.4.0  worker1  Running        Running 26 seconds
    redis.3.9bg7cezvedmkgg6c8yzvbhwsd  redis:7.4.0  worker2  Running        Running 26 seconds
    ```

    在此示例中，管理节点将 1 个任务分配到每台节点。不同环境中任务的分配可能有所差异。

5.  运行 `docker node update --availability drain <NODE-ID>` 将某个已分配任务的节点设为 `Drain`：

    ```console
    $ docker node update --availability drain worker1

    worker1
    ```

6.  检查该节点以确认其可用性：

    ```console
    $ docker node inspect --pretty worker1

    ID:			38ciaotwjuritcdtn9npbnkuz
    Hostname:		worker1
    Status:
     State:			Ready
     Availability:		Drain
    ...snip...
    ```

    被设置为 `Drain` 的节点，其 `Availability` 字段会显示为 `Drain`。

7.  再次执行 `docker service ps redis`，查看管理节点如何更新 `redis` 服务的任务分配：

    ```console
    $ docker service ps redis

    NAME                                    IMAGE        NODE      DESIRED STATE  CURRENT STATE           ERROR
    redis.1.7q92v0nr1hcgts2amcjyqg3pq       redis:7.4.0  manager1  Running        Running 4 minutes
    redis.2.b4hovzed7id8irg1to42egue8       redis:7.4.0  worker2   Running        Running About a minute
     \_ redis.2.7h2l8h3q3wqy5f66hlv9ddmi6   redis:7.4.0  worker1   Shutdown       Shutdown 2 minutes ago
    redis.3.9bg7cezvedmkgg6c8yzvbhwsd       redis:7.4.0  worker2   Running        Running 4 minutes
    ```

    管理节点通过在 `Drain` 节点上终止任务、并在 `Active` 节点上创建新任务的方式来维持期望状态。

8.  运行 `docker node update --availability active <NODE-ID>` 将被 Drain 的节点恢复为 `Active`：

    ```console
    $ docker node update --availability active worker1

    worker1
    ```

9.  检查该节点以查看更新后的状态：

    ```console
    $ docker node inspect --pretty worker1

    ID:			38ciaotwjuritcdtn9npbnkuz
    Hostname:		worker1
    Status:
     State:			Ready
     Availability:		Active
    ...snip...
    ```

    当你将节点恢复为 `Active` 后，它可以在以下情况下接收新任务：

    * 在服务扩容更新期间
    * 在滚动更新期间
    * 当你将另一台节点设为 `Drain` 时
    * 当其他 `Active` 节点上的任务失败时

## 下一步

接下来，你将学习如何使用 Swarm 模式下的路由网格（routing mesh）。

{{< button text="使用 Swarm 模式的路由网格" url="../ingress.md" >}}
