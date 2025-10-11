---
description: 了解如何更高效地使用 Device Mapper 存储驱动。
keywords: 容器, 存储, 驱动, device mapper
title: Device Mapper 存储驱动（已废弃）
aliases:
  - /storage/storagedriver/device-mapper-driver/
---

> **已废弃**
>
> Device Mapper 驱动[已被废弃](/manuals/engine/deprecated.md#device-mapper-storage-driver)，并在 Docker Engine v25.0 中移除。如果你正在使用 Device Mapper，必须在升级到 Docker Engine v25.0 之前迁移到受支持的存储驱动。请参阅[存储驱动选择](select-storage-driver.md)了解受支持的驱动。

Device Mapper 是 Linux 内核中的一个框架，为众多高级卷管理技术提供基础。Docker 的 `devicemapper` 存储驱动利用该框架的精简配置与快照能力来管理镜像与容器。本文将存储驱动称为 `devicemapper`，内核框架称为 Device Mapper。

在受支持的系统上，Linux 内核包含对 `devicemapper` 的支持。不过，将其用于 Docker 仍需要特定配置。

`devicemapper` 驱动使用 Docker 专用的块设备，并在块级而非文件级工作。可通过为宿主机增加物理存储来扩展这些设备；与操作系统层面的文件系统相比，这种方式通常性能更好。

## 先决条件

- `devicemapper` 在以下系统上的 Docker Engine - Community 受支持：CentOS、Fedora、SLES 15、Ubuntu、Debian、RHEL。
- 需要安装 `lvm2` 与 `device-mapper-persistent-data` 软件包。
- 更换存储驱动会导致本机已创建的容器不可访问。请使用 `docker save` 备份容器，并将现有镜像推送到 Docker Hub 或私有仓库，以免之后需要重新创建。

## Configure Docker with the `devicemapper` storage driver

Before following these procedures, you must first meet all the
[prerequisites](#prerequisites).

### 配置 `loop-lvm` 测试模式

该配置仅适用于测试。`loop-lvm` 模式采用回环（loopback）机制，使本地磁盘上的文件可被当作真实的物理磁盘或块设备来读写。但回环机制与操作系统文件系统层的交互会让 IO 变慢且资源开销更大，并可能引入竞态。尽管如此，先配置 `loop-lvm` 有助于在启用更复杂的 `direct-lvm` 之前，提前发现基础问题（缺少用户态软件包、内核驱动等）。因此，`loop-lvm` 只应用于配置 `direct-lvm` 前的初步测试。

生产环境请参考[配置生产可用的 direct-lvm 模式](#configure-direct-lvm-mode-for-production)。

1. 停止 Docker。

   ```console
   $ sudo systemctl stop docker
   ```

2.  编辑 `/etc/docker/daemon.json`（若不存在则创建）。若该文件此前为空，添加如下内容：

    ```json
    {
      "storage-driver": "devicemapper"
    }
    ```

    各存储驱动支持的所有存储选项，参见[守护进程参考文档](/reference/cli/dockerd/#options-per-storage-driver)

    如果 `daemon.json` 中 JSON 格式不正确，Docker 将无法启动。

3.  启动 Docker。

    ```console
    $ sudo systemctl start docker
    ```

4.  使用 `docker info` 并查看 `Storage Driver`，确认守护进程正在使用 `devicemapper`。

    ```console
    $ docker info

      Containers: 0
        Running: 0
        Paused: 0
        Stopped: 0
      Images: 0
      Server Version: 17.03.1-ce
      Storage Driver: devicemapper
      Pool Name: docker-202:1-8413957-pool
      Pool Blocksize: 65.54 kB
      Base Device Size: 10.74 GB
      Backing Filesystem: xfs
      Data file: /dev/loop0
      Metadata file: /dev/loop1
      Data Space Used: 11.8 MB
      Data Space Total: 107.4 GB
      Data Space Available: 7.44 GB
      Metadata Space Used: 581.6 KB
      Metadata Space Total: 2.147 GB
      Metadata Space Available: 2.147 GB
      Thin Pool Minimum Free Space: 10.74 GB
      Udev Sync Supported: true
      Deferred Removal Enabled: false
      Deferred Deletion Enabled: false
      Deferred Deleted Device Count: 0
      Data loop file: /var/lib/docker/devicemapper/data
      Metadata loop file: /var/lib/docker/devicemapper/metadata
      Library Version: 1.02.135-RHEL7 (2016-11-16)
    <...>
    ```

  以上输出表明该主机运行在 `loop-lvm` 模式下，该模式在生产环境中不受支持。可以从 `Data loop file` 与 `Metadata loop file` 位于 `/var/lib/docker/devicemapper` 下的稀疏文件判断。生产环境请参考[配置生产可用的 direct-lvm 模式](#configure-direct-lvm-mode-for-production)。


### 配置生产可用的 direct-lvm 模式

在生产环境中使用 `devicemapper` 存储驱动的主机必须启用 `direct-lvm` 模式。该模式通过块设备创建精简池，相比回环设备更快、资源利用更高，且块设备可按需扩展，但配置也更复杂。

满足[先决条件](#先决条件)后，按以下步骤将 Docker 配置为在 `direct-lvm` 模式下使用 `devicemapper` 存储驱动。

> [!WARNING]
> Changing the storage driver makes any containers you have already
> created inaccessible on the local system. Use `docker save` to save containers,
> and push existing images to Docker Hub or a private repository, so you do not
> need to recreate them later.

#### 允许 Docker 自动配置 direct-lvm 模式

Docker 可以代你管理块设备，简化 `direct-lvm` 的配置。**仅适用于全新 Docker 环境。** 此方式只能使用单一块设备；若需多个块设备，请[手动配置 direct-lvm](#configure-direct-lvm-mode-manually)。可用的新配置项如下：

| Option                          | Description                                                                                                                                                                        | Required? | Default | Example                            |
|:--------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------|:--------|:-----------------------------------|
| `dm.directlvm_device`           | The path to the block device to configure for `direct-lvm`.                                                                                                                        | Yes       |         | `dm.directlvm_device="/dev/xvdf"`  |
| `dm.thinp_percent`              | The percentage of space to use for storage from the passed in block device.                                                                                                        | No        | 95      | `dm.thinp_percent=95`              |
| `dm.thinp_metapercent`          | The percentage of space to use for metadata storage from the passed-in block device.                                                                                                   | No        | 1       | `dm.thinp_metapercent=1`           |
| `dm.thinp_autoextend_threshold` | The threshold for when lvm should automatically extend the thin pool as a percentage of the total storage space.                                                                   | No        | 80      | `dm.thinp_autoextend_threshold=80` |
| `dm.thinp_autoextend_percent`   | The percentage to increase the thin pool by when an autoextend is triggered.                                                                                                       | No        | 20      | `dm.thinp_autoextend_percent=20`   |
| `dm.directlvm_device_force`     | Whether to format the block device even if a filesystem already exists on it. If set to `false` and a filesystem is present, an error is logged and the filesystem is left intact. | No        | false   | `dm.directlvm_device_force=true`   |

编辑 `daemon.json` 并设置相应选项，随后重启 Docker 使之生效。以下 `daemon.json` 示例配置了上表中的全部选项：

```json
{
  "storage-driver": "devicemapper",
  "storage-opts": [
    "dm.directlvm_device=/dev/xdf",
    "dm.thinp_percent=95",
    "dm.thinp_metapercent=1",
    "dm.thinp_autoextend_threshold=80",
    "dm.thinp_autoextend_percent=20",
    "dm.directlvm_device_force=false"
  ]
}
```

各存储驱动支持的所有存储选项，参见[守护进程参考文档](/reference/cli/dockerd/#options-per-storage-driver)

重启 Docker 以使更改生效。Docker 将自动执行命令为你配置块设备。

> [!WARNING]
> Changing these values after Docker has prepared the block device for you is
> not supported and causes an error.

你仍需[执行周期性维护任务](#manage-devicemapper)。

#### 手动配置 direct-lvm 模式

以下步骤将创建一个配置为精简池（thin pool）的逻辑卷，作为存储池的后端。假设你有一块空闲块设备（例如 `/dev/xvdf`），容量足以完成任务。不同环境中的设备标识与卷大小可能不同，请在执行过程中替换为你的实际值。本步骤同样假定 Docker 守护进程处于 `stopped` 状态。

1.  确认要使用的块设备。设备位于 `/dev/`（如 `/dev/xvdf`），需要有足够空间存放该主机工作负载所需的镜像与容器层。优先选择 SSD。

2.  停止 Docker。

    ```console
    $ sudo systemctl stop docker
    ```

3.  安装以下软件包：

    - **RHEL / CentOS**：`device-mapper-persistent-data`、`lvm2` 及全部依赖

    - **Ubuntu / Debian / SLES 15**：`thin-provisioning-tools`、`lvm2` 及全部依赖

4.  在第 1 步选择的块设备上使用 `pvcreate` 创建物理卷。请将 `/dev/xvdf` 替换为你的设备名。

    > [!WARNING]
    > 接下来的若干步骤具有破坏性，请务必确认设备无误。

    ```console
    $ sudo pvcreate /dev/xvdf

    Physical volume "/dev/xvdf" successfully created.
    ```

5.  使用 `vgcreate` 在同一设备上创建名为 `docker` 的卷组。

    ```console
    $ sudo vgcreate docker /dev/xvdf

    Volume group "docker" successfully created
    ```

6.  使用 `lvcreate` 创建名为 `thinpool` 与 `thinpoolmeta` 的两个逻辑卷。最后一个参数用于预留空闲空间，以便在空间不足时临时自动扩展数据或元数据。以下为推荐值。

    ```console
    $ sudo lvcreate --wipesignatures y -n thinpool docker -l 95%VG

    Logical volume "thinpool" created.

    $ sudo lvcreate --wipesignatures y -n thinpoolmeta docker -l 1%VG

    Logical volume "thinpoolmeta" created.
    ```

7.  使用 `lvconvert` 将上述卷分别转换为精简池及其元数据存储位置。

    ```console
    $ sudo lvconvert -y \
    --zero n \
    -c 512K \
    --thinpool docker/thinpool \
    --poolmetadata docker/thinpoolmeta

    WARNING: Converting logical volume docker/thinpool and docker/thinpoolmeta to
    thin pool's data and metadata volumes with metadata wiping.
    THIS WILL DESTROY CONTENT OF LOGICAL VOLUME (filesystem etc.)
    Converted docker/thinpool to thin pool.
    ```

8.  通过 `lvm` 配置文件启用精简池自动扩展。

    ```console
    $ sudo vi /etc/lvm/profile/docker-thinpool.profile
    ```

9.  设置 `thin_pool_autoextend_threshold` 与 `thin_pool_autoextend_percent` 的取值。

    `thin_pool_autoextend_threshold` 表示触发 `lvm` 自动扩展前的空间使用百分比（100 = 禁用，不推荐）。

    `thin_pool_autoextend_percent` 表示自动扩展时增加的空间百分比（0 = 禁用）。

    例如：当磁盘使用达到 80% 时，自动增加 20% 的容量。

    ```text
    activation {
      thin_pool_autoextend_threshold=80
      thin_pool_autoextend_percent=20
    }
    ```

    保存文件。

10. 使用 `lvchange` 应用该 LVM 配置文件。

    ```console
    $ sudo lvchange --metadataprofile docker-thinpool docker/thinpool

    Logical volume docker/thinpool changed.
    ```

11. 确认逻辑卷监控已启用。

    ```console
    $ sudo lvs -o+seg_monitor

    LV       VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Monitor
    thinpool docker twi-a-t--- 95.00g             0.00   0.01                             not monitored
    ```

    若 `Monitor` 列显示为 `not monitored`，则需要显式启用监控。否则，无论配置文件如何设置，逻辑卷都不会自动扩展。

    ```console
    $ sudo lvchange --monitor y docker/thinpool
    ```

    再次执行 `sudo lvs -o+seg_monitor` 确认监控已启用；`Monitor` 列应显示 `monitored`。

12. 如果此前在该主机运行过 Docker，或存在 `/var/lib/docker/`，请先迁移该目录，以便 Docker 使用新的 LVM 池存放镜像与容器内容。

    ```console
    $ sudo su -
    # mkdir /var/lib/docker.bk
    # mv /var/lib/docker/* /var/lib/docker.bk
    # exit
    ```

    如后续步骤失败需要回滚，可删除 `/var/lib/docker` 并用 `/var/lib/docker.bk` 还原。

13. 编辑 `/etc/docker/daemon.json`，配置 `devicemapper` 存储驱动所需选项。若该文件此前为空，应包含如下内容：

    ```json
    {
        "storage-driver": "devicemapper",
        "storage-opts": [
        "dm.thinpooldev=/dev/mapper/docker-thinpool",
        "dm.use_deferred_removal=true",
        "dm.use_deferred_deletion=true"
        ]
    }
    ```

14. 启动 Docker。

    **systemd**:

    ```console
    $ sudo systemctl start docker
    ```

    **service**:

    ```console
    $ sudo service docker start
    ```

15. 使用 `docker info` 验证 Docker 已使用新配置。

    ```console
    $ docker info

    Containers: 0
     Running: 0
     Paused: 0
     Stopped: 0
    Images: 0
    Server Version: 17.03.1-ce
    Storage Driver: devicemapper
     Pool Name: docker-thinpool
     Pool Blocksize: 524.3 kB
     Base Device Size: 10.74 GB
     Backing Filesystem: xfs
     Data file:
     Metadata file:
     Data Space Used: 19.92 MB
     Data Space Total: 102 GB
     Data Space Available: 102 GB
     Metadata Space Used: 147.5 kB
     Metadata Space Total: 1.07 GB
     Metadata Space Available: 1.069 GB
     Thin Pool Minimum Free Space: 10.2 GB
     Udev Sync Supported: true
     Deferred Removal Enabled: true
     Deferred Deletion Enabled: true
     Deferred Deleted Device Count: 0
     Library Version: 1.02.135-RHEL7 (2016-11-16)
    <...>
    ```

    若配置正确，`Data file` 与 `Metadata file` 应为空，池名称应为 `docker-thinpool`。

16. 确认配置无误后，可以删除保存旧配置的 `/var/lib/docker.bk` 目录。

    ```console
    $ sudo rm -rf /var/lib/docker.bk
    ```

## 管理 devicemapper

### 监控精简池（thin pool）

不要仅依赖 LVM 的自动扩展。卷组会自动扩容，但卷仍可能写满。你可以使用 `lvs` 或 `lvs -a` 监控卷的可用空间，并考虑在操作系统层使用 Nagios 等监控工具。

查看 LVM 日志可使用 `journalctl`：

```console
$ sudo journalctl -fu dm-event.service
```

如果经常遇到精简池问题，可以在 `/etc/docker/daemon.json` 中设置存储选项 `dm.min_free_space`（百分比）。例如设置为 `10`，当可用空间接近或达到 10% 时，相关操作将带警告地失败。参见[守护进程存储驱动选项](/reference/cli/dockerd/#daemon-storage-driver)。

### 在线扩容设备容量

你可以在设备运行时增加精简池容量。当数据逻辑卷已满且卷组容量达到上限时，这非常有用。具体流程取决于你使用的是[loop-lvm 精简池](#resize-a-loop-lvm-thin-pool)还是[direct-lvm 精简池](#resize-a-direct-lvm-thin-pool)。

#### 调整 loop-lvm 精简池大小

最简单的方式是[使用 device_tool 工具](#use-the-device_tool-utility)调整 `loop-lvm` 精简池大小，也可以[使用操作系统工具](#use-operating-system-utilities)。

##### 使用 device_tool 工具

社区提供了脚本 `device_tool.go`（见 [moby/moby](https://github.com/moby/moby/tree/master/contrib/docker-device-tool) 仓库）。可用它来调整 `loop-lvm` 精简池，以避免上面的冗长流程。该工具不保证一定有效；无论如何，`loop-lvm` 仅应用于非生产系统。

如果不想使用 `device_tool`，可以改为[手动调整精简池](#use-operating-system-utilities)。

1.  To use the tool, clone the Github repository, change to the
    `contrib/docker-device-tool`, and follow the instructions in the `README.md`
    to compile the tool.

2.  Use the tool. The following example resizes the thin pool to 200GB.

    ```console
    $ ./device_tool resize 200GB
    ```

##### 使用操作系统工具

如果不想[使用 device_tool 工具](#use-the-device_tool-utility)，可以按以下步骤手动调整 `loop-lvm` 精简池大小。

在 `loop-lvm` 模式中，一个回环设备用于存储数据，另一个用于存储元数据。由于存在明显的性能与稳定性问题，`loop-lvm` 仅用于测试。

若使用 `loop-lvm`，`docker info` 输出会显示 `Data loop file` 与 `Metadata loop file` 的路径：

```console
$ docker info |grep 'loop file'

 Data loop file: /var/lib/docker/devicemapper/data
 Metadata loop file: /var/lib/docker/devicemapper/metadata
```

按以下步骤增加精简池大小。本例将 100 GB 扩容至 200 GB：

1.  列出设备文件大小。

    ```console
    $ sudo ls -lh /var/lib/docker/devicemapper/

    total 1175492
    -rw------- 1 root root 100G Mar 30 05:22 data
    -rw------- 1 root root 2.0G Mar 31 11:17 metadata
    ```

2.  使用 `truncate` 将 `data` 文件调整至 200G。该命令既可增大也可减小文件大小；注意减小大小是破坏性操作。

    ```console
    $ sudo truncate -s 200G /var/lib/docker/devicemapper/data
    ```

3.  验证文件大小已变更。

    ```console
    $ sudo ls -lh /var/lib/docker/devicemapper/

    total 1.2G
    -rw------- 1 root root 200G Apr 14 08:47 data
    -rw------- 1 root root 2.0G Apr 19 13:27 metadata
    ```

4.  回环文件在磁盘上已改变，但内存中的回环设备尚未刷新。先查看内存中设备大小（GB），然后重新加载并再查看；重载后应为 200 GB。

    ```console
    $ echo $[ $(sudo blockdev --getsize64 /dev/loop0) / 1024 / 1024 / 1024 ]

    100

    $ sudo losetup -c /dev/loop0

    $ echo $[ $(sudo blockdev --getsize64 /dev/loop0) / 1024 / 1024 / 1024 ]

    200
    ```

5.  重新加载 devicemapper 精简池。

    a. 首先获取池名。它是输出的第一个字段，以 `:` 分隔。以下命令可提取：

    ```console
    $ sudo dmsetup status | grep ' thin-pool ' | awk -F ': ' {'print $1'}
    docker-8:1-123141-pool
    ```

    b. 导出该精简池的 device-mapper 表：

    ```console
    $ sudo dmsetup table docker-8:1-123141-pool
    0 209715200 thin-pool 7:1 7:0 128 32768 1 skip_block_zeroing
    ```

    c. 使用输出的第二个字段计算精简池的总扇区数。该数值以 512K 扇区为单位。100G 文件对应 209715200 个 512K 扇区；扩到 200G 则为 419430400。

    d. 使用以下三个 `dmsetup` 命令带入新的扇区数，重新加载精简池：

    ```console
    $ sudo dmsetup suspend docker-8:1-123141-pool
    $ sudo dmsetup reload docker-8:1-123141-pool --table '0 419430400 thin-pool 7:1 7:0 128 32768 1 skip_block_zeroing'
    $ sudo dmsetup resume docker-8:1-123141-pool
    ```

#### 调整 direct-lvm 精简池大小

扩展 `direct-lvm` 精简池前，需要先向宿主机添加新的块设备，并记录内核分配的设备名。本例中新块设备为 `/dev/xvdg`。

按以下步骤扩展 `direct-lvm` 精简池；请根据实际替换块设备名与其他参数。

1.  收集卷组信息。

    使用 `pvdisplay` 查找当前被精简池使用的物理块设备，以及卷组名。

    ```console
    $ sudo pvdisplay |grep 'VG Name'

    PV Name               /dev/xvdf
    VG Name               docker
    ```

    在后续步骤中按需替换你的块设备或卷组名。

2.  使用上一步的 `VG Name` 与你的“新”块设备，通过 `vgextend` 扩展卷组。

    ```console
    $ sudo vgextend docker /dev/xvdg

    Physical volume "/dev/xvdg" successfully created.
    Volume group "docker" successfully extended
    ```

3.  扩展 `docker/thinpool` 逻辑卷。该命令会立即使用卷组 100% 的空闲空间，不启用自动扩展。若要扩展元数据池，请使用 `docker/thinpool_tmeta`。

    ```console
    $ sudo lvextend -l+100%FREE -n docker/thinpool

    Size of logical volume docker/thinpool_tdata changed from 95.00 GiB (24319 extents) to 198.00 GiB (50688 extents).
    Logical volume docker/thinpool_tdata successfully resized.
    ```

4.  通过 `docker info` 输出中的 `Data Space Available` 验证新容量；若扩展的是 `docker/thinpool_tmeta`，查看 `Metadata Space Available`。

    ```bash
    Storage Driver: devicemapper
     Pool Name: docker-thinpool
     Pool Blocksize: 524.3 kB
     Base Device Size: 10.74 GB
     Backing Filesystem: xfs
     Data file:
     Metadata file:
     Data Space Used: 212.3 MB
     Data Space Total: 212.6 GB
     Data Space Available: 212.4 GB
     Metadata Space Used: 286.7 kB
     Metadata Space Total: 1.07 GB
     Metadata Space Available: 1.069 GB
    <...>
    ```

### 重启后激活 `devicemapper`

如果重启主机后发现 `docker` 服务无法启动，并出现 “Non existing device” 错误，请使用以下命令重新激活逻辑卷：

```console
$ sudo lvchange -ay docker/thinpool
```

## `devicemapper` 存储驱动的工作原理

> [!WARNING]
> 请勿直接操作 `/var/lib/docker/` 下的任何文件或目录，这些均由 Docker 管理。

可使用 `lsblk` 从操作系统视角查看设备与其所属的池：

```console
$ sudo lsblk

NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda                    202:0    0    8G  0 disk
└─xvda1                 202:1    0    8G  0 part /
xvdf                    202:80   0  100G  0 disk
├─docker-thinpool_tmeta 253:0    0 1020M  0 lvm
│ └─docker-thinpool     253:2    0   95G  0 lvm
└─docker-thinpool_tdata 253:1    0   95G  0 lvm
  └─docker-thinpool     253:2    0   95G  0 lvm
```

使用 `mount` 查看 Docker 所使用的挂载点：

```console
$ mount |grep devicemapper
/dev/xvda1 on /var/lib/docker/devicemapper type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```

使用 `devicemapper` 时，Docker 将镜像与层的内容存放于精简池，并通过挂载到 `/var/lib/docker/devicemapper/` 的子目录的方式暴露给容器。

### 镜像与容器层的磁盘布局

`/var/lib/docker/devicemapper/metadata/` 目录包含 Devicemapper 自身配置以及每个镜像/容器层的元数据。`devicemapper` 使用快照机制，这些元数据也包含与快照相关的信息。文件为 JSON 格式。

`/var/lib/docker/devicemapper/mnt/` 目录为每个镜像层与容器层提供挂载点。镜像层的挂载点为空目录；容器的挂载点展示容器视角下的文件系统。


### 镜像分层与共享

`devicemapper` 使用专用块设备而非格式化文件系统，在写时复制（CoW）过程中以块级操作来实现更高性能。

#### 快照

`devicemapper` 的另一特点是使用快照（也称“精简设备 thin devices”或“虚拟设备”），以小而轻量的精简池存储每层引入的差异。快照带来诸多好处：

- 容器之间共享的层在磁盘上只存储一份（除非该层可写）。例如有 10 个都基于 `alpine` 的镜像，则 `alpine` 及其所有父镜像在磁盘上各只存储一次。

- 快照是写时复制（CoW）的具体实现。只有当容器修改或删除某文件/目录时，才会把该内容复制到容器的可写层。

- 由于 `devicemapper` 在块级工作，可写层中的多个块可同时被修改。

- 可使用操作系统级备份工具来备份快照：复制 `/var/lib/docker/devicemapper/` 即可。

#### Devicemapper 工作流

当以 `devicemapper` 存储驱动启动 Docker 时，与镜像层和容器层相关的所有对象都存放在 `/var/lib/docker/devicemapper/` 下，其后端由一个或多个块设备提供（回环设备仅用于测试，或物理磁盘）。

- 基础设备（base device）是最底层对象，即精简池本身。可通过 `docker info` 查看。它包含文件系统，是每个镜像层和容器层的起点。基础设备是 Device Mapper 的实现细节，而非 Docker 层。

- 基础设备及各镜像/容器层的元数据以 JSON 格式存于 `/var/lib/docker/devicemapper/metadata/`。这些层是写时复制快照，在与父层发生分歧之前保持为空。

- 每个容器的可写层挂载在 `/var/lib/docker/devicemapper/mnt/` 下的一个挂载点。每个只读镜像层与每个已停止的容器都有一个空目录。

每个镜像层都是其下一层的快照。每个镜像的最底层是池中基础设备的一个快照。运行容器时，容器本身就是其所依赖镜像的一个快照。下例展示了一台主机上两个运行中的容器：一个 `ubuntu`，一个 `busybox`。

![Ubuntu and busybox image layers](images/two_dm_container.webp?w=450&h=100)

## 使用 `devicemapper` 时容器的读写机制

### 读取文件

在 `devicemapper` 下，读取发生在块级。下图展示了示例容器读取单个块（`0x44f`）的高层过程。

![Reading a block with devicemapper](images/dm_container.webp?w=650)

应用请求读取容器中的块 `0x44f`。由于容器是镜像的精简快照，它本身不包含该块，但持有指向最近父镜像中该块的指针，于是从父镜像读取该块，并将其载入容器内存。

### 写入文件

**写入新文件**：使用 `devicemapper` 时，向容器写入新数据会通过“按需分配”完成。新文件的每个块都会在容器可写层中分配并写入。

**更新已有文件**：相关块从最近存在该文件的层读取；写入时仅将被修改的块写入容器可写层。

**删除文件或目录**：当在容器可写层删除文件或目录，或镜像层删除其父层中的文件时，`devicemapper` 会拦截对此路径的后续读取，并返回该文件或目录不存在。

**写入后再删除文件**：若容器写入某文件后又删除，所有操作都发生在可写层。此时使用 `direct-lvm` 会释放相应块；`loop-lvm` 下可能不会释放。这也是不建议在生产中使用 `loop-lvm` 的又一原因。

## Device Mapper 与 Docker 性能

- **`allocate-on demand` 的性能影响**：

  `devicemapper` 使用“按需分配”从精简池为容器可写层分配新块。每个块为 64KB，这是单次写入的最小占用单位。

- **写时复制的性能影响**：容器首次修改某个块时，该块会写入可写层。由于操作发生在块级而非文件级，性能影响被尽量降低。但当需要写入大量块时，性能仍可能受影响，某些场景下 `devicemapper` 甚至不如其他驱动。对于写入密集型负载，应使用数据卷以完全绕过存储驱动。

### 性能最佳实践

在使用 `devicemapper` 时，请注意以下建议以获得最佳性能：

- **使用 `direct-lvm`**：`loop-lvm` 性能较差，切勿在生产使用。

- **使用更快的存储**：SSD 通常比机械硬盘有更好的读写性能。

- **关注内存占用**：`devicemapper` 可能比其他驱动占用更多内存。每个容器会将其文件的一个或多个副本加载到内存，这取决于同时被修改的块数量。在高密度场景中，这可能不是最佳选择。

- **写入密集型负载使用卷**：卷能提供更好且更可预测的性能，因为它们绕过存储驱动，避免精简配置与写时复制带来的开销。卷还支持跨容器共享数据，并在没有容器使用时仍可持久存在。

  > [!NOTE]
  >
  > 当使用 `devicemapper` 搭配 `json-file` 日志驱动时，容器生成的日志仍默认存放在 Docker 数据根目录（默认 `/var/lib/docker`）。若日志量很大，可能导致磁盘占用飙升甚至磁盘写满，影响系统管理。你可以配置[日志驱动](/manuals/engine/logging/configure.md)将容器日志写到外部存储。

## 相关信息

- [卷](../volumes.md)
- [理解镜像、容器与存储驱动](./_index.md)
- [选择存储驱动](select-storage-driver.md)
