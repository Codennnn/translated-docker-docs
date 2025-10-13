---
description: 在 Swarm 中管理现有节点
keywords: guide, swarm mode, node
title: 管理 Swarm 中的节点
---

在 Swarm 的日常运维中，你可能需要进行以下操作：

* [查看 Swarm 中的节点列表](#list-nodes)
* [检查单个节点详情](#inspect-an-individual-node)
* [更新节点属性](#update-a-node)
* [使节点退出 Swarm](#leave-the-swarm)

## 查看节点列表

在管理节点上运行 `docker node ls` 可查看 Swarm 中所有节点：

```console
$ docker node ls

ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
46aqrk4e473hjbt745z53cr3t    node-5    Ready   Active        Reachable
61pi3d91s0w3b90ijw3deeb2q    node-4    Ready   Active        Reachable
a5b2m3oghd48m8eu391pefq5u    node-3    Ready   Active
e7p8btxeu3ioshyuj6lxiv6g0    node-2    Ready   Active
ehkv3bcimagdese79dn78otj5 *  node-1    Ready   Active        Leader
```

`AVAILABILITY` 列表示调度器是否可以向该节点分配任务：

* `Active`：调度器可以向该节点分配任务。
* `Pause`：调度器不再向该节点分配新任务，但已有任务继续运行。
* `Drain`：调度器不再向该节点分配新任务，并会关闭该节点上的现有任务，然后将它们调度到其他可用节点。

`MANAGER STATUS` 列表示节点在 Raft 共识中的角色：

* 为空：表示该节点为 worker，不参与 Swarm 管理。
* `Leader`：该节点为主管理节点，负责做出集群的编排与管理决策。
* `Reachable`：该节点为参与 Raft 共识投票的管理节点。如果 `Leader` 不可用，该节点有资格参与新 Leader 的选举。
* `Unavailable`：该节点为管理节点，但无法与其他管理节点通信。此时应添加新的管理节点，或将某个 worker 提升为管理节点。

更多 Swarm 管理内容，请参阅《[Swarm 管理指南](admin_guide.md)》。

## 检查单个节点

在管理节点上运行 `docker node inspect <NODE-ID>` 可查看某个节点的详细信息。默认输出为 JSON；你也可以添加 `--pretty` 标志，以更易读的格式显示结果。例如：

```console
$ docker node inspect self --pretty

ID:                     ehkv3bcimagdese79dn78otj5
Hostname:               node-1
Joined at:              2016-06-16 22:52:44.9910662 +0000 utc
Status:
 State:                 Ready
 Availability:          Active
Manager Status:
 Address:               172.17.0.2:2377
 Raft Status:           Reachable
 Leader:                Yes
Platform:
 Operating System:      linux
 Architecture:          x86_64
Resources:
 CPUs:                  2
 Memory:                1.954 GiB
Plugins:
  Network:              overlay, host, bridge, overlay, null
  Volume:               local
Engine Version:         1.12.0-dev
```

## 更新节点

你可以通过修改节点属性来：

* [更改节点可用性](#change-node-availability)
* [添加或删除标签元数据](#add-or-remove-label-metadata)
* [变更节点角色](#promote-or-demote-a-node)

### 更改节点可用性

通过调整节点的可用性，你可以：

* 将管理节点设置为 `Drain`，使其只执行集群管理，不参与任务分配。
* 将某节点设置为 `Drain` 以便进行维护停机。
* 将节点设置为 `Pause`，阻止其接收新任务。
* 恢复处于不可用或暂停状态节点的可用性。

例如，将某个管理节点设置为 `Drain`：

```console
$ docker node update --availability drain node-1

node-1
```

不同可用性状态的说明见「[查看节点列表](#list-nodes)」。

### 添加或删除标签元数据

节点标签是一种灵活的组织方式。你也可以在创建服务时使用标签作为约束，限制调度器将该服务的任务分配到哪些节点。

在管理节点上运行 `docker node update --label-add` 为节点添加标签。`--label-add` 可以接受 `<key>` 或 `<key>=<value>` 形式。

为每个要添加的标签分别传入一次 `--label-add`：

```console
$ docker node update --label-add foo --label-add bar=baz node-1

node-1
```

通过 `docker node update` 设置的标签仅用于 Swarm 内的“节点实体”。不要将其与 Docker 守护进程（`dockerd`）的引擎标签混淆，后者见《[管理资源：标签](/manuals/engine/manage-resources/labels.md)》。

因此，你可以使用节点标签将关键任务限定在满足特定要求的节点上。例如，仅在满足 [PCI-SS 合规性](https://www.pcisecuritystandards.org/) 的机器上运行某些特殊工作负载。

即便某个 worker 被攻破，也无法篡改这些节点标签，从而难以影响这类敏感工作负载。

另一方面，引擎标签依然有其价值：对于不影响容器安全编排的功能，分散设置会更灵活。例如，可以为某个引擎设置一个标签，标识其具备某种磁盘设备类型——这与安全无直接关系，但此类标签更容易被编排器“信任”。

关于服务约束的更多用法，参阅 `docker service create` 的《[CLI 参考](/reference/cli/docker/service/create.md)》。

### 提升或降级节点角色

你可以将 worker 提升为 manager。当某个管理节点不可用，或计划让其下线维护时，这很有用。相应地，你也可以将 manager 降级为 worker。

> [!NOTE]
>
> 无论因何进行节点的提升或降级，都必须始终保证集群中管理节点达到法定数量（quorum）。更多信息参阅《[Swarm 管理指南](admin_guide.md)》。

要提升一个或多个节点，请在管理节点上运行 `docker node promote`：

```console
$ docker node promote node-3 node-2

Node node-3 promoted to a manager in the swarm.
Node node-2 promoted to a manager in the swarm.
```

要降级一个或多个节点，请在管理节点上运行 `docker node demote`：

```console
$ docker node demote node-3 node-2

Manager node-3 demoted in the swarm.
Manager node-2 demoted in the swarm.
```

`docker node promote` 与 `docker node demote` 是便捷命令，分别等价于 `docker node update --role manager` 与 `docker node update --role worker`。

## 在 Swarm 节点上安装插件

如果你的 Swarm 服务依赖一个或多个[插件](/engine/extend/plugin_api/)，那么这些插件必须存在于所有可能被调度到的节点上。你可以在每台节点上手动安装，或编写脚本自动安装。也可以通过 Docker API 类似“全局服务”的方式进行部署，只需在任务模板中指定 `PluginSpec`（而非 `ContainerSpec`）。

> [!NOTE]
>
> 目前无法通过 Docker CLI 或 Docker Compose 将插件直接部署到 Swarm；此外，也无法从私有仓库安装插件。

[`PluginSpec`](/engine/extend/plugin_api/#json-specification) 由插件开发者定义。若要将插件添加到所有 Docker 节点，可调用 [`service/create`](/reference/api/engine/v1.31/#operation/ServiceCreate) API，并在 `TaskTemplate` 中传入 `PluginSpec` 的 JSON。

## 退出 Swarm

在目标节点上运行 `docker swarm leave`，即可将其从 Swarm 中移除。

例如，在某个 worker 节点上执行退出：

```console
$ docker swarm leave

Node left the swarm.
```

节点退出后，Docker Engine 将停止运行 Swarm 模式，编排器也不会再向该节点调度任务。

如果该节点是管理节点，你会看到关于维持法定数量的警告。可通过 `--force` 强制执行。但如果最后一个管理节点也离开，整个集群将不可用，你需要执行灾难恢复。

关于维持法定数量与灾难恢复，请参阅《[Swarm 管理指南](admin_guide.md)》。

节点退出后，可在管理节点上运行 `docker node rm` 将其从节点列表中删除。

For instance:

```console
$ docker node rm node-2
```

## 延伸阅读

* [Swarm 管理指南](admin_guide.md)
* [Docker Engine 命令行参考](/reference/cli/docker/)
* [Swarm 模式入门教程](swarm-tutorial/_index.md)
