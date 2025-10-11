---
description: 使用绑定挂载
title: 绑定挂载（Bind mounts）
weight: 20
keywords: storage, persistence, data persistence, mounts, bind mounts
aliases:
  - /engine/admin/volumes/bind-mounts/
  - /storage/bind-mounts/
---

使用绑定挂载（bind mount）时，会将宿主机上的某个文件或目录挂载到容器中。相比之下，使用卷（volume）时，Docker 会在宿主机上的 Docker 存储目录内创建一个新的目录，并由 Docker 负责管理该目录的内容。

## 何时使用绑定挂载

绑定挂载适用于以下场景：

- 在 Docker 宿主机上的开发环境与容器之间共享源代码或构建产物。

- 需要在容器内创建或生成文件，并将这些文件持久化到宿主机的文件系统上。

- 从宿主机向容器共享配置文件。Docker 默认就是通过将宿主机的 `/etc/resolv.conf` 挂载进每个容器来提供 DNS 解析的。

在构建阶段也可以使用绑定挂载：可以把宿主机上的源代码绑定到构建容器中，用于测试、Lint 或编译项目。

## 在已有数据上进行绑定挂载

如果你将某个文件或目录绑定挂载到容器内一个本就包含文件或子目录的目录上，该目录中原有的内容会被挂载后的内容“遮蔽”。这类似于在 Linux 宿主机上向 `/mnt` 保存了文件后，再把一个 U 盘挂载到 `/mnt`：在卸载 U 盘之前，`/mnt` 下原有的内容都会被 U 盘的内容遮蔽。

在容器中，并没有直接“移除挂载、恢复显示被遮蔽文件”的简便做法。最可取的方式是，重新创建一个不包含该挂载的容器。

## 注意事项与限制

- 绑定挂载默认对宿主机文件具有写权限。

  使用绑定挂载的一个副作用是：容器内的进程可以修改宿主机的文件系统，包括创建、修改或删除重要的系统文件或目录。这可能带来安全影响，例如影响宿主机上非 Docker 的进程。

  你可以使用 `readonly` 或 `ro` 选项，阻止容器向该挂载写入。

- 绑定挂载是相对于 Docker 守护进程所在的主机创建的，而不是相对于客户端。

  如果你使用远程 Docker 守护进程，则无法通过绑定挂载让容器访问客户端机器上的文件。

  对于 Docker Desktop，守护进程运行在 Linux 虚拟机中，而不是直接运行在本机。Docker Desktop 内置了机制，可透明地处理绑定挂载，允许将本机的文件路径共享给运行在虚拟机中的容器。

- 使用绑定挂载的容器与宿主机耦合较强。

  绑定挂载依赖宿主机文件系统存在特定的目录结构。这意味着，如果在另一台没有相同目录结构的主机上运行带有绑定挂载的容器，容器可能会失败。

## 语法

要创建绑定挂载，可以使用 `--mount` 或 `--volume` 标志。

```console
$ docker run --mount type=bind,src=<host-path>,dst=<container-path>
$ docker run --volume <host-path>:<container-path>
```

一般推荐使用 `--mount`。主要区别在于，`--mount` 更加显式，并且支持全部可用选项。

如果使用 `--volume` 将一个在宿主机上尚不存在的文件或目录进行绑定挂载，Docker 会在宿主机上为你自动创建该路径，并且总是以“目录”的形式创建。

如果使用 `--mount`，当宿主机上指定的挂载源路径不存在时，Docker 不会自动创建目录，而是返回错误：

```console
$ docker run --mount type=bind,src=/dev/noexist,dst=/mnt/foo alpine
docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /dev/noexist.
```

### --mount 的可选项

`--mount` 由多个以逗号分隔的键值对组成，每个都是一个 `<key>=<value>` 元组。键的顺序不重要。

```console
$ docker run --mount type=bind,src=<host-path>,dst=<container-path>[,<key>=<value>...]
```

