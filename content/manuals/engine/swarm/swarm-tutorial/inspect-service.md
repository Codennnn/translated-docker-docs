---
description: 查看服务详情
keywords: tutorial, cluster management, swarm mode, get started
title: 在 swarm 上检查服务
weight: 40
notoc: true
---

当你已将[服务部署](deploy-service.md)到 swarm 后，可以使用 Docker CLI 查看该服务在集群中的详细信息。

1.  如果尚未操作，请打开终端并通过 ssh 登录到运行管理节点的那台机器。例如，本教程使用名为 `manager1` 的机器。

2.  运行 `docker service inspect --pretty <SERVICE-ID>` 以更便于阅读的格式展示服务详情。

    查看 `helloworld` 服务的详情：

    ```console
    [manager1]$ docker service inspect --pretty helloworld

    ID:		9uk4639qpg7npwf3fn2aasksr
    Name:		helloworld
    Service Mode:	REPLICATED
     Replicas:		1
    Placement:
    UpdateConfig:
     Parallelism:	1
    ContainerSpec:
     Image:		alpine
     Args:	ping docker.com
    Resources:
    Endpoint Mode:  vip
    ```

    > [!TIP]
    >
    > 若要以 JSON 格式返回服务详情，请在运行相同命令时不加 `--pretty` 标志。

    ```console
    [manager1]$ docker service inspect helloworld
    [
    {
        "ID": "9uk4639qpg7npwf3fn2aasksr",
        "Version": {
            "Index": 418
        },
        "CreatedAt": "2016-06-16T21:57:11.622222327Z",
        "UpdatedAt": "2016-06-16T21:57:11.622222327Z",
        "Spec": {
            "Name": "helloworld",
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "alpine",
                    "Args": [
                        "ping",
                        "docker.com"
                    ]
                },
                "Resources": {
                    "Limits": {},
                    "Reservations": {}
                },
                "RestartPolicy": {
                    "Condition": "any",
                    "MaxAttempts": 0
                },
                "Placement": {}
            },
            "Mode": {
                "Replicated": {
                    "Replicas": 1
                }
            },
            "UpdateConfig": {
                "Parallelism": 1
            },
            "EndpointSpec": {
                "Mode": "vip"
            }
        },
        "Endpoint": {
            "Spec": {}
        }
    }
    ]
    ```

3.  运行 `docker service ps <SERVICE-ID>` 查看服务运行在哪些节点上：

    ```console
    [manager1]$ docker service ps helloworld

    NAME                                    IMAGE   NODE     DESIRED STATE  CURRENT STATE           ERROR               PORTS
    helloworld.1.8p1vev3fq5zm0mi8g0as41w35  alpine  worker2  Running        Running 3 minutes
    ```

    在这个示例中，`helloworld` 服务的 1 个实例运行在 `worker2` 节点上。你也可能看到该服务运行在管理节点上：默认情况下，swarm 中的管理节点与工作节点一样可以执行任务。

    Swarm 还会展示服务任务的 `DESIRED STATE` 与 `CURRENT STATE`，以便你判断任务是否按服务定义运行。

4.  在任务所在节点上运行 `docker ps`，查看承载该任务的容器详情。

    > [!TIP]
    >
    > 如果 `helloworld` 运行在管理节点之外的其他节点上，需要先 ssh 到该节点再执行命令。

    ```console
    [worker2]$ docker ps

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    e609dde94e47        alpine:latest       "ping docker.com"   3 minutes ago       Up 3 minutes                            helloworld.1.8p1vev3fq5zm0mi8g0as41w35
    ```

## 下一步

接下来，你将修改该服务的规模（副本数）。

{{< button text="修改规模" url="scale-service.md" >}}
