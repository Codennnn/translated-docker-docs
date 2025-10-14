---
title: 选择存储驱动
weight: 10
description: 了解如何为你的容器选择合适的存储驱动。
keywords: 容器, 存储, 驱动, btrfs, zfs, overlay, overlay2
aliases:
  - /storage/storagedriver/selectadriver/
  - /storage/storagedriver/select-storage-driver/
---

理想情况下，容器的可写层应当只包含极少量数据，大部分数据应尽可能写入 Docker 卷。然而，某些工作负载确实需要向容器的可写层写入数据，这正是存储驱动发挥作用的场景。

Docker 采用可插拔架构，支持多种存储驱动。存储驱动负责控制镜像与容器在宿主机上的存储与管理方式。当你阅读完[存储驱动概览](./_index.md)后，下一步就是为你的工作负载选择最合适的驱动。一般情况下，应优先选择那些在常见场景中能够兼顾性能与稳定性的驱动。

> [!NOTE]
> 本页讨论的是 Linux 上 Docker Engine 的存储驱动。如果你的宿主机操作系统是 Windows，Docker 守护进程只支持 `windowsfilter` 存储驱动。参见
> [windowsfilter](windowsfilter-driver.md) 了解详情。

Docker Engine 在 Linux 上提供以下存储驱动：

| Driver            | Description                                                                                                                                                                                                                                                                                                                                          |
| :---------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `overlay2`        | `overlay2` 是当前所有受支持 Linux 发行版的首选存储驱动，无需额外配置即可直接使用。                                                                                                                                                                                                                     |
| `fuse-overlayfs`  | 仅在旧版宿主机不支持无根（rootless）模式下的 `overlay2` 时使用，是运行 Rootless Docker 的首选方案。从 Linux 5.11 开始，`overlay2` 已可在无根模式下正常工作，因此不再需要 `fuse-overlayfs`。详见[无根模式文档](/manuals/engine/security/rootless.md)。 |
| `btrfs` and `zfs` | `btrfs` 与 `zfs` 存储驱动支持诸如快照等高级功能，但需要更多的配置与维护工作。它们依赖于后端文件系统的正确配置。                                                                                                                                                                                             |
| `vfs`             | `vfs` 存储驱动主要用于测试目的，或用于无法使用写时复制（copy-on-write）文件系统的特殊场景。该驱动性能较差，不建议在生产环境中使用。                                                                                                                                                                                                    |

<!-- markdownlint-disable reference-links-images -->

