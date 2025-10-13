---
description: 管理节点运维指南
keywords: docker, 容器, swarm, 管理节点, raft
title: 运维与维护由多个 Docker Engine 组成的 swarm
aliases:
- /engine/swarm/manager-administration-guide/
---

当你运行一个由多个 Docker Engine 组成的 swarm 时，管理节点（manager）是负责管理 swarm 与保存其状态的关键组件。要正确部署并维护 swarm，理解管理节点的一些关键特性非常重要。

参见[节点如何工作](how-swarm-mode-works/nodes.md)，快速了解 Docker 的 Swarm 模式，以及管理节点与工作节点（worker）的区别。

## 在 swarm 中运维管理节点

swarm 的管理节点使用 [Raft 一致性算法](raft.md) 来管理 swarm 状态。为了运维一个 swarm，你只需要理解 Raft 的一些基本概念即可。

管理节点的数量没有硬性上限。选择多少个管理节点，是性能与容错之间的权衡。向 swarm 增加管理节点可以提升容错能力；但同时，由于提交更新需要更多节点确认，写入性能会下降，网络往返也会增加。

Raft 要求多数管理节点（即“仲裁”，quorum）对更新提案达成一致，例如添加或移除节点。成员关系变更与状态复制遵循相同的约束。

### 维持管理节点的仲裁（quorum） {#maintain-the-quorum-of-managers}

如果 swarm 丧失了管理节点的仲裁，就无法执行管理任务。若你的 swarm 有多个管理节点，请务必保持数量大于 2。要维持仲裁，必须有超过半数的管理节点可用。建议使用奇数个管理节点，因为增加到下一个偶数并不会更容易维持仲裁。例如：不论是 3 个还是 4 个管理节点，你最多都只能失去 1 个仍维持仲裁；5 个或 6 个管理节点，最多只能失去 2 个。

即使 swarm 丧失了管理节点的仲裁，现有工作节点上的任务仍会继续运行。但你将无法添加、更新或移除节点，也无法启动、停止、迁移或更新新的或现有的任务。

