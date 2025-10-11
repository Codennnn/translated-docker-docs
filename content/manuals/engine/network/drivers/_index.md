---
title: 网络驱动
weight: 20
description: 了解 Docker 网络驱动的基础知识
keywords: networking, drivers, bridge, routing, routing mesh, overlay, ports
---

Docker 的网络子系统采用可插拔的驱动机制。默认提供若干驱动，覆盖核心网络功能：

- `bridge`：默认网络驱动。如果未显式指定驱动，将创建此类型的网络。桥接网络常用于当你的应用运行在容器中且需要与同一主机上的其他容器通信的场景。
  参见 [Bridge 网络驱动](bridge.md)。

- `host`：移除容器与 Docker 主机之间的网络隔离，直接使用主机网络。
  参见 [Host 网络驱动](host.md)。

- `overlay`：Overlay 网络用于连接多个 Docker 守护进程，使 Swarm 服务与容器能够跨节点通信，从而无需在操作系统层进行路由配置。
  参见 [Overlay 网络驱动](overlay.md)。

- `ipvlan`：为 IPv4 与 IPv6 地址分配提供完全控制。基于此，VLAN 驱动还为运维人员提供对二层 VLAN 标记以及 IPvlan 三层路由的完整控制，适用于需要与底层（underlay）网络集成的场景。
  参见 [IPvlan 网络驱动](ipvlan.md)。

- `macvlan`：允许为容器分配一个 MAC 地址，使其在你的网络中看起来像一台物理设备。Docker 守护进程会根据 MAC 地址将流量路由到容器。当处理期望直接连接到物理网络（而非经由主机网络栈路由）的遗留应用时，`macvlan` 往往是最佳选择。
  参见 [Macvlan 网络驱动](macvlan.md)。

- `none`：将容器与主机及其他容器完全隔离。`none` 不适用于 Swarm 服务。
  参见 [None 网络驱动](none.md)。

- [网络插件](/engine/extend/plugins_network/)：你可以在 Docker 中安装并使用第三方网络插件。

### 网络驱动摘要

- 默认 bridge 网络适合运行不需要特殊网络能力的容器。
- 用户自定义 bridge 网络使同一 Docker 主机上的容器能够相互通信。用户自定义网络通常为属于同一项目或组件的多个容器定义一个相对隔离的网络。
- Host 网络将主机网络与容器共享。使用该驱动时，容器网络不再与主机隔离。
- 当需要不同 Docker 主机上的容器进行通信，或多个应用需通过 Swarm 服务协同工作时，Overlay 网络是最佳选择。
- 当你从 VM 方案迁移，或需要容器在网络中表现得像物理主机（每个都有唯一 MAC 地址）时，Macvlan 最合适。
- IPvlan 与 Macvlan 类似，但不会为容器分配唯一的 MAC 地址。当网络接口或端口可分配的 MAC 数量受限时，优先考虑 IPvlan。
- 第三方网络插件可以让 Docker 集成专用的网络栈。

## 网络教程

现在你已了解 Docker 网络的基础知识，建议通过以下教程进一步加深理解：

- [独立网络教程](/manuals/engine/network/tutorials/standalone.md)
- [Host 网络教程](/manuals/engine/network/tutorials/host.md)
- [Overlay 网络教程](/manuals/engine/network/tutorials/overlay.md)
- [Macvlan 网络教程](/manuals/engine/network/tutorials/macvlan.md)
