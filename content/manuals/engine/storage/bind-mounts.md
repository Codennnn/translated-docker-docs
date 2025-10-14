---
description: 了解如何使用绑定挂载，将宿主机的文件或目录挂载到容器。
title: 绑定挂载（Bind mounts）
weight: 20
keywords: storage, persistence, data persistence, mounts, bind mounts
aliases:
  - /engine/admin/volumes/bind-mounts/
  - /storage/bind-mounts/
---

绑定挂载（bind mount）的作用是将宿主机上的文件或目录直接挂载到容器中。这与卷（volume）的机制不同：卷是由 Docker 在宿主机的存储目录中创建并管理的独立目录。

## 何时使用绑定挂载

绑定挂载适用于以下场景：

- 在开发机与容器之间共享源代码或构建产物。例如，将本地项目目录挂载到容器中，实现代码的实时同步。

- 在容器内创建或生成文件，并将其持久化保存到宿主机文件系统。这样即使容器被删除，文件依然存在。

- 从宿主机向容器共享配置文件。例如，Docker 默认会将宿主机的 `/etc/resolv.conf` 文件挂载到每个容器中，以此为容器提供 DNS 解析功能。

在构建阶段也可以使用绑定挂载：将宿主机上的源代码目录绑定到构建容器内，用于执行测试、Lint 检查或编译操作。

## 在已有数据上进行绑定挂载

如果将某个文件或目录绑定挂载到容器内一个本身就包含文件或子目录的非空目录上，该目录中原有的内容会被挂载内容所“遮蔽”（隐藏）。这类似于在 Linux 宿主机上先向 `/mnt` 目录保存文件，然后再把一个 U 盘挂载到 `/mnt`：在 U 盘被卸载之前，`/mnt` 下原有的内容都会被 U 盘的内容所遮蔽，暂时无法访问。

容器运行时并没有提供“临时移除某个挂载以恢复显示被遮蔽文件”的简便方法。最稳妥的处理方式是重新创建一个不包含该挂载配置的新容器。

## 注意事项与限制

- 绑定挂载默认对宿主机文件具有读写权限。

  使用绑定挂载时需要注意：容器内的进程可以修改宿主机的文件系统，包括创建、修改或删除重要的系统文件或目录。这可能会带来安全隐患，甚至影响宿主机上非 Docker 进程的正常运行。

  你可以通过添加 `readonly` 或 `ro` 选项，将绑定挂载设置为只读，从而禁止容器向该挂载路径写入数据。

- 绑定挂载是相对于 Docker 守护进程（daemon）所在的主机创建的，而不是相对于客户端所在的机器。

  如果你使用远程 Docker 守护进程，就无法通过绑定挂载让容器访问客户端机器上的本地文件。

  对于 Docker Desktop 用户，守护进程实际运行在一个 Linux 虚拟机内，而不是直接运行在你的本地操作系统上。Docker Desktop 内置了路径转换机制，可以透明地处理绑定挂载，让你能够将本地文件路径共享给运行在虚拟机中的容器使用。

- 使用绑定挂载的容器与宿主机之间存在较强的耦合关系。

  绑定挂载依赖于宿主机文件系统具有特定的目录结构。这意味着，如果在另一台目录结构不同的主机上运行相同的容器配置，容器可能会因为找不到预期的挂载路径而启动失败。

## 语法

要创建绑定挂载，可以使用 `--mount` 或 `--volume` 参数。

```console
$ docker run --mount type=bind,src=<host-path>,dst=<container-path>
$ docker run --volume <host-path>:<container-path>
```

推荐优先使用 `--mount` 参数。它的语法更加明确，并且支持所有可用的配置选项。

如果使用 `--volume` 参数绑定挂载一个宿主机上尚不存在的文件或目录，Docker 会自动在宿主机上创建该路径，并且始终以“目录”的形式创建。

如果使用 `--mount` 参数，当宿主机上指定的源路径不存在时，Docker 不会自动创建目录，而是直接返回错误：

```console
$ docker run --mount type=bind,src=/dev/noexist,dst=/mnt/foo alpine
docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /dev/noexist.
```

### --mount 的可选项

