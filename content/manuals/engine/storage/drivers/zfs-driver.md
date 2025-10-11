---
description: 了解如何更高效地使用 ZFS 存储驱动。
keywords: 容器, 存储, 驱动, ZFS
title: ZFS 存储驱动
aliases:
  - /storage/storagedriver/zfs-driver/
---

ZFS 是下一代文件系统，支持多种高级存储技术，例如卷管理、快照、校验、压缩与去重、复制等。

ZFS 由 Sun Microsystems（现 Oracle）创建，并以 CDDL 许可开源。由于 CDDL 与 GPL 授权不兼容，ZFS 无法作为主线 Linux 内核的一部分发布。但 ZFS On Linux（ZoL）项目提供了独立于内核树之外的内核模块与用户态工具，可单独安装。

ZFS on Linux（ZoL）移植项目成熟度良好、正持续完善。但在当前时点，除非你对 Linux 上的 ZFS 具有相当经验，否则不建议在生产环境使用 `zfs` Docker 存储驱动。

> [!NOTE]
>
> Linux 平台上还存在基于 FUSE 的 ZFS 实现。并不推荐使用。原生 ZFS 驱动（ZoL）测试更充分、性能更好，且使用更广泛。本文余下内容均指代原生 ZoL 实现。

## 先决条件

- ZFS 需要一个或多个专用块设备，优先推荐使用固态硬盘（SSD）。
- `/var/lib/docker/` 目录必须挂载在使用 ZFS 格式化的文件系统上。
- 更换存储驱动会导致本机已创建的容器不可访问。请使用 `docker save` 保存容器，并将现有镜像推送到 Docker Hub 或私有仓库，以免之后需要重新创建。

> [!NOTE]
>
> 无需使用 `MountFlags=slave`，因为 `dockerd` 与 `containerd` 处于不同的挂载命名空间。

## 配置 Docker 使用 `zfs` 存储驱动

1.  停止 Docker。

2.  将 `/var/lib/docker/` 的内容复制到 `/var/lib/docker.bk`，然后清空 `/var/lib/docker/`：

    ```console
    $ sudo cp -au /var/lib/docker /var/lib/docker.bk

    $ sudo rm -rf /var/lib/docker/*
    ```

3.  在专用块设备上创建新的 `zpool` 并将其挂载到 `/var/lib/docker/`。务必确认设备无误，因为该操作具有破坏性。以下示例向池中添加两块设备：

    ```console
    $ sudo zpool create -f zpool-docker -m /var/lib/docker /dev/xvdf /dev/xvdg
    ```

    上述命令会创建名为 `zpool-docker` 的 `zpool`。该名称仅用于显示，你也可以使用其他名称。可通过 `zfs list` 检查池是否已正确创建并挂载：

    ```console
    $ sudo zfs list

    NAME           USED  AVAIL  REFER  MOUNTPOINT
    zpool-docker    55K  96.4G    19K  /var/lib/docker
    ```

4.  配置 Docker 使用 `zfs`。编辑 `/etc/docker/daemon.json`，将 `storage-driver` 设置为 `zfs`。若文件先前为空，应如下所示：

    ```json
    {
      "storage-driver": "zfs"
    }
    ```

    保存并关闭文件。

5.  启动 Docker。使用 `docker info` 验证当前存储驱动为 `zfs`：

    ```console
    $ sudo docker info
      Containers: 0
       Running: 0
       Paused: 0
       Stopped: 0
      Images: 0
      Server Version: 17.03.1-ce
      Storage Driver: zfs
       Zpool: zpool-docker
       Zpool Health: ONLINE
       Parent Dataset: zpool-docker
       Space Used By Parent: 249856
       Space Available: 103498395648
       Parent Quota: no
       Compression: off
    <...>
    ```

## 管理 `zfs`

### 在线扩容设备容量

要增加 `zpool` 的容量，需要先向 Docker 宿主机添加专用块设备，然后使用 `zpool add` 将其加入 `zpool`：

```console
$ sudo zpool add zpool-docker /dev/xvdh
```

### 限制容器可写存储配额

如果你希望按镜像/数据集维度设置配额，可以通过 `size` 存储选项限制单个容器可写层可用的空间。

编辑 `/etc/docker/daemon.json`，添加如下内容：

```json
{
  "storage-driver": "zfs",
  "storage-opts": ["size=256M"]
}
```

