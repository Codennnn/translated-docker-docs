---
description: 使用 tmpfs 挂载
title: tmpfs 挂载
weight: 30
keywords: storage, persistence, data persistence, tmpfs
aliases:
  - /engine/admin/volumes/tmpfs/
  - /storage/tmpfs/
---

[卷](volumes.md) 与[绑定挂载](bind-mounts.md)可在宿主机与容器之间共享文件，从而在容器停止后依然持久化数据。

如果你在 Linux 上运行 Docker，还有第三种选择：tmpfs 挂载。当你为容器使用 tmpfs 挂载时，容器会在自身可写层之外创建文件。

与卷和绑定挂载不同，tmpfs 挂载是临时的，只存在于宿主机内存中。容器停止时，tmpfs 挂载会被移除，写入其中的文件不会被持久化。

当你既不希望数据保存在宿主机上，也不希望保存在容器内时，最适合使用 tmpfs 挂载。这通常出于安全考虑，或是在应用需要写入大量非持久化状态数据时，避免影响容器性能。

> [!IMPORTANT]
> Docker 中的 tmpfs 挂载直接映射为 Linux 内核的 [tmpfs](https://en.wikipedia.org/wiki/Tmpfs)。
> 因此，临时数据可能会写入交换分区（swap），从而间接持久化到文件系统。

## 在已有数据之上进行挂载

如果你把 tmpfs 挂载到容器内本就包含文件或子目录的目录上，该目录中原有的内容会被挂载“遮蔽”。这类似于在 Linux 宿主机上向 `/mnt` 保存了文件后，再把一个 U 盘挂载到 `/mnt`：在卸载 U 盘之前，`/mnt` 下原有内容都会被 U 盘内容遮蔽。

在容器内部，没有直接“移除挂载并恢复显示被遮蔽文件”的简便做法。最稳妥的方法是：重新创建一个不包含该挂载的容器。

## tmpfs 挂载的限制

- 与卷或绑定挂载不同，多个容器之间无法共享同一个 tmpfs 挂载。
- 仅当你在 Linux 上运行 Docker 时才可用。
- 在 tmpfs 上设置权限，可能会[在容器重启后被重置](https://github.com/docker/for-linux/issues/138)。某些情况下，[设置 uid/gid](https://github.com/docker/compose/issues/3425#issuecomment-423091370) 可以作为变通方案。

## 语法

使用 `docker run` 挂载 tmpfs 时，可以使用 `--mount` 或 `--tmpfs` 参数。

```console
$ docker run --mount type=tmpfs,dst=<mount-path>
$ docker run --tmpfs <mount-path>
```

一般推荐使用 `--mount`，因为它更显式。另一方面，`--tmpfs` 更简洁，同时支持设置更多挂载选项，灵活性更高。

在 Swarm 服务中不能使用 `--tmpfs`，必须使用 `--mount`。

### --tmpfs 的可选项

`--tmpfs` 参数由两个以冒号（`:`）分隔的字段组成。

```console
$ docker run --tmpfs <mount-path>[:opts]
```

第一个字段是容器内要作为 tmpfs 的挂载路径。第二个字段为可选项，用于设置挂载参数。`--tmpfs` 支持的有效选项包括：

| 选项         | 说明                                                      |
| ------------ | --------------------------------------------------------- |
| `ro`         | 创建只读的 tmpfs 挂载。                                   |
| `rw`         | 创建读写的 tmpfs 挂载（默认）。                           |
| `nosuid`     | 禁止在执行时生效 `setuid` 与 `setgid` 位。                |
| `suid`       | 允许在执行时生效 `setuid` 与 `setgid` 位（默认）。        |
| `nodev`      | 允许创建设备文件，但不可使用（访问会报错）。              |
| `dev`        | 允许创建设备文件，且可正常使用。                          |
| `exec`       | 允许在该挂载的文件系统中执行二进制文件。                  |
| `noexec`     | 不允许在该挂载的文件系统中执行二进制文件。                |
| `sync`       | 对该文件系统的所有 I/O 同步执行。                         |
| `async`      | 对该文件系统的所有 I/O 异步执行（默认）。                 |
| `dirsync`    | 目录更新以同步方式进行。                                  |
| `atime`      | 每次访问文件时更新访问时间。                              |
| `noatime`    | 访问文件时不更新访问时间。                                |
| `diratime`   | 每次访问目录时更新目录访问时间。                          |
| `nodiratime` | 访问目录时不更新访问时间。                                |
| `size`       | 指定 tmpfs 挂载的大小，例如 `size=64m`。                  |
| `mode`       | 指定 tmpfs 挂载的文件权限（例如 `mode=1777`）。           |
| `uid`        | 指定 tmpfs 挂载所有者的用户 ID（例如 `uid=1000`）。       |
| `gid`        | 指定 tmpfs 挂载所有者的用户组 ID（例如 `gid=1000`）。     |
| `nr_inodes`  | 指定该 tmpfs 的最大 inode 数量（例如 `nr_inodes=400k`）。 |
| `nr_blocks`  | 指定该 tmpfs 的最大块数量（例如 `nr_blocks=1024`）。      |

```console {title="Example"}
$ docker run --tmpfs /data:noexec,size=1024,mode=1777
```

并非 Linux `mount` 命令支持的所有 tmpfs 能力都能通过 `--tmpfs` 使用。如果需要更高级的 tmpfs 选项或功能，你可能需要运行特权容器，或在 Docker 之外配置该挂载。

> [!CAUTION]
> 使用 `--privileged` 运行容器会赋予其更高权限，可能使宿主机暴露于安全风险之中。仅在确有必要且环境可信时使用该选项。

```console
$ docker run --privileged -it debian sh
/# mount -t tmpfs -o <options> tmpfs /data
```

### --mount 的可选项

`--mount` 由多个以逗号分隔的键值对组成，每个都是 `<key>=<value>` 元组。键的顺序不重要。

```console
$ docker run --mount type=tmpfs,dst=<mount-path>[,<key>=<value>...]
```

`--mount type=tmpfs` 支持的有效选项包括：

| 选项                           | 说明                                                                                     |
| :----------------------------- | :--------------------------------------------------------------------------------------- |
| `destination`, `dst`, `target` | 在容器内作为 tmpfs 的挂载路径。                                                          |
| `tmpfs-size`                   | 以字节为单位指定 tmpfs 挂载的大小。若未设置，默认最大值为宿主机总内存的 50%。            |
| `tmpfs-mode`                   | 以八进制指定 tmpfs 的文件权限，例如 `700` 或 `0770`。默认值为 `1777`（即所有用户可写）。 |

```console {title="Example"}
$ docker run --mount type=tmpfs,dst=/app,tmpfs-size=21474836480,tmpfs-mode=1770
```

## 在容器中使用 tmpfs 挂载

要在容器中使用 `tmpfs` 挂载，可以使用 `--tmpfs` 参数，或使用 `--mount` 并指定 `type=tmpfs` 与 `destination`。tmpfs 挂载没有 `source`。下面示例在 Nginx 容器中于 `/app` 创建一个 `tmpfs` 挂载。第一个示例使用 `--mount`，第二个示例使用 `--tmpfs`。

{{< tabs >}}
{{< tab name="`--mount`" >}}

```console
$ docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
```

通过 `docker inspect` 输出中的 `Mounts` 部分验证该挂载是否为 `tmpfs`：

```console
$ docker inspect tmptest --format '{{ json .Mounts }}'
[{"Type":"tmpfs","Source":"","Destination":"/app","Mode":"","RW":true,"Propagation":""}]
```

{{< /tab >}}
{{< tab name="`--tmpfs`" >}}

```console
$ docker run -d \
  -it \
  --name tmptest \
  --tmpfs /app \
  nginx:latest
```

同样可以通过 `docker inspect` 的 `Mounts` 部分进行验证：

```console
$ docker inspect tmptest --format '{{ json .Mounts }}'
{"/app":""}
```

{{< /tab >}}
{{< /tabs >}}

停止并删除容器：

```console
$ docker stop tmptest
$ docker rm tmptest
```

## 进一步阅读

- 了解[卷](volumes.md)
- 了解[绑定挂载](bind-mounts.md)
- 了解[存储驱动](/engine/storage/drivers/)
