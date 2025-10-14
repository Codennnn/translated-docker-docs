---
title: 使用 Docker Engine 插件
aliases:
- "/engine/extend/plugins/"
description: "如何通过插件为 Docker 增加功能"
keywords: "Examples, Usage, plugins, docker, documentation, user guide"
---

本文介绍 Docker Engine 中普遍可用的插件。若需了解由 Docker 托管（Managed）的插件，请参阅《[Docker Engine 插件系统](_index.md)》。

你可以通过加载第三方插件来扩展 Docker Engine 的能力。本文说明插件的类型，并提供若干卷与网络插件的链接。

## 插件类型

插件用于扩展 Docker 的功能，按类型划分。例如，[卷插件](plugins_volume.md) 可以让卷在多个 Docker 主机之间保持持久化；[网络插件](plugins_network.md) 则提供网络连通能力。

目前 Docker 支持授权（authorization）、卷（volume）与网络（network）驱动插件；未来可能会支持更多类型。

## 安装插件

请按照各插件文档中的说明进行安装。

## 查找插件

下列各节对部分第三方插件进行概览。

### 网络插件

| 插件                                                                                | 描述                                                                                                                                                                                                                                                                                                                                                 |
| :--------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Contiv Networking](https://github.com/contiv/netplugin)                           | 开源网络插件，为多租户微服务部署提供基础设施与安全策略，同时为非容器工作负载提供与物理网络的集成。实现了 Docker 1.9 及以上版本提供的远程驱动与 IPAM API。                                                                                                                                                |
| [Kuryr Network Plugin](https://github.com/openstack/kuryr)                         | OpenStack Kuryr 项目的一部分，利用 OpenStack 网络服务 Neutron 实现 Docker 网络（libnetwork）远程驱动 API，并包含一个 IPAM 驱动。                                                                                                                                                                           |
| [Kathará Network Plugin](https://github.com/KatharaFramework/NetworkPlugin)        | 由 Kathará 使用的 Docker 网络插件。Kathará 是基于容器的开源网络仿真系统，可用于交互式演示/课程、在沙箱环境测试生产网络，或开发新的网络协议。                                                                                                                                                                 |

### 卷插件

| 插件                                                                                              | 描述                                                                                                                                                                                                                                                                                                            |
|:---------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Azure File Storage plugin](https://github.com/Azure/azurefile-dockervolumedriver)                 | 使用 SMB 3.0 协议，将 Microsoft [Azure File Storage](https://azure.microsoft.com/blog/azure-file-storage-now-generally-available/) 共享以卷的形式挂载到 Docker 容器中。[了解更多](https://azure.microsoft.com/blog/persistent-docker-volumes-with-azure-file-storage/)。                                     |
| [BeeGFS Volume Plugin](https://github.com/RedCoolBeans/docker-volume-beegfs)                       | 开源卷插件，在 BeeGFS 并行文件系统中创建持久卷。                                                                                                                                                                                                                                                              |
| [Blockbridge plugin](https://github.com/blockbridge/blockbridge-docker-volume)                     | 提供对可扩展的基于容器的持久存储选项的访问。支持单机与多主机 Docker 环境，具备租户隔离、自动化供应、加密、安全删除、快照与 QoS 等特性。                                                                                                                                                                        |
| [Contiv Volume Plugin](https://github.com/contiv/volplugin)                                        | 开源卷插件，提供多租户、持久、分布式存储，并支持基于意图的消费模型。支持 Ceph 与 NFS。                                                                                                                                                                                                                        |
| [Convoy plugin](https://github.com/rancher/convoy)                                                 | 支持多种后端（包括 device mapper 与 NFS）的卷插件。它是用 Go 编写的独立可执行文件，并提供支持厂商扩展（如快照、备份与恢复）的框架。                                                                                                                                                                              |
| [DigitalOcean Block Storage plugin](https://github.com/omallo/docker-volume-plugin-dostorage)      | 将 DigitalOcean 的[块存储](https://www.digitalocean.com/products/storage/) 集成到 Docker 生态中，自动将指定块存储卷挂载到 Droplet，并将卷内容提供给运行于该 Droplet 上的容器使用。                                                                                                                             |
| [DRBD plugin](https://www.drbd.org/en/supported-projects/docker)                                   | 通过 [DRBD](https://www.drbd.org) 复制实现高可用存储的卷插件。写入到 Docker 卷的数据会在 DRBD 节点集群中复制。                                                                                                                                                                                                  |
| [Flocker plugin](https://github.com/ScatterHQ/flocker)                                             | 提供多主机可移植卷，使你能够运行数据库等有状态容器，并在一组机器之间迁移它们。                                                                                                                                                                                                                                |
| [Fuxi Volume Plugin](https://github.com/openstack/fuxi)                                            | OpenStack Kuryr 项目的一部分，利用 OpenStack 块存储服务 Cinder 实现 Docker 卷插件 API。                                                                                                                                                                                                                        |
| [gce-docker plugin](https://github.com/mcuadros/gce-docker)                                        | 可附加、格式化并挂载 Google Compute 的[持久磁盘](https://cloud.google.com/compute/docs/disks/persistent-disks) 的卷插件。                                                                                                                                                                                     |
| [GlusterFS plugin](https://github.com/calavera/docker-volume-glusterfs)                            | 使用 GlusterFS 为 Docker 提供多主机卷管理的插件。                                                                                                                                                                                                                                                             |
| [Horcrux Volume Plugin](https://github.com/muthu-r/horcrux)                                        | 支持按需访问与版本控制的数据访问。该开源插件由 Go 编写，支持 SCP、[MinIO](https://www.minio.io) 与 Amazon S3。                                                                                                                                                                                                  |
| [HPE 3Par Volume Plugin](https://github.com/hpe-storage/python-hpedockerplugin/)                   | 支持 HPE 3Par 与 StoreVirtual iSCSI 存储阵列的卷插件。                                                                                                                                                                                                                                                         |
| [Infinit volume plugin](https://infinit.sh/documentation/docker/volume-plugin)                     | 便于通过 Docker 挂载与管理 Infinit 卷的插件。                                                                                                                                                                                                                                                                  |
| [IPFS Volume Plugin](https://github.com/vdemeester/docker-volume-ipfs)                              | 允许将 [IPFS](https://ipfs.io/) 文件系统作为卷使用的开源插件。                                                                                                                                                                                                                                                 |
| [Keywhiz plugin](https://github.com/calavera/docker-volume-keywhiz)                                | 使用 Keywhiz 作为集中式仓库，提供凭据与机密管理的插件。                                                                                                                                                                                                                                                        |
| [Linode Volume Plugin](https://github.com/linode/docker-volume-linode)                             | 使你能够在 Linode 中将 Linode Block Storage 作为 Docker 卷进行管理的插件。                                                                                                                                                                                                                                      |
| [Local Persist Plugin](https://github.com/CWSpear/local-persist)                                   | 扩展默认 `local` 驱动的功能，允许你在宿主机任意位置指定挂载点，使文件即便在通过 `docker volume rm` 移除卷后也能始终持久存在。                                                                                                                                                                                   |
| [NetApp Plugin](https://github.com/NetApp/netappdvp) (nDVP)                                        | 为 NetApp 存储产品提供与 Docker 生态的直接集成。nDVP 包支持从存储平台到 Docker 主机的资源供应与管理，并提供稳健的框架以便未来添加更多平台。                                                                                                                                                                       |
| [Netshare plugin](https://github.com/ContainX/docker-volume-netshare)                              | 为 NFS 3/4、AWS EFS 与 CIFS 文件系统提供卷管理的插件。                                                                                                                                                                                                                                                          |
| [Nimble Storage Volume Plugin](https://scod.hpedev.io/docker_volume_plugins/hpe_nimble_storage/index.html) | 与 Nimble Storage Unified Flash Fabric 阵列集成的卷插件。该插件将阵列卷能力抽象给 Docker 管理员，便于自助式地供应安全的多租户卷与克隆。                                                                                                                                                                            |
| [OpenStorage Plugin](https://github.com/libopenstorage/openstorage)                                | 面向集群的卷插件，为文件与块存储方案提供卷管理。它实现了厂商无关的扩展规范（如 CoS、加密与快照），并包含基于 FUSE、NFS、NBD、EBS 等的示例驱动。                                                                                                                                                                  |
| [Portworx Volume Plugin](https://github.com/portworx/px-dev)                                       | 将任意服务器变为可横向扩展的融合计算/存储节点，提供以容器为粒度的存储与跨任意节点的高可用卷，采用与任意 Docker 调度器配合的无共享后端。                                                                                                                                                                          |
| [Quobyte Volume Plugin](https://github.com/quobyte/docker-volume)                                  | 将 Docker 连接到 [Quobyte](https://www.quobyte.com/containers) 数据中心文件系统的卷插件，这是一种通用、可扩展且容错的存储平台。                                                                                                                                                                                   |
| [REX-Ray plugin](https://github.com/emccode/rexray)                                                | 使用 Go 编写，为多个平台提供高级存储功能的卷插件，包括 VirtualBox、EC2、Google Compute Engine、OpenStack 与 EMC。                                                                                                                                                                                               |
| [Virtuozzo Storage and Ploop plugin](https://github.com/virtuozzo/docker-volume-ploop)             | 支持 Virtuozzo Storage 分布式云文件系统以及 ploop 设备的卷插件。                                                                                                                                                                                                                                                |
| [VMware vSphere Storage Plugin](https://github.com/vmware/docker-volume-vsphere)                   | 面向 vSphere 的 Docker 卷驱动，帮助在 vSphere 环境中满足容器的持久化存储需求。                                                                                                                                                                                                                                   |

### 授权插件

| 插件                                                                | 描述                                                                                                                                                                                                                                                                                                                               |
|:------------------------------------------------------------------- |:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Casbin AuthZ Plugin](https://github.com/casbin/casbin-authz-plugin) | 基于 [Casbin](https://github.com/casbin/casbin) 的授权插件，支持 ACL、RBAC、ABAC 等访问控制模型；访问控制模型可自定义；策略可持久化到文件或数据库。                                                                                                                                                                  |
| [HBM plugin](https://github.com/kassisol/hbm)                        | 一款授权插件，用于阻止带有特定参数的命令被执行。                                                                                                                                                                                                                                                                                  |
| [Twistlock AuthZ Broker](https://github.com/twistlock/authz)         | 基础且可扩展的授权插件，可直接在宿主机或容器内运行。该插件允许你定义用户策略，并在授权阶段进行评估。当 Docker 守护进程使用 `--tlsverify` 启动时，它提供基础授权（用户名从证书的 Common Name 中提取）。                                                                                                       |

## 插件疑难解答

如果加载插件后遇到 Docker 问题，请向该插件的作者寻求帮助。Docker 团队可能无法协助你解决这类问题。

## 编写插件

如果你有兴趣为 Docker 编写插件，或想了解其底层工作方式，请参阅《[Docker 插件参考](plugin_api.md)》。
