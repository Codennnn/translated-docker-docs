---
description: 了解如何更高效地使用 Btrfs 存储驱动。
keywords: 容器, 存储, 驱动, Btrfs
title: Btrfs 存储驱动
aliases:
  - /storage/storagedriver/btrfs-driver/
---

> [!IMPORTANT]
>
> 大多数情况下，你都应使用 `overlay2` 存储驱动。仅仅因为系统的根文件系统是 Btrfs，并不意味着必须使用 `btrfs` 存储驱动。
>
> `btrfs` 驱动已知存在一些问题。更多信息参见 [Moby issue #27653](https://github.com/moby/moby/issues/27653)。

Btrfs 是一种写时复制（CoW）文件系统，支持多种高级存储技术，因而非常适合 Docker 使用。Btrfs 已被合入 Linux 主线内核。

Docker 的 `btrfs` 存储驱动在镜像与容器管理中充分利用了 Btrfs 的能力，包括块级操作、精简配置、写时复制快照以及更易维护等。你可以将多个物理块设备聚合为一个 Btrfs 文件系统。

本文将 Docker 的 Btrfs 存储驱动称为 `btrfs`，将该文件系统本身称为 Btrfs。

> [!NOTE]
>
> `btrfs` 存储驱动仅在 SLES、Ubuntu 与 Debian 上的 Docker Engine CE 受到支持。

## 先决条件

满足以下条件即可使用 `btrfs`：

- 仅推荐在 Ubuntu 或 Debian 的 Docker CE 上使用 `btrfs`。

- 更换存储驱动会导致本机上已创建的容器不可访问。请使用 `docker save` 备份容器，并将现有镜像推送到 Docker Hub 或私有仓库，以免之后需要重新创建。

- `btrfs` 需要专用的块设备（如物理磁盘）。该块设备必须被格式化为 Btrfs，并挂载到 `/var/lib/docker/`。下面的配置步骤会引导你完成这一过程。SLES 的根分区 `/` 默认使用 Btrfs，因此在 SLES 上不强制要求单独的块设备，但你可以出于性能考虑选择独立设备。

- 内核需支持 `btrfs`。运行以下命令检查：

  ```console
  $ grep btrfs /proc/filesystems

  btrfs
  ```

- 在操作系统层面管理 Btrfs 文件系统需要 `btrfs` 命令。如未安装，请在 SLES 上安装 `btrfsprogs`，在 Ubuntu 上安装 `btrfs-tools`。

## 配置 Docker 使用 btrfs 存储驱动

以下步骤在 SLES 与 Ubuntu 上基本一致。

1. Stop Docker.

2. Copy the contents of `/var/lib/docker/` to a backup location, then empty
   the contents of `/var/lib/docker/`:

   ```console
   $ sudo cp -au /var/lib/docker /var/lib/docker.bk
   $ sudo rm -rf /var/lib/docker/*
   ```

3. 将专用块设备格式化为 Btrfs 文件系统。以下示例假设使用两块设备 `/dev/xvdf` 与 `/dev/xvdg`。务必确认设备名，因为该操作具有破坏性。

   ```console
   $ sudo mkfs.btrfs -f /dev/xvdf /dev/xvdg
   ```

   Btrfs 还有更多可选项（如条带化与 RAID）。参见 [Btrfs 文档](https://btrfs.wiki.kernel.org/index.php/Using_Btrfs_with_Multiple_Devices)。

4. 将新建的 Btrfs 文件系统挂载到 `/var/lib/docker/`。可以指定创建该 Btrfs 文件系统时使用的任意块设备。

   ```console
   $ sudo mount -t btrfs /dev/xvdf /var/lib/docker
   ```

   > [!NOTE]
   >
   > 为了在重启后保持挂载生效，请在 `/etc/fstab` 中添加相应条目。

5. 将 `/var/lib/docker.bk` 的内容拷贝回 `/var/lib/docker/`。

   ```console
   $ sudo cp -au /var/lib/docker.bk/* /var/lib/docker/
   ```

6. 配置 Docker 使用 `btrfs` 存储驱动。即使 `/var/lib/docker/` 已是 Btrfs 文件系统，仍需显式设置。编辑或创建 `/etc/docker/daemon.json`；若为新文件，添加以下内容；若已存在，仅需添加相应键值，注意在非最后一行结尾处添加逗号：

   ```json
   {
     "storage-driver": "btrfs"
   }
   ```

   各存储驱动支持的所有存储选项，参见[守护进程参考文档](/reference/cli/dockerd/#options-per-storage-driver)

7. 启动 Docker。运行后，验证当前存储驱动为 `btrfs`：

   ```console
   $ docker info

   Containers: 0
    Running: 0
    Paused: 0
    Stopped: 0
   Images: 0
   Server Version: 17.03.1-ce
   Storage Driver: btrfs
    Build Version: Btrfs v4.4
    Library Version: 101
   <...>
   ```

8. 确认无误后，可以删除 `/var/lib/docker.bk` 目录。

## 管理 Btrfs 卷

Btrfs 的优势之一在于无需卸载文件系统或重启 Docker，即可轻松管理 Btrfs 文件系统。

当空间不足时，Btrfs 会以约 1 GB 的块大小自动扩展卷。

要向 Btrfs 卷新增块设备，可使用 `btrfs device add` 与 `btrfs filesystem balance`：

```console
$ sudo btrfs device add /dev/svdh /var/lib/docker

$ sudo btrfs filesystem balance /var/lib/docker
```

> [!NOTE]
>
   > 这些操作可以在 Docker 运行时执行，但会影响性能。建议在维护窗口内进行卷平衡。

## `btrfs` 存储驱动的工作原理

与其他存储驱动不同，`btrfs` 会将整个 `/var/lib/docker/` 存放在一个 Btrfs 卷上。

### 镜像与容器层的磁盘布局

镜像层与容器可写层的相关信息位于 `/var/lib/docker/btrfs/subvolumes/`。该目录下每个镜像层或容器层对应一个子目录；统一文件系统由该层及其所有父层组合而成。子卷（subvolume）原生支持写时复制，按需从底层存储池分配空间，且可以嵌套与快照。下图展示了 4 个子卷，其中“子卷 2”和“子卷 3”为嵌套关系，“子卷 4”展示了自身的目录树。

![Subvolume example](images/btfs_subvolume.webp?w=350&h=100)

只有镜像的基底层会作为真正的子卷存储，其余各层都以快照形式保存，仅包含该层引入的差异。你也可以基于快照再创建快照，如下图所示。

![Snapshots diagram](images/btfs_snapshots.webp?w=350&h=100)

在磁盘上，快照看起来与子卷很相似，但实际上更小且更节省空间。系统通过写时复制最大化存储效率、最小化各层体积；容器可写层中的写入在块级被管理。下图展示了子卷与其快照共享数据的关系。

![Snapshot and subvolume sharing data](images/btfs_pool.webp?w=450&h=200)

为获得更高效率，容器在需要更多空间时会以约 1 GB 的块大小进行分配。

`btrfs` 存储驱动会将每个镜像层与容器分别存放在独立的 Btrfs 子卷或快照中：镜像基底层为子卷，子层镜像与容器为快照，如下图所示。

![Btrfs container layers](images/btfs_container_layer.webp?w=600)

在使用 `btrfs` 驱动的宿主机上，创建镜像与容器的大致流程如下：

1. 镜像的基底层存放在 `/var/lib/docker/btrfs/subvolumes` 下的 Btrfs 子卷中。

2. 后续镜像层以父层子卷或快照的 Btrfs 快照形式保存，并包含该层引入的更改。这些差异在块级进行存储。

3. 容器的可写层是最终镜像层的 Btrfs 快照，包含容器运行期间产生的差异，亦在块级存储。

## 使用 `btrfs` 时容器的读写机制

### 读取文件

容器本质上是镜像的一个高效快照。快照中的元数据会指向存储池中的实际数据块，这与子卷的行为一致。因此，从快照读取与从子卷读取在本质上相同。

### 写入文件

需注意，在 Btrfs 上大量写入或更新小文件可能导致性能下降。

以下示例说明容器在 Btrfs 上以写方式打开文件的三种情形：

#### 写入新文件

向容器写入新文件会触发按需分配，为容器快照分配新的数据块，随后将文件写入该空间。按需分配是 Btrfs 的默认写入机制，与向子卷写入新数据的方式一致。因此，向容器快照写入新文件可达到 Btrfs 的原生速度。

#### 修改已有文件

在容器中更新已有文件会触发写时复制（Btrfs 称为重定向写入 redirect-on-write）。系统会从当前存放该文件的层读取原始数据，仅将修改后的块写入容器可写层，然后更新快照中的文件系统元数据指向新数据。该过程会产生少量开销。

#### 删除文件或目录

当容器删除位于更低层的文件或目录时，Btrfs 会对其进行“遮蔽”，使之不可见。如果容器创建了文件后又删除，则该操作发生在 Btrfs 文件系统本身，空间将被回收。

## Btrfs 与 Docker 性能

在 `btrfs` 存储驱动下，有多种因素会影响 Docker 的性能。

> [!NOTE]
>
> 对于写入密集型工作负载，将数据放入 Docker 卷而非容器可写层，能在很大程度上缓解这些问题。但在 Btrfs 场景下，只有当 `/var/lib/docker/volumes/` 不由 Btrfs 承载时，卷才不会受到这些影响。

### 页面缓存（Page cache）

Btrfs 不支持页面缓存共享。这意味着每个访问同一文件的进程都会在宿主机内存中各自缓存一份拷贝。因此，在 PaaS 等高密度场景中，`btrfs` 可能不是最佳选择。

### 小块写入

容器执行大量小写入（例如在短时间内频繁启动、停止大量容器时也会出现类似模式）会导致 Btrfs 块使用效率不佳，可能让 Btrfs 文件系统过早耗尽空间，造成宿主机磁盘不足。请使用 `btrfs filesys show` 密切监控 Btrfs 设备的可用空间。

### 顺序写入

Btrfs 在落盘时使用日志技术，这会影响顺序写入的性能，最高可能下降约 50%。

### 碎片化

碎片化是 Btrfs 等写时复制文件系统的副产物。大量小的随机写会加剧这一问题：在 SSD 上表现为 CPU 峰值，在机械硬盘上表现为磁头频繁寻道；二者都会损害性能。

如果你的 Linux 内核版本不低于 3.9，可以在挂载 Btrfs 卷时启用 `autodefrag`。在投入生产前，请在你的实际负载上验证，因为部分测试显示其可能对性能产生负面影响。

### SSD 性能

Btrfs 针对 SSD 提供了原生优化。可在挂载时添加 `-o ssd` 选项启用。这些优化会绕过不适用于固态介质的“寻道优化”等策略，以提升写入性能。

### 定期平衡（balance）Btrfs 文件系统

建议使用操作系统计划任务（如 `cron`）在业务低峰期定期执行 Btrfs 文件系统的平衡操作，以回收未分配块并防止无谓地占满文件系统。注意：当 Btrfs 文件系统已被完全写满时，若不增加新的物理块设备，将无法进行再平衡。

可参考 [Btrfs Wiki](https://btrfs.wiki.kernel.org/index.php/Balance_Filters#Balancing_to_fix_filesystem_full_errors)。

### 使用更快的存储介质

固态硬盘（SSD）通常比机械硬盘提供更快的读写性能。

### 写入密集型工作负载优先使用卷

对于写入密集型工作负载，卷提供了最佳且更可预测的性能。这是因为卷绕过了存储驱动，避免了精简配置与写时复制带来的潜在开销。卷还具备其他优势，例如：支持在容器间共享数据，并在没有容器使用时依然持久存在。

## 相关信息

- [卷](../volumes.md)
- [理解镜像、容器与存储驱动](index.md)
- [选择存储驱动](select-storage-driver.md)
