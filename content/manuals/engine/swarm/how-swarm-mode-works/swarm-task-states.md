---
title: Swarm 任务状态
description: 了解在 Swarm 中被调度执行的任务及其状态。
keywords: swarm, task, service
aliases:
- /datacenter/ucp/2.2/guides/admin/monitor-and-troubleshoot/troubleshoot-task-state/
---

Docker 允许你创建服务，服务会启动任务（task）。服务描述的是“期望状态”，而任务负责“实际执行”。在 Swarm 节点上，调度流程一般如下：

1.  通过 `docker service create` 创建服务。
2.  请求到达某个 Docker 管理节点。
3.  管理节点将该服务调度到特定节点上运行。
4.  每个服务可以启动多个任务。
5.  每个任务都有生命周期，例如 `NEW`、`PENDING`、`COMPLETE` 等状态。

任务是“一次性执行单元”，运行一次直至完成。某个任务停止后不会再次执行，但可能会有新的任务替代它继续工作。

任务会依次经历一系列状态直至完成或失败。任务以 `NEW` 状态初始化，之后仅向前推进，不会回退。例如，不会从 `COMPLETE` 回到 `RUNNING`。

任务通常按如下顺序流转：

| 任务状态      | 说明                                                                                                        |
| ------------- | ----------------------------------------------------------------------------------------------------------- |
| `NEW`         | 任务已初始化。                                                                                               |
| `PENDING`     | 已为任务分配资源。                                                                                           |
| `ASSIGNED`    | Docker 已将任务分配到节点。                                                                                  |
| `ACCEPTED`    | 工作节点已接受该任务；若拒绝则转为 `REJECTED`。                                                               |
| `READY`       | 工作节点已准备好启动任务。                                                                                   |
| `PREPARING`   | Docker 正在为任务做准备。                                                                                    |
| `STARTING`    | Docker 正在启动任务。                                                                                        |
| `RUNNING`     | 任务正在执行。                                                                                               |
| `COMPLETE`    | 任务正常退出（无错误码）。                                                                                   |
| `FAILED`      | 任务以错误码退出。                                                                                           |
| `SHUTDOWN`    | Docker 请求该任务关闭。                                                                                      |
| `REJECTED`    | 工作节点拒绝了该任务。                                                                                       |
| `ORPHANED`    | 节点长时间不可用导致任务“孤儿化”。                                                                           |
| `REMOVE`      | 任务本身非终止态，但关联服务已被移除或缩容。                                                                 |

## 查看任务状态

运行 `docker service ps <service-name>` 可查看任务状态。`CURRENT STATE` 字段表示任务当前状态以及已持续的时间。

```console
$ docker service ps webserver
ID             NAME              IMAGE    NODE        DESIRED STATE  CURRENT STATE            ERROR                              PORTS
owsz0yp6z375   webserver.1       nginx    UbuntuVM    Running        Running 44 seconds ago
j91iahr8s74p    \_ webserver.1   nginx    UbuntuVM    Shutdown       Failed 50 seconds ago    "No such container: webserver.…"
7dyaszg13mw2    \_ webserver.1   nginx    UbuntuVM    Shutdown       Failed 5 hours ago       "No such container: webserver.…"
```

## 下一步

- [进一步了解 swarm 任务](https://github.com/docker/swarmkit/blob/master/design/task_model.md)