如果确实丧失了仲裁，请参见[从丧失仲裁中恢复](#recover-from-losing-the-quorum)获取排错步骤。

## 将管理节点配置为使用静态 IP 进行对外通告 {#configure-the-manager-to-advertise-on-a-static-ip-address}

在初始化一个 swarm 时，必须通过 `--advertise-addr` 标志将你的地址通告给 swarm 中的其他管理节点。更多信息参见[在 Swarm 模式下运行 Docker Engine](swarm-mode.md#configure-the-advertise-address)。由于管理节点旨在作为基础设施中的稳定组件，建议对对外通告地址使用“固定 IP 地址”，以避免机器重启后导致 swarm 不稳定。

如果整个 swarm 发生重启，并且每个管理节点随后都获得了新的 IP 地址，那么任何节点都无法联系到已有的管理节点。此时，swarm 会停滞在互相尝试使用旧 IP 通信的状态。

对工作节点而言，使用动态 IP 是可以的。

## 为容错添加管理节点 {#add-manager-nodes-for-fault-tolerance}

建议在 swarm 中维持奇数个管理节点，以应对管理节点故障。使用奇数个管理节点有助于在网络发生分区时仍更有可能保持仲裁可用，从而继续处理请求。若出现超过两个网络分区，仍无法保证维持仲裁。

| Swarm 规模 | 多数（仲裁） | 容错能力 |
|:---------:|:------------:|:--------:|
|     1     |      1       |    0     |
|     2     |      2       |    0     |
|   **3**   |      2       |  **1**   |
|     4     |      3       |    1     |
|   **5**   |      3       |  **2**   |
|     6     |      4       |    2     |
|   **7**   |      4       |  **3**   |
|     8     |      5       |    3     |
|   **9**   |      5       |  **4**   |

例如，在一个包含 5 个节点的 swarm 中，若失去 3 个节点，你将不再拥有仲裁。因此，在恢复至少一个不可用的管理节点，或通过灾难恢复命令恢复 swarm 之前，你都无法添加或移除节点。参见[灾难恢复](#recover-from-disaster)。

虽然可以把 swarm 缩容到仅剩一个管理节点，但无法降级最后一个管理节点。此举是为了保证你仍能访问 swarm，且 swarm 仍可处理请求。缩容为单一管理节点是不安全的做法，不建议使用。如果在降级过程中最后一个节点意外离开了 swarm，那么在你重启该节点或使用 `--force-new-cluster` 重新启动之前，swarm 将不可用。

你可以使用 `docker swarm` 与 `docker node` 子命令来管理 swarm 成员。如何添加工作节点并将其晋升为管理节点，参见[向 swarm 添加节点](join-nodes.md)。

### 分布管理节点 {#distribute-manager-nodes}

除维持奇数个管理节点外，还需在放置管理节点时关注数据中心拓扑。为了获得最佳容错能力，建议将管理节点分散到至少 3 个可用区中，以应对整组机器的故障或常见维护场景。如果任一可用区发生故障，swarm 仍应维持足够的管理节点仲裁，以继续处理请求并重新均衡负载。

| 管理节点数量 |  分布建议（覆盖 3 个可用区） |
|:------------:|:----------------------------:|
|      3       |             1-1-1            |
|      5       |             2-2-1            |
|      7       |             3-2-2            |
|      9       |             3-3-3            |

### 仅运行管理节点（不承载任务） {#run-manager-only-nodes}

默认情况下，管理节点也作为工作节点使用，这意味着调度器可以在管理节点上分配任务。对于小规模或非关键性的 swarm，只要在调度服务时合理设置 CPU 与内存等资源约束，把任务分配给管理节点的风险相对较低。

但由于管理节点依赖 Raft 共识以一致地复制数据，它们对资源饥饿较为敏感。你应当让管理节点远离可能阻塞 swarm 心跳或领导者选举等关键操作的进程。

为避免干扰管理节点的职能，可以将管理节点设置为“drain”状态，使其不再作为工作节点接受任务：

```console
$ docker node update --availability drain <NODE>
```

当你对节点执行 drain 后，调度器会把该节点上正在运行的任务重新分配到 swarm 中其他可用的工作节点上；同时也会阻止调度器再向该节点分配新任务。

## 通过添加工作节点实现负载均衡 {#add-worker-nodes-for-load-balancing}

[向 swarm 添加节点](join-nodes.md) 以均衡集群负载。只要工作节点满足服务对资源的要求，复制型服务任务会在 swarm 中随着时间尽可能均匀地分布。如果你限制某个服务只能运行在特定类型的节点上（例如指定 CPU 数量或内存容量），请记住不满足这些要求的工作节点无法承载这些任务。

## 监控 swarm 健康状况 {#monitor-swarm-health}

你可以通过 `/nodes` HTTP 端点以 JSON 格式查询 docker 的 `nodes` API 来监控管理节点的健康状况。更多信息参见[Nodes API 文档](/reference/api/engine/version/v1.25/#tag/Node)。

在命令行中，使用 `docker node inspect <id-node>` 查询节点信息。例如，查询该节点作为管理节点的可达性：

```console
$ docker node inspect manager1 --format "{{ .ManagerStatus.Reachability }}"
reachable
```

查询该节点作为可接受任务的工作节点时的状态：

```console
$ docker node inspect manager1 --format "{{ .Status.State }}"
ready
```

从以上命令可以看到，`manager1` 作为管理节点的状态为 `reachable`，作为工作节点的状态为 `ready`。

若健康状态为 `unreachable`，表示这个管理节点对其他管理节点不可达。此时需要采取行动恢复该管理节点：

- 重启 Docker 守护进程，观察该管理节点是否恢复为可达。
- 重启机器。
- 如果重启守护进程与重启机器都无效，应添加新的管理节点或将某个工作节点晋升为管理节点。同时需要使用 `docker node demote <NODE>` 与 `docker node rm <id-node>` 将故障节点从管理节点集合中干净移除。

你也可以在任一管理节点上使用 `docker node ls` 获取 swarm 健康概览：

```console
$ docker node ls
ID                           HOSTNAME  MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS
1mhtdwhvsgr3c26xxbnzdc3yp    node05    Accepted    Ready   Active
516pacagkqp2xc3fk9t1dhjor    node02    Accepted    Ready   Active        Reachable
9ifojw8of78kkusuc4a6c23fx *  node01    Accepted    Ready   Active        Leader
ax11wdpwrrb6db3mfjydscgk7    node04    Accepted    Ready   Active
bb1nrq2cswhtbg4mrsqnlx1ck    node03    Accepted    Ready   Active        Reachable
di9wxgz8dtuh9d2hn089ecqkf    node06    Accepted    Ready   Active
```

## 排查管理节点故障 {#troubleshoot-a-manager-node}

不要通过拷贝其他节点的 `raft` 目录来重建或重启某个管理节点。数据目录与节点 ID 一一对应。一个节点只能用某个节点 ID 加入 swarm 一次；节点 ID 必须在全局范围内唯一。

要将某个管理节点干净地重新加入集群：

1. 使用 `docker node demote <NODE>` 将其降级为工作节点。
2. 使用 `docker node rm <NODE>` 将该节点从 swarm 中移除。
3. 使用 `docker swarm join` 以全新状态将该节点重新加入 swarm。

关于将管理节点加入 swarm 的更多信息，参见[向 swarm 添加节点](join-nodes.md)。

## 强制移除节点 {#forcibly-remove-a-node}

多数情况下，应先关闭节点，再使用 `docker node rm` 将其从 swarm 中移除。如果某个节点不可达、无响应或已被入侵，可以使用 `--force` 标志在未关闭的情况下强制移除。例如，若 `node9` 被入侵：

```console
$ docker node rm node9

Error response from daemon: rpc error: code = 9 desc = node node9 is not down and can't be removed

$ docker node rm --force node9

Node node9 removed from swarm
```

在强制移除管理节点之前，必须先将其降级为工作节点。请确保在降级或移除管理节点后，集群中仍保持奇数个管理节点。

## 备份 swarm {#back-up-the-swarm}

Docker 管理节点将 swarm 状态与管理日志存储在 `/var/lib/docker/swarm/` 目录中。该数据包含用于加密 Raft 日志的密钥。没有这些密钥，无法恢复 swarm。

你可以在任一管理节点上执行备份。按如下步骤进行：

1.  如果 swarm 启用了自动锁（auto-lock），恢复时需要解锁密钥。必要时先获取解锁密钥并妥善保存。如不确定，请阅读[锁定 swarm 以保护其加密密钥](swarm_manager_locking.md)。

2.  在执行备份前，先停止该管理节点上的 Docker，以避免备份过程中发生数据变化。虽然也可以在管理节点运行时进行“热备份”，但不推荐这么做，恢复结果的可预期性也较差。在管理节点停止期间，其他节点仍会继续产生不包含在本次备份中的 swarm 数据。

    > [!NOTE]
    > 
    > 请务必维持管理节点的仲裁。在某个管理节点停止期间，如果再失去更多节点，swarm 更容易丧失仲裁。管理节点数量是一种权衡。如果你会定期下线管理节点做备份，考虑运行 5 个管理节点的 swarm，这样即便在备份期间再失去 1 个管理节点，也不至于影响服务。

3.  备份整个 `/var/lib/docker/swarm` 目录。

4.  重新启动该管理节点。

恢复步骤参见[从备份恢复](#restore-from-a-backup)。

## 灾难恢复 {#recover-from-disaster}

### 从备份恢复 {#restore-from-a-backup}

按[备份 swarm](#back-up-the-swarm) 的说明完成备份后，使用以下步骤将数据恢复到新的 swarm。

1.  关闭用于恢复的目标主机上的 Docker。

2.  清空新 swarm 上的 `/var/lib/docker/swarm` 目录内容。

3.  用备份内容恢复 `/var/lib/docker/swarm` 目录。

    > [!NOTE]
    > 
    > 新节点使用与旧节点相同的磁盘存储加密密钥。目前无法更改磁盘存储加密密钥。
    >
    > 若 swarm 启用了自动锁，解锁密钥也与旧 swarm 相同，恢复时同样需要该解锁密钥。

4.  在新节点上启动 Docker。必要时先解锁 swarm。使用以下命令重新初始化 swarm，使该节点不再尝试连接旧 swarm 中的其他节点（这些节点很可能已经不存在）：

    ```console
    $ docker swarm init --force-new-cluster
    ```

5.  验证 swarm 状态是否符合预期。验证可以是应用层面的自测，也可以是简单检查 `docker service ls` 的输出，确认所有预期服务均已存在。

6.  如果你使用自动锁，请[轮换解锁密钥](swarm_manager_locking.md#rotate-the-unlock-key)。

7.  添加管理节点与工作节点，使新的 swarm 达到可用容量。

8.  在新 swarm 上恢复你之前的备份策略。

### 从丧失仲裁中恢复 {#recover-from-losing-the-quorum}

swarm 对故障具有一定的韧性，能够从任意数量的临时节点故障（例如机器重启、崩溃并自动重启）或其他暂时性错误中恢复。然而，一旦丧失仲裁，swarm 无法自动恢复。现有工作节点上的任务会继续运行，但无法执行管理操作，包括对服务的扩缩容或更新、以及在 swarm 中加入或移除节点。最佳的恢复方式是让缺失的管理节点重新上线。如果做不到，请继续阅读以下选项。

在一个包含 `N` 个管理节点的 swarm 中，必须始终有“仲裁”（多数）管理节点可用。例如，在 5 个管理节点的 swarm 中，至少应有 3 个处于可用并且彼此连通的状态。换言之，swarm 最多可以承受 `(N-1)/2` 个永久性故障，超过该数量后，与 swarm 管理相关的请求将无法被处理。这类故障包括数据损坏或硬件故障。

当你丧失管理节点仲裁后，将无法对 swarm 进行管理。如果在这种情况下尝试执行任何管理操作，会出现如下错误：

```text
Error response from daemon: rpc error: code = 4 desc = context deadline exceeded
```

恢复仲裁的最佳方式是让故障节点重新上线。如果无法做到，只能在某个管理节点上使用 `--force-new-cluster` 进行恢复。该操作会移除除当前节点外的所有管理节点。由于此时仅剩一个管理节点，因此自动满足仲裁。随后将其他节点晋升为管理节点，直到达到你期望的管理节点数量。

在要进行恢复的节点上，运行：

```console
$ docker swarm init --force-new-cluster --advertise-addr node01:2377
```

当你使用 `--force-new-cluster` 运行 `docker swarm init` 时，执行该命令的 Docker Engine 会成为一个“单管理节点 swarm”的管理节点，具备管理与运行服务的能力。该管理节点保留了此前关于服务与任务的全部信息；工作节点仍属于该 swarm，服务也仍在运行。你需要添加或重新添加管理节点，以恢复之前的任务分布，并确保拥有足够的管理节点来维持高可用并避免再次丧失仲裁。

## 强制让 swarm 重新均衡 {#force-the-swarm-to-rebalance}

通常不需要强制 swarm 重新均衡任务。当向 swarm 新增节点，或某个节点在一段不可用时间之后重新连回 swarm 时，swarm 不会自动把负载迁移到该空闲节点。这是刻意的设计：如果 swarm 为了“均衡”而定期迁移任务，使用这些任务的客户端会受到干扰。目标是在尽量不干扰正在运行的服务的前提下，实现最终的负载均衡。当有新任务启动，或某个承载任务的节点不可用时，这些任务会被分配给负载较轻的节点。目标是“最终趋于均衡”，同时把对终端用户的干扰降到最低。

你可以在 `docker service update` 命令中使用 `--force` 或 `-f` 标志，强制服务将其任务在可用的工作节点之间重新分布。这样会导致服务任务重启，可能会对客户端应用造成影响。若已配置，该服务会使用[滚动更新](swarm-tutorial/rolling-update.md)。

如果你使用的是较早版本，并且希望在不介意中断正在运行任务的前提下实现更均匀的负载分布，可以通过临时放大服务规模来强制重新均衡。使用 `docker service inspect --pretty <servicename>` 查看服务的规模配置。当你使用 `docker service scale` 时，会优先把新负载分配给当前任务数最少的节点。你的 swarm 中可能存在多个负载不足的节点；为了在所有节点上达成你想要的均衡程度，可能需要多次、小幅度地放大服务规模。

当负载分布达到你的预期后，可以把服务规模缩回原值。你可以使用 `docker service ps` 来评估当前服务在各节点上的负载分布情况。

另请参见：[`docker service scale`](/reference/cli/docker/service/scale.md) 与 [`docker service ps`](/reference/cli/docker/service/ps.md)。