`--mount` 参数由多个以逗号分隔的键值对组成，每项都是 `<key>=<value>` 格式的元组。键值对的顺序不重要。

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

`--volume` 或 `-v` 参数由三个以冒号（`:`）分隔的字段组成，字段的顺序必须正确。

```console
$ docker run -v <host-path>:<container-path>[:opts]
```

第一个字段是要从宿主机绑定到容器内的文件或目录路径。第二个字段是该文件或目录在容器内的挂载路径。

第三个字段是可选的，用于指定以逗号分隔的选项列表。用于绑定挂载时，`--volume` 支持的选项包括：

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

假设你有一个目录 `source`，在构建该目录下的源代码时，构建产物会保存到 `source/target/` 目录中。你希望这些构建产物在容器的 `/app/` 路径下可用，并且每次在开发机上完成新的构建后，容器都能自动看到最新的产物文件。可以使用下面的命令，将 `target/` 目录绑定挂载到容器的 `/app/` 路径。请在 `source` 目录下执行命令。其中 `$(pwd)` 子命令会在 Linux 或 macOS 上展开为当前工作目录的绝对路径。
如果你在 Windows 上，请同时参考[Windows 上的路径转换](/manuals/desktop/troubleshoot-and-support/troubleshoot/topics.md)。

下面的 `--mount` 与 `-v` 示例效果完全相同。除非在运行第一个示例后删除 `devtest` 容器，否则不要同时运行这两个命令。

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

这表明挂载类型为 `bind`，源路径与目标路径均正确，挂载模式为可读写（`RW=true`），传播策略为 `rprivate`。

停止并删除容器：

```console
$ docker container rm -fv devtest
```

### 挂载到容器内的非空目录

如果将一个目录绑定挂载到容器内的非空目录上，该目录中现有的内容会被绑定挂载所“遮蔽”（覆盖）。这在某些场景下很有用，例如你希望在不重新构建镜像的情况下测试应用的新版本。不过这种行为也可能出乎意料，并且与[卷](volumes.md)的行为有所不同。

下面这个极端的示例演示了将宿主机的 `/tmp/` 目录挂载到容器的 `/usr/` 目录，从而替换了容器内 `/usr/` 的原有内容。在大多数情况下，这会导致容器无法正常运行。

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

在某些开发场景中，容器需要向绑定挂载写入数据，以便将变更同步回宿主机；而在另一些场景中，容器只需要读取权限即可。

下面的示例在前文基础上，将目录以只读方式挂载到容器：在挂载路径后面添加 `ro` 选项（默认情况下选项列表为空）。如果存在多个选项，用逗号分隔。

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

当绑定挂载的源路径本身包含其他挂载点时，这些子挂载默认也会被包含在绑定挂载中。你可以通过 `--mount` 参数的 `bind-recursive` 选项来配置这一行为。需要注意的是，该选项仅在使用 `--mount` 时支持，使用 `-v` 或 `--volume` 时不支持。

如果绑定挂载本身是只读的，Docker 引擎会尝试将所有子挂载也设置为只读，这被称为“递归只读挂载”。递归只读挂载功能需要 Linux 内核 5.12 或更高版本。如果内核版本较低，子挂载会默认以读写方式挂载。在内核 5.12 之前的版本上尝试通过 `bind-recursive=readonly` 强制将子挂载设为只读会导致错误。

`bind-recursive` 选项支持的取值如下：

| 取值              | 说明                                                                          |
| :---------------- | :---------------------------------------------------------------------------- |
| `enabled`（默认） | 在内核 v5.12 及以上版本时，将只读挂载递归设置为只读；否则子挂载为读写模式。 |
| `disabled`        | 忽略所有子挂载（不包含在绑定挂载中）。                                        |
| `writable`        | 所有子挂载均为读写模式。                                                      |
| `readonly`        | 所有子挂载均为只读模式。需要内核 v5.12 或更高版本。                           |

## 配置绑定传播 {#configure-bind-propagation}

绑定传播（bind propagation）在绑定挂载和卷上的默认值均为 `rprivate`。该配置仅对绑定挂载可用，并且只在 Linux 宿主机上生效。绑定传播属于进阶主题，大多数用户无需配置。