`--mount type=bind` 的可选项包括：

| 选项                           | 说明                                                              |
| ------------------------------ | ----------------------------------------------------------------- |
| `source`, `src`                | 宿主机上文件或目录的位置。可以是绝对路径或相对路径。              |
| `destination`, `dst`, `target` | 文件或目录在容器内的挂载路径。必须是绝对路径。                    |
| `readonly`, `ro`               | 如果提供，则以只读方式[挂载到容器](#use-a-read-only-bind-mount)。 |
| `bind-propagation`             | 如果提供，则修改[绑定传播](#configure-bind-propagation)设置。     |

```console {title="Example"}
$ docker run --mount type=bind,src=.,dst=/project,ro,bind-propagation=rshared
```

### --volume 的可选项

`--volume` 或 `-v` 标志由三个以冒号（`:`）分隔的字段组成，字段顺序必须正确。

```console
$ docker run -v <host-path>:<container-path>[:opts]
```

第一个字段是要从宿主机绑定到容器内的路径。第二个字段是该文件或目录在容器内的挂载路径。

第三个字段是可选的，为以逗号分隔的选项列表。用于绑定挂载时，`--volume` 支持的选项包括：

| 选项              | 说明                                                                                   |
| ----------------- | -------------------------------------------------------------------------------------- |
| `readonly`, `ro`  | 如果提供，则以只读方式[挂载到容器](#use-a-read-only-bind-mount)。                      |
| `z`, `Z`          | 配置 SELinux 标签。参见[配置 SELinux 标签](#configure-the-selinux-label)。             |
| `rprivate` (默认) | 将该挂载的绑定传播设置为 `rprivate`。参见[配置绑定传播](#configure-bind-propagation)。 |
| `private`         | 将该挂载的绑定传播设置为 `private`。参见[配置绑定传播](#configure-bind-propagation)。  |
| `rshared`         | 将该挂载的绑定传播设置为 `rshared`。参见[配置绑定传播](#configure-bind-propagation)。  |
| `shared`          | 将该挂载的绑定传播设置为 `shared`。参见[配置绑定传播](#configure-bind-propagation)。   |
| `rslave`          | 将该挂载的绑定传播设置为 `rslave`。参见[配置绑定传播](#configure-bind-propagation)。   |
| `slave`           | 将该挂载的绑定传播设置为 `slave`。参见[配置绑定传播](#configure-bind-propagation)。    |

```console {title="Example"}
$ docker run -v .:/project:ro,rshared
```

## 使用绑定挂载启动容器

假设你有一个目录 `source`，构建该目录下的源代码时，构建产物会保存到 `source/target/`。你希望这些产物在容器的 `/app/` 下可用，并且每次在开发机上完成一次新构建后，容器都能看到最新产物。可以使用下面的命令，将 `target/` 绑定挂载到容器的 `/app/`。请在 `source` 目录下执行命令。`$(pwd)` 子命令会在 Linux 或 macOS 上展开为当前工作目录。
如果你在 Windows 上，请同时参考[Windows 上的路径转换](/manuals/desktop/troubleshoot-and-support/troubleshoot/topics.md)。

下面的 `--mount` 与 `-v` 示例效果相同。除非在运行第一个示例后删除 `devtest` 容器，否则不能两者都运行。

{{< tabs >}}
{{< tab name="`--mount`" >}}

```console
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```

{{< /tab >}}
{{< tab name="`-v`" >}}

```console
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  nginx:latest
```

{{< /tab >}}
{{< /tabs >}}

使用 `docker inspect devtest` 验证绑定挂载是否创建正确，查看输出中的 `Mounts` 字段：

```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

这段输出表明挂载类型为 `bind`，源与目标路径正确，挂载为读写（RW=true），并且传播策略为 `rprivate`。

停止并删除容器：

```console
$ docker container rm -fv devtest
```

### 挂载到容器内的非空目录

如果将一个目录绑定挂载到容器内的非空目录上，该目录中现有的内容会被绑定挂载所“遮蔽”。这在某些场景下很有用，例如你希望在不构建新镜像的情况下测试应用的新版本。不过该行为也可能令人意外，并且与[卷](volumes.md)的行为不同。

下面这个极端的示例把容器内的 `/usr/` 目录内容替换为宿主机的 `/tmp/` 目录。在大多数情况下，这会导致容器无法正常工作。

`--mount` 与 `-v` 两种写法的结果相同。

{{< tabs >}}
{{< tab name="`--mount`" >}}

```console
$ docker run -d \
  -it \
  --name broken-container \
  --mount type=bind,source=/tmp,target=/usr \
  nginx:latest

docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```

{{< /tab >}}
{{< tab name="`-v`" >}}

```console
$ docker run -d \
  -it \
  --name broken-container \
  -v /tmp:/usr \
  nginx:latest

docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```

{{< /tab >}}
{{< /tabs >}}

容器已创建但无法启动。将其删除：

```console
$ docker container rm broken-container
```

## 使用只读绑定挂载 {#use-a-read-only-bind-mount}

在某些开发场景中，容器需要向绑定挂载写入，以便变更能回写到宿主机。其他时候，容器仅需要读取权限。

下面的示例在前一个示例的基础上，将该目录以只读方式挂载。方法是在容器内挂载点之后，为（默认为空的）选项列表添加 `ro`。当存在多个选项时，用逗号分隔。

`--mount` 与 `-v` 两种写法的结果相同。

{{< tabs >}}
{{< tab name="`--mount`" >}}

```console
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
```

{{< /tab >}}
{{< tab name="`-v`" >}}

```console
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:ro \
  nginx:latest
```

{{< /tab >}}
{{< /tabs >}}

使用 `docker inspect devtest` 验证绑定挂载是否创建正确，查看输出中的 `Mounts` 字段：

```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "ro",
        "RW": false,
        "Propagation": "rprivate"
    }
],
```

停止并删除容器：

```console
$ docker container rm -fv devtest
```

## 递归挂载

当绑定挂载的路径自身包含其他挂载点时，这些子挂载默认也会包含在绑定挂载中。你可以通过 `--mount` 的 `bind-recursive` 选项对该行为进行配置。该选项仅在使用 `--mount` 时支持，`-v` 或 `--volume` 不支持。

如果该绑定挂载是只读的，Docker 引擎会尽力将子挂载也设为只读，这称为“递归只读挂载”。递归只读挂载需要 Linux 内核版本 5.12 或更高。如果你的内核版本更低，子挂载会默认以读写方式挂载。在 5.12 之前的内核上尝试通过 `bind-recursive=readonly` 将子挂载设为只读会导致错误。

`bind-recursive` 选项支持的取值如下：

| 取值              | 说明                                                            |
| :---------------- | :-------------------------------------------------------------- |
| `enabled`（默认） | 在内核 v5.12 及以上时，将只读挂载递归为只读；否则子挂载为读写。 |
| `disabled`        | 忽略子挂载（不包含在绑定挂载中）。                              |
| `writable`        | 子挂载为读写。                                                  |
| `readonly`        | 子挂载为只读。需要内核 v5.12 或更高。                           |

## 配置绑定传播 {#configure-bind-propagation}

绑定传播（bind propagation）在绑定挂载与卷上默认均为 `rprivate`。它只对绑定挂载可配置，且仅在 Linux 宿主机上生效。绑定传播属于进阶主题，很多用户无需配置。

绑定传播用于控制：在某个绑定挂载内部创建的挂载点，是否会传播到该挂载的“副本”上。假设有一个挂载点 `/mnt`，它也被挂载到了 `/tmp` 上。传播设置决定在 `/tmp/a` 上新增的挂载是否也会出现在 `/mnt/a`。每种传播设置都有其递归对应项。对于递归情况，设想把 `/tmp/a` 也挂载为 `/foo`，传播设置将决定 `/mnt/a` 与（或）`/tmp/a` 是否存在。

> [!NOTE]
> 在 Docker Desktop 中，挂载传播不可用。

| 传播设置   | 说明                                                                                             |
| :--------- | :----------------------------------------------------------------------------------------------- |
| `shared`   | 原始挂载的子挂载会暴露给副本挂载，副本挂载的子挂载也会传播回原始挂载。                           |
| `slave`    | 与 `shared` 类似，但仅单向。若原始挂载暴露子挂载，副本可见；但副本暴露子挂载时，原始挂载不可见。 |
| `private`  | 私有挂载。内部的子挂载不会暴露给副本挂载，副本挂载的子挂载也不会暴露给原始挂载。                 |
| `rshared`  | 与 `shared` 相同，但传播还会扩展到原始或副本挂载点内部嵌套的所有挂载点。                         |
| `rslave`   | 与 `slave` 相同，但传播还会扩展到原始或副本挂载点内部嵌套的所有挂载点。                          |
| `rprivate` | 默认设置。与 `private` 相同，表示原始与副本挂载点内的任意挂载点都不会向任一方向传播。            |

在为某个挂载点设置绑定传播之前，宿主机的文件系统必须已经支持该能力。

关于绑定传播的更多信息，参见
[Linux 内核 shared subtree 文档](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)。

下面的示例把 `target/` 目录在容器内挂载两次，第二次挂载同时设置了 `ro` 选项与 `rslave` 绑定传播选项。

`--mount` 与 `-v` 两种写法的结果相同。

{{< tabs >}}
{{< tab name="`--mount`" >}}

```console
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  --mount type=bind,source="$(pwd)"/target,target=/app2,readonly,bind-propagation=rslave \
  nginx:latest
```

{{< /tab >}}
{{< tab name="`-v`" >}}

```console
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  -v "$(pwd)"/target:/app2:ro,rslave \
  nginx:latest
```

{{< /tab >}}
{{< /tabs >}}

现在，如果你创建 `/app/foo/`，那么 `/app2/foo/` 也会同时存在。

## 配置 SELinux 标签 {#configure-the-selinux-label}

如果你使用 SELinux，可以添加 `z` 或 `Z` 选项来修改被挂载到容器内的宿主机文件或目录的 SELinux 标签。这会影响宿主机上的文件或目录本身，影响范围超出 Docker 之外。

- `z` 选项表示绑定挂载的内容可在多个容器之间共享。
- `Z` 选项表示绑定挂载的内容是私有的，不会共享。

使用这些选项时务必格外谨慎。对诸如 `/home` 或 `/usr` 这样的系统目录使用 `Z` 选项进行绑定挂载，可能会让你的宿主机无法正常工作，你可能需要手动为宿主机文件重新打标签。

> [!IMPORTANT]
>
> 在服务（services）中使用绑定挂载时，SELinux 标签（`:Z` 与 `:z`）以及 `:ro` 会被忽略。详情参见
> [moby/moby #32579](https://github.com/moby/moby/issues/32579)。

下面的示例设置了 `z` 选项，表示多个容器可以共享该绑定挂载的内容：

使用 `--mount` 标志无法修改 SELinux 标签。

```console
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:z \
  nginx:latest
```

## 在 Docker Compose 中使用绑定挂载

包含一个绑定挂载的 Docker Compose 服务示例如下：

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - type: bind
        source: ./static
        target: /opt/app/static
volumes:
  myapp:
```

关于在 Compose 中使用 `bind` 类型卷的更多信息，参见
[Compose 服务的 volumes 参考](/reference/compose-file/services.md#volumes)
以及
[Compose 中卷配置参考](/reference/compose-file/services.md#volumes)。

## 进一步阅读

- 了解[卷](./volumes.md)。
- 了解[tmpfs 挂载](./tmpfs.md)。
- 了解[存储驱动](/engine/storage/drivers/)。
