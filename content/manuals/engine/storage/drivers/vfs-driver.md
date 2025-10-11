---
description: 了解如何更高效地使用 VFS 存储驱动。
keywords: 容器, 存储, 驱动, vfs
title: VFS 存储驱动
aliases:
  - /storage/storagedriver/vfs-driver/
---

VFS 存储驱动并非联合文件系统。每一层都对应磁盘上的一个目录，且不支持写时复制（copy-on-write）。当创建新层时，会对上一层执行“深拷贝”。与其他存储驱动相比，这通常带来更低的性能与更高的磁盘占用。但它足够健壮、稳定，几乎在任何环境都能工作；在测试环境中，它也常被用来作为对比基准，验证其他后端存储的行为。

## 配置 Docker 使用 `vfs` 存储驱动

1. 停止 Docker。

   ```console
   $ sudo systemctl stop docker
   ```

2.  编辑 `/etc/docker/daemon.json`（若不存在则创建）。若文件为空，添加如下内容：

    ```json
    {
      "storage-driver": "vfs"
    }
    ```

    如需为 VFS 存储驱动设置配额以限制其最大使用空间，可在 `storage-opts` 中设置 `size` 选项：

    ```json
    {
      "storage-driver": "vfs",
      "storage-opts": ["size=256M"]
    }
    ```

    如果 `daemon.json` 包含无效 JSON，Docker 将无法启动。

3.  启动 Docker。

    ```console
    $ sudo systemctl start docker
    ```

4.  使用 `docker info` 并查看 `Storage Driver`，确认守护进程正在使用 `vfs` 存储驱动。

    ```console
    $ docker info

    Storage Driver: vfs
    ...
    ```

至此，Docker 将使用 `vfs` 存储驱动。Docker 会自动创建 `/var/lib/docker/vfs/` 目录，用于存放运行中容器所使用的所有层。

## `vfs` 存储驱动的工作原理

每个镜像层与容器的可写层，都会以 `/var/lib/docker/` 下的子目录形式存在于宿主机上。联合挂载提供了这些层的统一视图。目录名与各层的 ID 并非一一对应。

VFS 不支持写时复制（COW）。每当创建新层时，都会对父层进行一次深拷贝。这些层都位于 `/var/lib/docker/vfs/dir/` 下。

### 示例：磁盘上的镜像与容器

下面的 `docker pull` 命令展示了主机下载一个包含五层的 Docker 镜像：

```console
$ docker pull ubuntu

Using default tag: latest
latest: Pulling from library/ubuntu
e0a742c2abfd: Pull complete
486cb8339a27: Pull complete
dc6f0d824617: Pull complete
4f7a5649a30e: Pull complete
672363445ad2: Pull complete
Digest: sha256:84c334414e2bfdcae99509a6add166bbb4fa4041dc3fa6af08046a66fed3005f
Status: Downloaded newer image for ubuntu:latest
```

拉取完成后，每一层都会在 `/var/lib/docker/vfs/dir/` 下表现为一个子目录。目录名与 `docker pull` 输出中的镜像层 ID 不对应。要查看每层在磁盘上占用的空间，可以使用 `du -sh` 命令，它会以更易读的形式显示大小。

```console
$ ls -l /var/lib/docker/vfs/dir/

total 0
drwxr-xr-x.  2 root root  19 Aug  2 18:19 3262dfbe53dac3e1ab7dcc8ad5d8c4d586a11d2ac3c4234892e34bff7f6b821e
drwxr-xr-x. 21 root root 224 Aug  2 18:23 6af21814449345f55d88c403e66564faad965d6afa84b294ae6e740c9ded2561
drwxr-xr-x. 21 root root 224 Aug  2 18:23 6d3be4585ba32f9f5cbff0110e8d07aea5f5b9fbb1439677c27e7dfee263171c
drwxr-xr-x. 21 root root 224 Aug  2 18:23 9ecd2d88ca177413ab89f987e1507325285a7418fc76d0dcb4bc021447ba2bab
drwxr-xr-x. 21 root root 224 Aug  2 18:23 a292ac6341a65bf3a5da7b7c251e19de1294bd2ec32828de621d41c7ad31f895
drwxr-xr-x. 21 root root 224 Aug  2 18:23 e92be7a4a4e3ccbb7dd87695bca1a0ea373d4f673f455491b1342b33ed91446b
```

```console
$ du -sh /var/lib/docker/vfs/dir/*

4.0K	/var/lib/docker/vfs/dir/3262dfbe53dac3e1ab7dcc8ad5d8c4d586a11d2ac3c4234892e34bff7f6b821e
125M	/var/lib/docker/vfs/dir/6af21814449345f55d88c403e66564faad965d6afa84b294ae6e740c9ded2561
104M	/var/lib/docker/vfs/dir/6d3be4585ba32f9f5cbff0110e8d07aea5f5b9fbb1439677c27e7dfee263171c
125M	/var/lib/docker/vfs/dir/9ecd2d88ca177413ab89f987e1507325285a7418fc76d0dcb4bc021447ba2bab
104M	/var/lib/docker/vfs/dir/a292ac6341a65bf3a5da7b7c251e19de1294bd2ec32828de621d41c7ad31f895
104M	/var/lib/docker/vfs/dir/e92be7a4a4e3ccbb7dd87695bca1a0ea373d4f673f455491b1342b33ed91446b
```

上面的输出显示，其中三层各占用 104M，另外两层各占用 125M。尽管这些目录之间只有很小的差异，但它们各自都占据了完整的磁盘空间。这正是使用 `vfs` 存储驱动的劣势之一。

## 相关信息

- [理解镜像、容器与存储驱动](index.md)
- [选择存储驱动](select-storage-driver.md)
