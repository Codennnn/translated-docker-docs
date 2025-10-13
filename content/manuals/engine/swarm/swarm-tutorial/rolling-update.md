---
description: 对 swarm 上的服务应用滚动更新
keywords: tutorial, cluster management, swarm, service, rolling-update
title: 为服务应用滚动更新
weight: 70
notoc: true
---

在本教程的前一步中，你已经[扩缩](scale-service.md)了某个服务的实例数。本节将基于 Redis 7.4.0 镜像标签部署一个服务，然后通过滚动更新将其升级为 Redis 7.4.1 镜像。

1.  如果尚未操作，请打开终端并通过 ssh 登录到运行管理节点的那台机器。例如，本教程使用名为 `manager1` 的机器。

2.  将指定 Redis 标签部署到 swarm，并配置 10 秒的更新延迟。注意：下面示例使用较早的 Redis 标签：

    ```console
    $ docker service create \
      --replicas 3 \
      --name redis \
      --update-delay 10s \
      redis:7.4.0

    0u6a4s31ybk7yw2wyvtikmu50
    ```

    你可以在创建服务时配置滚动更新策略。

    `--update-delay` 用于配置任务（或任务集合）之间的更新时间间隔。时间 `T` 可以用秒 `Ts`、分 `Tm`、时 `Th` 的组合来表示，例如 `10m30s` 表示 10 分 30 秒。

    默认情况下，调度器一次更新一个任务。你可以通过 `--update-parallelism` 配置调度器可同时更新的任务最大数。

    默认行为是：当某个任务更新后进入 `RUNNING` 状态后，调度器再更新下一个，直到全部更新完成。若在更新过程中任一任务进入 `FAILED`，调度器会暂停更新。你可以在 `docker service create` 或 `docker service update` 中通过 `--update-failure-action` 控制此行为。

3.  检查 `redis` 服务：

    ```console
    $ docker service inspect --pretty redis

    ID:             0u6a4s31ybk7yw2wyvtikmu50
    Name:           redis
    Service Mode:   Replicated
     Replicas:      3
    Placement:
     Strategy:	    Spread
    UpdateConfig:
     Parallelism:   1
     Delay:         10s
    ContainerSpec:
     Image:         redis:7.4.0
    Resources:
    Endpoint Mode:  vip
    ```

4.  现在更新 `redis` 的镜像。swarm 管理节点会根据 `UpdateConfig` 策略将更新应用到各节点：

    ```console
    $ docker service update --image redis:7.4.1 redis
    redis
    ```

    调度器默认按如下流程执行滚动更新：

    * 停止第一个任务
    * 为已停止的任务执行更新
    * 启动更新后的任务容器
    * 若该任务更新后进入 `RUNNING`，等待配置的延迟时间后再开始下一个任务
    * 若在更新过程中任一任务进入 `FAILED`，则暂停更新

5.  运行 `docker service inspect --pretty redis`，查看期望状态中的新镜像：

    ```console
    $ docker service inspect --pretty redis

    ID:             0u6a4s31ybk7yw2wyvtikmu50
    Name:           redis
    Service Mode:   Replicated
     Replicas:      3
    Placement:
     Strategy:	    Spread
    UpdateConfig:
     Parallelism:   1
     Delay:         10s
    ContainerSpec:
     Image:         redis:7.4.1
    Resources:
    Endpoint Mode:  vip
    ```

    若更新因失败而被暂停，`service inspect` 的输出会显示相关信息：

    ```console
    $ docker service inspect --pretty redis

    ID:             0u6a4s31ybk7yw2wyvtikmu50
    Name:           redis
    ...snip...
    Update status:
     State:      paused
     Started:    11 seconds ago
     Message:    update paused due to failure or early termination of task 9p7ith557h8ndf0ui9s0q951b
    ...snip...
    ```

    若要重启暂停的更新，运行 `docker service update <SERVICE-ID>`。例如：

    ```console
    $ docker service update redis
    ```

    为避免重复的更新失败，你可能需要在 `docker service update` 中追加参数重新配置该服务。

6.  运行 `docker service ps <SERVICE-ID>` 观察滚动更新过程：

    ```console
    $ docker service ps redis

    NAME                                   IMAGE        NODE       DESIRED STATE  CURRENT STATE            ERROR
    redis.1.dos1zffgeofhagnve8w864fco      redis:7.4.1  worker1    Running        Running 37 seconds
     \_ redis.1.88rdo6pa52ki8oqx6dogf04fh  redis:7.4.0  worker2    Shutdown       Shutdown 56 seconds ago
    redis.2.9l3i4j85517skba5o7tn5m8g0      redis:7.4.1  worker2    Running        Running About a minute
     \_ redis.2.66k185wilg8ele7ntu8f6nj6i  redis:7.4.0  worker1    Shutdown       Shutdown 2 minutes ago
    redis.3.egiuiqpzrdbxks3wxgn8qib1g      redis:7.4.1  worker1    Running        Running 48 seconds
     \_ redis.3.ctzktfddb2tepkr45qcmqln04  redis:7.4.0  mmanager1  Shutdown       Shutdown 2 minutes ago
    ```

    在 Swarm 更新完全部任务前，你会看到一部分任务运行 `redis:7.4.0`，另一部分运行 `redis:7.4.1`。上面的输出展示了滚动更新完成后的状态。

## 下一步

接下来，你将学习如何将 swarm 中的某个节点设为 Drain。

{{< button text="将节点设为 Drain" url="drain-node.md" >}}