当你未显式配置存储驱动且满足前置条件时，Docker Engine 会按优先级顺序自动选择一个兼容的存储驱动。你可以在[对应版本的 Docker Engine 源码 {{% param "docker_ce_version" %}}](https://github.com/moby/moby/blob/v{{% param "docker_ce_version" %}}/daemon/graphdriver/driver_linux.go#L52-L53)中查看这个选择顺序。
{ #storage-driver-order }

<!-- markdownlint-enable reference-links-images -->

部分存储驱动要求使用特定格式的后端文件系统。如果你必须使用某种特定的后端文件系统，这可能会限制你可选的存储驱动范围。请参见[支持的后端文件系统](#supported-backing-filesystems)了解详情。

在缩小可选范围后，你应当根据工作负载的特性以及对稳定性的要求来做最终选择。请参见[其他考虑因素](#other-considerations)以获得决策参考。

## 各发行版支持的存储驱动

> [!NOTE]
>
> 在 Docker Desktop 上，不支持通过编辑守护进程配置文件来修改存储驱动。仅支持默认的 `overlay2` 或 [containerd 存储](/manuals/desktop/features/containerd.md)。下表同样不适用于无根模式；关于无根模式下可用的驱动，请参见[无根模式文档](/manuals/engine/security/rootless.md)。

你的操作系统与内核版本并不一定支持所有存储驱动。例如，只有当系统的后端文件系统使用 `btrfs` 时，才支持 `btrfs` 驱动。一般来说，以下配置在较新的 Linux 发行版上可以正常工作：

| Linux 发行版         | 推荐存储驱动                   | 可选替代驱动          |
| :------------------- | :--------------------------- | :------------------- |
| Ubuntu               | `overlay2`                   | `zfs`, `vfs`         |
| Debian               | `overlay2`                   | `vfs`                |
| CentOS               | `overlay2`                   | `zfs`, `vfs`         |
| Fedora               | `overlay2`                   | `zfs`, `vfs`         |
| SLES 15              | `overlay2`                   | `vfs`                |
| RHEL                 | `overlay2`                   | `vfs`                |

如果拿不定主意，最佳的通用做法是：选择支持 `overlay2` 的现代 Linux 发行版与内核版本，并将写入密集型工作负载的数据写入 Docker 卷，而不是依赖容器的可写层。

`vfs` 存储驱动通常不是最佳选择，它主要用于其他驱动不可用时的故障排查与调试场景。使用前请务必阅读其[性能、存储特性及局限性](vfs-driver.md)。

上表中的推荐组合已在大量用户场景中得到验证。如果你在推荐配置下遇到可复现的问题，通常会很快得到修复。若你选择表中未推荐的驱动组合，请自行承担风险；虽然你仍然应当报告遇到的问题，但这些问题的优先级可能会低于使用推荐配置时遇到的问题。

不同发行版可能还提供其他驱动（如 `btrfs`）。这些驱动在特定场景下可能具有优势，但通常需要额外的配置与维护工作，因此不推荐在常见场景中使用。更多详情请参见各驱动的专门文档。

## 支持的后端文件系统

在 Docker 语境中，后端文件系统是指 `/var/lib/docker/` 目录所在的文件系统。部分存储驱动只支持特定的后端文件系统。

| 存储驱动         | 支持的后端文件系统                                     |
| :--------------- | :-----------------------------------------------------|
| `overlay2`       | `xfs`（ftype=1）、`ext4`、`btrfs`（以及更多）            |
| `fuse-overlayfs` | 任意文件系统                                          |
| `btrfs`          | `btrfs`                                               |
| `zfs`            | `zfs`                                                 |
| `vfs`            | 任意文件系统                                          |

> [!NOTE]
>
> 只要文件系统具备必要的特性，大多数文件系统都能正常工作。更多信息可参考 [OverlayFS 内核文档](https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html)。


## 其他考虑因素

### 与工作负载的适配性

每种存储驱动都有各自的性能特征，使其更适合或不太适合某些特定的工作负载。以下是一些经验参考：

- `overlay2` 在文件级别（而非块级别）工作，内存使用更加高效，但在写入密集型场景下，容器可写层可能增长较快。
- `btrfs`、`zfs` 等块级驱动在写入密集型工作负载上表现更好（但仍不如直接使用 Docker 卷）。
- `btrfs` 与 `zfs` 的内存消耗相对较高。
- 在 PaaS 等高密度部署场景中，`zfs` 是不错的选择。

关于性能、适配性与最佳实践的更多详细信息，请查阅各存储驱动的专门文档。

### 共享存储系统与存储驱动

如果你使用 SAN、NAS、硬件 RAID 或其他共享存储系统，这些系统可能会提供高可用性、性能增强、精简配置、去重与压缩等功能。在许多场景中，Docker 可以运行在这些系统之上，但 Docker 并不会与它们进行深度集成。

每种 Docker 存储驱动都基于某种 Linux 文件系统或卷管理器。在共享存储系统之上部署时，请遵循这些文件系统或卷管理器的既有最佳实践。例如，如果在共享存储系统之上使用 ZFS 存储驱动，请遵循在该系统上部署与运维 ZFS 的最佳实践。

### 稳定性

对部分用户而言，稳定性比性能更加重要。虽然本文提到的各存储驱动均被视为稳定，但有些驱动相对较新，仍在积极演进中。通常来说，`overlay2` 的稳定性最高。

### 使用你的实际负载进行测试

你可以在不同的存储驱动上运行自身的工作负载，以测试 Docker 在实际场景中的表现。请尽量在等效的硬件配置与负载条件下进行测试，以便更接近生产环境，从而更准确地评估哪种驱动的综合性能更优。

## 查看当前存储驱动

各存储驱动的详细文档都会完整说明使用该驱动所需的配置步骤。

要查看 Docker 当前使用的存储驱动，请运行 `docker info` 命令并关注 `Storage Driver` 这一行：

```console
$ docker info

Containers: 0
Images: 0
Storage Driver: overlay2
 Backing Filesystem: xfs
<...>
```

如需更换存储驱动，请参见目标驱动的专门说明文档。部分驱动需要额外配置，包括在宿主机上配置物理或逻辑磁盘。

> [!IMPORTANT]
>
> 当你切换存储驱动时，现有的镜像与容器将变得不可访问，因为它们的层数据无法被新驱动使用。如果你回滚到原来的驱动，旧的镜像与容器会恢复可访问状态，但在新驱动下拉取或创建的镜像与容器则会变得不可访问。

## 相关信息

- [存储驱动](./_index.md)
- [`overlay2` 存储驱动](overlayfs-driver.md)
- [`btrfs` 存储驱动](btrfs-driver.md)
- [`zfs` 存储驱动](zfs-driver.md)
- [`windowsfilter` 存储驱动](windowsfilter-driver.md)
