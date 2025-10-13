---
description: 回顾 Docker 守护进程的攻击面与安全要点
keywords: Docker, Docker 文档, 安全
title: Docker Engine 安全
linkTitle: 安全
weight: 80
aliases:
- /articles/security/
- /engine/articles/security/
- /engine/security/security/
- /security/security/
---

审视 Docker 安全性时，主要需要关注以下四个方面：

 - 内核自身的安全性，以及对命名空间（namespaces）与控制组（cgroups）的支持
 - Docker 守护进程本身的攻击面
 - 容器配置（默认或用户自定义）中的漏洞
 - 内核的各种“加固”安全特性以及它们与容器的交互方式

## 内核命名空间（Kernel namespaces）

Docker 容器与 LXC 容器非常相似，并具备类似的安全特性。使用 `docker run` 启动容器时，Docker 会在后台为该容器创建一组命名空间与控制组。

命名空间提供了第一层、也是最直接的隔离机制。容器内的进程无法看到、更无法影响其他容器或宿主机中的进程。

每个容器也拥有独立的网络协议栈，这意味着它无法以特权方式访问其他容器的套接字或网络接口。当然，如果宿主机配置允许，容器可以通过各自的网络接口进行互通——就像与外部主机交互一样。为容器公开端口或使用
[links](/manuals/engine/network/links.md) 时，容器之间允许 IP 层面的通信：它们可以互相 ping、收发 UDP 包、建立 TCP 连接；必要时也可以进行限制。从网络架构角度看，同一宿主机上的所有容器都位于桥接接口之上，这与多台物理机器通过以太网交换机相连并无二致。