各存储驱动支持的所有存储选项，参见[守护进程参考文档](/reference/cli/dockerd/#daemon-storage-driver)

保存关闭文件，并重启 Docker。

## `zfs` 存储驱动的工作原理

ZFS 使用以下对象：

- **filesystems（文件系统）**：精简配置，按需从 `zpool` 分配空间。
- **snapshots（快照）**：文件系统在某时刻的只读、空间高效的副本。
- **clones（克隆）**：快照的读写副本，用于保存相对于前一层的差异。

克隆的创建过程：

![ZFS snapshots and clones](images/zfs_clones.webp?w=450)


1.  基于文件系统创建一个只读快照。
2.  基于该快照创建一个可写克隆，其中包含相对父层的所有差异。

文件系统、快照与克隆均从底层 `zpool` 分配空间。

### 磁盘上的镜像与容器层

每个正在运行容器的统一文件系统都会挂载在 `/var/lib/docker/zfs/graph/` 下的一个挂载点。继续阅读了解该统一文件系统是如何组成的。

### 镜像分层与共享

镜像的基底层是一个 ZFS 文件系统。每个子层是基于其下层快照创建的 ZFS 克隆。容器本身是基于镜像顶层快照创建的一个 ZFS 克隆。

下图展示了一个基于两层镜像的运行中容器，它们是如何组合在一起的：

![ZFS pool for Docker container](images/zfs_zpool.webp?w=600)

启动容器时，按以下顺序发生：

1.  镜像的基底层以 ZFS 文件系统的形式存在于宿主机上。

2.  额外的镜像层是对其下方镜像层所在数据集的克隆。

    例如图中 “Layer 1” 的添加过程：先对基底层进行 ZFS 快照，再基于该快照创建克隆。该克隆可写，并按需从 zpool 分配空间；快照为只读，保证基底层不可变。

3.  启动容器时，会在镜像之上添加一个可写层。

    如图所示，容器的读写层是对镜像顶层（Layer 1）创建快照并基于该快照创建克隆生成的。

4.  随着容器修改其可写层内容，系统会为发生变化的块分配空间。默认块大小为 128k。


## 使用 `zfs` 时容器的读写机制

### 读取文件

每个容器的可写层是一个 ZFS 克隆，它与其来源数据集（父层的快照）共享所有数据。即便读取的数据位于很深的层级，读取操作依然很快。下图展示了块共享的工作方式：

![ZFS block sharing](images/zpool_blocks.webp?w=450)


### 写入文件

**写入新文件**：按需从底层 `zpool` 分配空间，并将块直接写入容器可写层。

**修改已有文件**：仅为变更的块分配空间，并使用写时复制（CoW）策略将这些块写入容器可写层。这样既能最小化层的体积，也能提升写入性能。

**删除文件或目录**：
  - 当删除位于更低层的文件或目录时，ZFS 驱动会在容器可写层对其进行“遮蔽”。尽管该文件或目录仍存在于更低的只读层，但对容器不可见。
  - 若在容器可写层中创建并随后删除文件或目录，相应块会被 `zpool` 回收。


## ZFS 与 Docker 性能

在使用 `zfs` 存储驱动时，有多种因素会影响 Docker 的性能：

- **内存**：内存对 ZFS 性能影响重大。ZFS 最初面向具备大量内存的大型企业级服务器设计。

- **ZFS 特性**：ZFS 包含去重功能。启用去重可节省磁盘空间，但会占用大量内存。除非使用 SAN、NAS 或其他硬件 RAID，否则建议在用于 Docker 的 `zpool` 上禁用该功能。

- **ZFS 缓存**：ZFS 使用自适应替换缓存（ARC）在内存中缓存磁盘块。其 Single Copy ARC 特性允许多个克隆共享同一块的缓存副本。借助该特性，多容器可共享同一缓存块，使 ZFS 适合 PaaS 等高密度场景。

- **碎片化**：碎片化是 ZFS 等写时复制文件系统的自然副产物。ZFS 通过较小的 128k 块大小缓解该问题；ZFS 意图日志（ZIL）与写入合并（延迟写）也有所帮助。可通过 `zpool status` 监控碎片情况；但若不重新格式化并恢复文件系统，无法对 ZFS 进行碎片整理。

- **使用 Linux 原生 ZFS 驱动**：不推荐使用 ZFS 的 FUSE 实现，其性能较差。

### 性能最佳实践

- **使用更快的存储**：固态硬盘（SSD）通常比机械硬盘提供更快的读写性能。

- **写入密集型工作负载优先使用卷**：卷通常提供最佳且可预测的性能，因为它们绕过存储驱动，避免精简配置与写时复制带来的开销。卷还便于容器间共享数据，并在没有容器使用时依然持久存在。