绑定传播用于控制：在某个绑定挂载内部创建的子挂载点，是否会传播到该挂载的“副本”上。举例来说，假设有一个挂载点 `/mnt`，它同时也被挂载到了 `/tmp`。传播设置决定了当在 `/tmp/a` 上创建新的挂载时，这个挂载是否也会出现在 `/mnt/a` 中。每种传播设置都有其递归对应项。对于递归情况，假设将 `/tmp/a` 也挂载到 `/foo`，传播设置将决定 `/mnt/a` 和（或）`/tmp/a` 下是否会出现这个挂载。

> [!NOTE]
> 在 Docker Desktop 中，挂载传播不可用。

| 传播设置   | 说明                                                                                                   |
| :--------- | :----------------------------------------------------------------------------------------------------- |
| `shared`   | 原始挂载的子挂载会暴露给副本挂载，副本挂载的子挂载也会传播回原始挂载。双向传播。                       |
| `slave`    | 与 `shared` 类似，但仅单向传播。若原始挂载暴露子挂载，副本可见；但副本暴露子挂载时，原始挂载不可见。   |
| `private`  | 私有挂载。内部的子挂载不会暴露给副本挂载，副本挂载的子挂载也不会暴露给原始挂载。完全隔离。             |
| `rshared`  | 与 `shared` 相同，但传播还会递归扩展到原始或副本挂载点内部嵌套的所有挂载点。                           |
| `rslave`   | 与 `slave` 相同，但传播还会递归扩展到原始或副本挂载点内部嵌套的所有挂载点。                            |
| `rprivate` | 默认设置。与 `private` 相同，表示原始与副本挂载点内的任意子挂载点都不会向任何方向传播。完全递归隔离。 |

在为某个挂载点设置绑定传播之前，宿主机文件系统必须已经支持绑定传播功能。

关于绑定传播的更多详细信息，请参见
[Linux 内核 shared subtree 文档](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)。

下面的示例将 `target/` 目录在容器内挂载两次，第二次挂载同时设置了 `ro`（只读）选项和 `rslave` 绑定传播选项。

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

现在，如果你在 `/app/foo/` 中创建内容，那么 `/app2/foo/` 中也会同时存在相应的内容。

## 配置 SELinux 标签 {#configure-the-selinux-label}

如果你使用 SELinux，可以通过添加 `z` 或 `Z` 选项来修改被挂载到容器内的宿主机文件或目录的 SELinux 标签。需要注意的是，这些操作会直接影响宿主机上的文件或目录本身，影响范围超出 Docker 容器之外。

- `z` 选项：表示绑定挂载的内容可以在多个容器之间共享。
- `Z` 选项：表示绑定挂载的内容是私有的，仅供单个容器使用，不会共享给其他容器。

使用这些选项时务必格外谨慎。对诸如 `/home` 或 `/usr` 这样的系统目录使用 `Z` 选项进行绑定挂载，可能会导致宿主机无法正常运行，你可能需要手动为宿主机文件重新设置标签才能恢复。

> [!IMPORTANT]
>
> 在服务（services）中使用绑定挂载时，SELinux 标签（`:Z` 和 `:z`）以及 `:ro` 选项会被忽略。详情请参见
> [moby/moby #32579](https://github.com/moby/moby/issues/32579)。

下面的示例设置了 `z` 选项，表示多个容器可以共享该绑定挂载的内容：

需要注意的是，使用 `--mount` 参数无法修改 SELinux 标签。

```console
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:z \
  nginx:latest
```

## 在 Docker Compose 中使用绑定挂载

下面是一个包含绑定挂载配置的 Docker Compose 服务示例：

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

关于在 Compose 中使用 `bind` 类型卷的更多详细信息，请参见
[Compose 服务的 volumes 配置参考](/reference/compose-file/services.md#volumes)
以及
[Compose 中卷的配置参考](/reference/compose-file/services.md#volumes)。

## 进一步阅读

- 了解[卷](./volumes.md)。
- 了解[tmpfs 挂载](./tmpfs.md)。
- 了解[存储驱动](/engine/storage/drivers/)。