用于实现命名空间与专用网络的内核代码有多成熟？命名空间是在
[2.6.15 到 2.6.26](https://man7.org/linux/man-pages/man7/namespaces.7.html)
版本之间引入的。自 2008 年 7 月（2.6.26 发布）以来，命名空间代码已在大量生产系统中得到验证与审视。此外，其设计与灵感更早可追溯至 [OpenVZ](https://en.wikipedia.org/wiki/OpenVZ)：命名空间实际上是将 OpenVZ 的特性以能被主线内核接纳的方式重新实现。OpenVZ 早在 2005 年发布，因此无论设计还是实现都相当成熟。

## 控制组（Control groups, cgroups）

控制组是 Linux 容器的另一关键组件，用于实现资源记账与限制。它不仅提供诸多有用的度量指标，更能确保每个容器公平地使用内存、CPU、磁盘 I/O 等资源；更重要的是，避免单个容器因耗尽某项资源而拖垮整机。

虽然控制组并不直接用于阻止容器之间访问或影响彼此的数据与进程，但它对于防范某些拒绝服务攻击（DoS）至关重要。尤其在多租户平台（如公有或私有 PaaS）上，即便部分应用出现异常，也能帮助保障总体可用性与性能。

控制组同样历史悠久：相关代码始于 2006 年，并最初在 2.6.24 内核中合入主线。

## Docker 守护进程的攻击面

用 Docker 运行容器（与应用）意味着需要运行 Docker 守护进程。除非启用 [Rootless 模式](rootless.md)，该守护进程需要 `root` 权限，因此你需要了解一些重要事项。

首先，仅允许可信用户控制你的 Docker 守护进程。这直接源于 Docker 的强大能力：例如，它允许在宿主机与容器间共享目录，且可以不限制容器对该目录的访问权限。这意味着你可以启动一个容器，把宿主机的 `/` 挂载为容器内的 `/host`，容器即可不受限制地修改宿主机文件系统。这与虚拟化系统的文件系统共享机制类似——并没有什么能阻止你把根文件系统（甚至根块设备）共享给一台虚拟机。

这带来重要的安全影响：例如，若你通过 Web 服务器暴露 API 来调用 Docker 创建容器，就必须格外谨慎地校验参数，防止恶意用户通过构造参数让 Docker 创建任意容器。

基于上述原因，Docker 0.5.2 起用于 CLI 与守护进程通信的 REST API 端点改为使用 Unix 套接字，替代原先绑定在 127.0.0.1 的 TCP 套接字（如果你在非虚拟机的本机上直接运行 Docker，后者容易遭受跨站请求伪造攻击）。借助传统的 Unix 权限控制，你可以限制对该控制套接字的访问。

如有明确需求，你也可以通过 HTTP 暴露 REST API。但必须意识到上述安全影响。即使使用了防火墙限制来自网络其他主机对 REST API 的访问，该端点仍可能被容器访问，从而导致轻易的权限提升。因此，使用
[HTTPS 与证书](protect-access.md) 保护 API 端点是*强制性*要求。禁止在无 TLS 的 HTTP 上暴露守护进程 API；此类配置会导致守护进程在启动早期即失败，参见
[Unauthenticated TCP connections](../deprecated.md#unauthenticated-tcp-connections)。同时建议仅允许可信网络或 VPN 访问该端点。

如果你更偏好 SSH 而非 TLS，也可以使用 `DOCKER_HOST=ssh://USER@HOST` 或 `ssh -L /path/to/docker.sock:/var/run/docker.sock`。

守护进程还可能受到其他输入向量影响，例如从磁盘加载镜像（`docker load`）或从网络拉取镜像（`docker pull`）。自 Docker 1.3.2 起，Linux/Unix 平台上解包镜像的过程在 chroot 的子进程中完成，作为更大范围“权限分离”工作的第一步。自 Docker 1.10.0 起，所有镜像均以内容的加密校验和进行存储与访问，从而降低攻击者与现有镜像产生哈希碰撞的可能。

最后，如果你在服务器上运行 Docker，建议该服务器尽可能只运行 Docker，将其他服务迁移到由 Docker 管理的容器中。当然，保留常用的管理工具（至少包括 SSH 服务器）以及既有的监控/运维进程（如 NRPE、collectd）是没问题的。

## Linux 内核能力（Capabilities）

默认情况下，Docker 以受限的能力集合（capabilities）启动容器。这意味着什么？

能力模型将“root/非 root”的二元划分转化为更细粒度的访问控制系统。比如，仅需绑定 1024 以下端口的进程（如 Web 服务器）不必以 root 身份运行，只需授予 `net_bind_service` 能力即可。还有许多其他能力覆盖几乎所有通常需要 root 权限的场景，这对容器安全意义重大。

典型的服务器会以 `root` 身份运行多个进程，包括 SSH 守护进程、`cron`、日志守护进程、内核模块、网络配置工具等。容器的情况不同，因为这些任务大多由容器外部的基础设施来承担：

 - SSH 访问通常由宿主机上的单个 SSH 服务器统一管理
 - 如需 `cron`，应作为应用的用户进程运行，为该应用量身定制，而非平台级公共服务
 - 日志管理通常交由 Docker 或第三方服务（如 Loggly、Splunk）处理
 - 硬件管理与容器无关，你无需在容器内运行 `udevd` 或类似守护进程
 - 网络管理发生在容器之外，以尽量强化关注点分离；容器通常不应执行 `ifconfig`、`route` 或 `ip` 命令（除非该容器专门被设计为路由器或防火墙）

这意味着在大多数情况下，容器根本不需要“真正的”root 权限。因此，容器可以在裁剪后的能力集合下运行，也就是说容器内的“root”拥有的权限远少于宿主级的 root。例如，可以：

 - 拒绝所有“mount”操作
 - 拒绝访问原始套接字（防止数据包伪造）
 - 拒绝部分文件系统操作，如创建设备节点、修改文件所有者或更改属性（包括不可变标志）
 - 拒绝内核模块加载

这意味着即使入侵者在容器内提升到了 root 权限，也更难造成严重破坏，或进一步升级至宿主机。

这对常规的 Web 应用影响不大，却显著减少了恶意用户的攻击面。默认情况下，Docker 会移除除
[必要能力](https://github.com/moby/moby/blob/master/daemon/pkg/oci/caps/defaults.go#L6-L19)
之外的所有能力，即采用允许列表（allowlist）而非拒绝列表（denylist）策略。完整能力列表可参见
[Linux manpages](https://man7.org/linux/man-pages/man7/capabilities.7.html)。

运行 Docker 容器的一个主要风险在于：容器默认授予的能力与挂载集合，可能单独或与内核漏洞叠加时，导致隔离不完整。

Docker 支持增删能力集合，从而使用非默认的配置。移除不必要的能力可以提升安全性；反之，增加能力可能降低安全性。最佳实践是仅保留应用显式需要的能力，移除其余能力。

## Docker 内容信任（Docker Content Trust）签名验证

Docker Engine 可以配置为仅运行已签名的镜像。Docker 内容信任（Content Trust）的签名验证功能直接内置于 `dockerd` 可执行文件中，并通过 Dockerd 配置文件启用。

要启用该功能，可在 `daemon.json` 中配置 trustpinning，使得仅允许拉取并运行由用户指定根密钥签名的仓库。

相较于以往仅依赖 CLI 强制与执行镜像签名验证的做法，该功能能为管理员提供更多可见性与更强的约束能力。

有关配置 Docker 内容信任签名验证的更多信息，参见
[Content trust in Docker](trust/_index.md)。

## 其他内核安全特性

能力模型只是现代 Linux 内核众多安全特性之一。配合 Docker，还可以利用成熟的安全机制，如 TOMOYO、AppArmor、SELinux、GRSEC 等。

尽管 Docker 当前只默认启用能力模型，但不会妨碍其他安全系统的使用。因此有多种方式可以加固 Docker 宿主机。例如：

 - 运行带有 GRSEC 与 PAX 的内核：它在编译期与运行期加入大量安全检查，并通过地址随机化等技术抵御多种利用方式。这些特性系统级生效，无需 Docker 特定配置。
 - 使用发行版提供的 Docker 容器安全模板：例如，我们提供了适用于 AppArmor 的模板；Red Hat 提供了 Docker 的 SELinux 策略。这些模板可作为额外的安全保障（尽管与能力模型部分重叠）。
 - 使用你偏好的访问控制机制自定义策略。

正如可以使用第三方工具为容器引入特殊网络拓扑或共享文件系统一样，也存在很多工具可以在无需修改 Docker 本身的前提下加强容器安全。

自 Docker 1.10 起，Docker 守护进程原生支持用户命名空间（User Namespaces）。该特性允许将容器内的 root 用户映射为容器外的非 0 号用户，从而降低容器逃逸的风险。该功能可用但默认未启用。

更多信息可参考命令行文档中的
[daemon 命令](/reference/cli/dockerd.md#daemon-user-namespace-options)。
关于 Docker 中用户命名空间实现的更多背景，参见
[这篇博客](https://integratedcode.us/2015/10/13/user-namespaces-have-arrived-in-docker/)。

## 结论

默认情况下，Docker 容器相当安全；尤其是在容器内以非特权用户运行进程时。

通过启用 AppArmor、SELinux、GRSEC 或其他合适的加固系统，你可以进一步提升安全保障。

如果你有让 Docker 更加安全的想法，欢迎在 Docker 社区发起特性建议、提交 PR 或参与讨论。

## 相关信息

* [使用可信镜像](trust/_index.md)
* [Docker 的 Seccomp 安全配置](seccomp.md)
* [Docker 的 AppArmor 安全配置](apparmor.md)
* [On the Security of Containers (2014)](https://medium.com/@ewindisch/on-the-security-of-containers-2c60ffe25a9e)
* [Docker Swarm 模式 Overlay 网络安全模型](/manuals/engine/network/drivers/overlay.md)
