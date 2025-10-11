---
description: "使用 Docker CLI 运行与配置容器"
keywords: "docker, run, cli"
aliases:
- /reference/run/
title: 运行容器
---

Docker 会在隔离的容器中运行进程。容器本质上是运行在宿主上的一个进程，宿主可以是本机或远程主机。执行 `docker run` 时，容器中的进程与宿主隔离：它拥有独立的文件系统、独立的网络命名空间，以及与宿主分离的进程树。

本文介绍如何使用 `docker run` 命令来运行容器。

## 基本用法

`docker run` 的通用格式如下：

```console
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

使用 `docker run` 时必须指定用于创建容器的[镜像引用](#image-references)。

### 镜像引用

镜像引用由镜像名称与版本组成。你可以使用镜像引用基于某个镜像创建或运行容器。

- `docker run IMAGE[:TAG][@DIGEST]`
- `docker create IMAGE[:TAG][@DIGEST]`

镜像标签（tag）表示镜像的版本，若省略则默认为 `latest`。你可以通过标签从指定版本的镜像启动容器。例如，运行 `ubuntu` 镜像的 `24.04` 版本：`docker run ubuntu:24.04`。

#### 镜像摘要（digest）

采用 v2 或更新格式的镜像具有基于内容寻址的标识符，称为摘要（digest）。只要构建镜像的输入不变，摘要值就是可预测的。

下面的示例通过摘要 `sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0` 来运行 `alpine` 镜像：

```console
$ docker run alpine@sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0 date
```

### 选项

`[OPTIONS]` 用于配置容器的各种行为。例如，可以为容器指定名称（`--name`），或以后台方式运行（`-d`）。你也可以通过选项控制资源约束、网络等。

### 命令与参数

你可以使用位置参数 `[COMMAND]` 与 `[ARG...]` 指定容器启动时要执行的命令及参数。例如，将 `[COMMAND]` 指定为 `sh`，并结合 `-i` 与 `-t` 选项，即可在容器中启动交互式 Shell（前提是所选镜像的 `PATH` 中存在 `sh` 可执行文件）。

```console
$ docker run -it IMAGE sh
```

> [!NOTE]
> 取决于你的 Docker 系统配置，使用 `docker run` 时可能需要加上 `sudo`。为避免在执行 `docker` 命令时使用 `sudo`，系统管理员可以创建一个名为 `docker` 的 Unix 组并将用户加入其中。具体步骤请参考对应操作系统的 Docker 安装文档。

## 前台与后台

默认情况下，容器以“前台”方式运行。如果希望在后台运行容器，可以使用 `--detach`（或 `-d`）标志。这样容器会在后台启动，而不会占用当前终端。

```console
$ docker run -d <IMAGE>
```

当容器在后台运行时，你仍可以使用其他 CLI 命令与容器交互。例如，使用 `docker logs` 查看容器日志，或用 `docker attach` 将其切回前台。

```console
$ docker run -d nginx
0246aa4d1448a401cabd2ce8f242192b6e7af721527e48a810463366c7ff54f1
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS        PORTS     NAMES
0246aa4d1448   nginx     "/docker-entrypoint.…"   2 seconds ago   Up 1 second   80/tcp    pedantic_liskov
$ docker logs -n 5 0246aa4d1448
2023/11/06 15:58:23 [notice] 1#1: start worker process 33
2023/11/06 15:58:23 [notice] 1#1: start worker process 34
2023/11/06 15:58:23 [notice] 1#1: start worker process 35
2023/11/06 15:58:23 [notice] 1#1: start worker process 36
2023/11/06 15:58:23 [notice] 1#1: start worker process 37
$ docker attach 0246aa4d1448
^C
2023/11/06 15:58:40 [notice] 1#1: signal 2 (SIGINT) received, exiting
...
```

关于前台/后台的相关标志，请参见：

- [`docker run --detach`](https://docs.docker.com/reference/cli/docker/container/run/#detach)：以后台方式运行容器
- [`docker run --attach`](https://docs.docker.com/reference/cli/docker/container/run/#attach)：附加到 `stdin`、`stdout` 与 `stderr`
- [`docker run --tty`](https://docs.docker.com/reference/cli/docker/container/run/#tty)：分配伪终端（pseudo-tty）
- [`docker run --interactive`](https://docs.docker.com/reference/cli/docker/container/run/#interactive)：即使没有附加也保持 `stdin` 打开

如需了解如何重新附加到后台容器，请参见 [`docker attach`](https://docs.docker.com/reference/cli/docker/container/attach/)。

## 容器标识

你可以通过以下三种方式标识一个容器：

| 标识类型             | 示例值                                                                 |
|:---------------------|:------------------------------------------------------------------------|
| UUID 长 ID           | `f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778`      |
| UUID 短 ID           | `f78375b1c487`                                                          |
| 名称                 | `evil_ptolemy`                                                          |

UUID 由守护进程随机分配。

守护进程也会为容器自动生成随机名称。你也可以通过 [`--name` 标志](https://docs.docker.com/reference/cli/docker/container/run/#name)自定义容器名称。合理的命名有助于理解容器用途。在用户定义的网络中，你可以使用名称来引用容器（前台/后台容器均适用）。

需要注意的是，容器标识符与镜像引用不是一回事。镜像引用用于指定运行容器所用的镜像。你不能通过 `docker exec nginx:alpine sh` 打开基于 `nginx:alpine` 的容器的 Shell，因为 `docker exec` 需要的是容器标识（名称或 ID），而不是镜像。

虽然容器所用镜像本身不构成容器标识，但可以通过 `--filter` 查找使用某镜像的容器 ID。例如，下面的 `docker ps` 会返回所有基于 `nginx:alpine` 的正在运行的容器 ID：

```console
$ docker ps -q --filter ancestor=nginx:alpine
```

关于筛选的更多信息，请参见[筛选](https://docs.docker.com/config/filter/)。

## 容器网络

容器默认启用网络功能，并且可以发起出站连接。如果你需要让多个容器之间互联通信，可以创建自定义网络并将容器加入该网络。

当多个容器连接到同一个自定义网络时，它们可以使用容器名称作为 DNS 主机名彼此通信。下面的示例创建了名为 `my-net` 的自定义网络，并将两个容器连接到该网络：

```console
$ docker network create my-net
$ docker run -d --name web --network my-net nginx:alpine
$ docker run --rm -it --network my-net busybox
/ # ping web
PING web (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.326 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.257 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.281 ms
^C
--- web ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.257/0.288/0.326 ms
```

关于容器网络的更多内容，请参见[网络概览](https://docs.docker.com/network/)。

## 文件系统挂载

默认情况下，容器中的数据存储在一个短暂（ephemeral）的可写容器层中，删除容器会一并删除其数据。如果你希望持久化数据，可以使用文件系统挂载，将数据保存在宿主机上。挂载还可以在多个容器与宿主机之间共享数据。

Docker 支持两大类挂载：

- 卷（volume）挂载
- 绑定（bind）挂载

卷非常适合用于容器数据的持久化存储以及容器之间共享数据；而绑定挂载更适合在容器与宿主机之间共享数据。

你可以在 `docker run` 中通过 `--mount` 为容器添加文件系统挂载。

下面的章节展示了创建卷与绑定挂载的基础示例。更深入的示例与说明，请参见文档的[存储](https://docs.docker.com/storage/)部分。

### 卷挂载（Volume mounts）

创建一个卷挂载：

```console
$ docker run --mount source=<VOLUME_NAME>,target=[PATH] [IMAGE] [COMMAND...]
```

在这种用法中，`--mount` 至少包含两个参数：`source` 与 `target`。其中，`source` 为卷的名称，`target` 为该卷在容器内的挂载路径。创建卷后，写入该卷的数据会被持久化，即使停止或删除容器也不会丢失：

```console
$ docker run --rm --mount source=my_volume,target=/foo busybox \
  echo "hello, volume!" > /foo/hello.txt
$ docker run --mount source=my_volume,target=/bar busybox
  cat /bar/hello.txt
hello, volume!
```

`target` 必须是绝对路径，如 `/src/docs`。绝对路径以 `/` 开头。卷名必须以字母或数字开头，后续可以是 `a-z0-9`、下划线 `_`、点号 `.` 或连字符 `-`。

### 绑定挂载（Bind mounts）

创建一个绑定挂载：

```console
$ docker run -it --mount type=bind,source=[PATH],target=[PATH] busybox
```

此时，`--mount` 包含三个参数：类型（`bind`）与两条路径。`source` 是宿主机上要挂载进容器的路径；`target` 是容器内的挂载目标路径。

绑定挂载默认是读写的，这意味着你可以在容器中读取与修改该挂载位置中的文件；这些修改会同步反映到宿主机的文件系统上：

```console
$ docker run -it --mount type=bind,source=.,target=/foo busybox
/ # echo "hello from container" > /foo/hello.txt
/ # exit
$ cat hello.txt
hello from container
```

## 退出状态码

`docker run` 的退出码可以反映容器为何运行失败或退出的原因。以下对不同退出码的含义进行说明。

### 125

退出码 `125` 表示 Docker 守护进程自身出现错误。

```console
$ docker run --foo busybox; echo $?

flag provided but not defined: --foo
See 'docker run --help'.
125
```

### 126

退出码 `126` 表示指定的容器内命令无法被调用。下例中容器内的命令为 `/etc`：

```console
$ docker run busybox /etc; echo $?

docker: Error response from daemon: Container command '/etc' could not be invoked.
126
```

### 127

退出码 `127` 表示容器内指定的命令未找到。

```console
$ docker run busybox foo; echo $?

docker: Error response from daemon: Container command 'foo' not found or does not exist.
127
```

### 其他退出码

除 `125`、`126`、`127` 以外的任何退出码，都表示容器中“所执行命令本身”的退出码。

```console
$ docker run busybox /bin/sh -c 'exit 3'
$ echo $?
3
```

## 运行时资源约束

运维人员可以通过以下选项调整容器的性能参数：

| 选项                      | 描述                                                                                                                                                                                                                                                                                 |
|:--------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `-m`, `--memory=""`       | 内存上限（格式：`<number>[<unit>]`）。`number` 为正整数，单位可为 `b`、`k`、`m` 或 `g`。最小值为 6M。                                                                                                                                                                              |
| `--memory-swap=""`        | 内存总上限（内存 + 交换分区，格式：`<number>[<unit>]`）。`number` 为正整数，单位可为 `b`、`k`、`m` 或 `g`。                                                                                                                                                                          |
| `--memory-reservation=""` | 内存软限制（格式：`<number>[<unit>]`）。`number` 为正整数，单位可为 `b`、`k`、`m` 或 `g`。                                                                                                                                                                                            |
| `--kernel-memory=""`      | 内核内存上限（格式：`<number>[<unit>]`）。`number` 为正整数，单位可为 `b`、`k`、`m` 或 `g`。最小值为 4M。                                                                                                                                                                             |
| `-c`, `--cpu-shares=0`    | CPU 份额（相对权重）                                                                                                                                                                                                                                                                |
| `--cpus=0.000`            | CPU 数量。为小数。`0.000` 表示不限制。                                                                                                                                                                                                                                               |
| `--cpu-period=0`          | 限制 CPU CFS（完全公平调度器）周期                                                                                                                                                                                                                                                  |
| `--cpuset-cpus=""`        | 允许容器运行的 CPU 集（如 `0-3`、`0,1`）                                                                                                                                                                                                                                             |
| `--cpuset-mems=""`        | 允许容器使用的内存节点（仅在 NUMA 系统有效，如 `0-3`、`0,1`）                                                                                                                                                                                                                       |
| `--cpu-quota=0`           | 限制 CPU CFS（完全公平调度器）配额                                                                                                                                                                                                                                                   |
| `--cpu-rt-period=0`       | 限制 CPU 实时周期（微秒）。需要已配置父 cgroup，且不能高于父级；同时需关注 `rtprio` ulimit。                                                                                                                                                                                         |
| `--cpu-rt-runtime=0`      | 限制 CPU 实时运行时长（微秒）。需要已配置父 cgroup，且不能高于父级；同时需关注 `rtprio` ulimit。                                                                                                                                                                                      |
| `--blkio-weight=0`        | 块 IO 权重（相对权重），取值 10 到 1000。                                                                                                                                                                                                                                            |
| `--blkio-weight-device=""`| 块 IO 权重（特定设备权重，格式：`DEVICE_NAME:WEIGHT`）                                                                                                                                                                                                                                |
| `--device-read-bps=""`    | 限制从某设备的读取速率（格式：`<device-path>:<number>[<unit>]`）。`number` 为正整数，单位可为 `kb`、`mb` 或 `gb`。                                                                                                                                                                   |
| `--device-write-bps=""`   | 限制写入某设备的速率（格式：`<device-path>:<number>[<unit>]`）。`number` 为正整数，单位可为 `kb`、`mb` 或 `gb`。                                                                                                                                                                     |
| `--device-read-iops=""`   | 限制从某设备读取的 IOPS（格式：`<device-path>:<number>`）。`number` 为正整数。                                                                                                                                                                                                       |
| `--device-write-iops=""`  | 限制写入某设备的 IOPS（格式：`<device-path>:<number>`）。`number` 为正整数。                                                                                                                                                                                                         |
| `--oom-kill-disable=false` | 是否为容器禁用 OOM Killer。                                                                                                                                                                                                                                                          |
| `--oom-score-adj=0`       | 调整容器的 OOM 评分（-1000 到 1000）                                                                                                                                                                                                                                                  |
| `--memory-swappiness=""`  | 调整容器的内存换出（swappiness）行为，取值 0 到 100。                                                                                                                                                                                                                                 |
| `--shm-size=""`           | `/dev/shm` 的大小。格式为 `<number><unit>`，`number` 必须大于 0。单位可选：`b`、`k`、`m`、`g`；若省略单位则使用字节；若完全省略大小则默认 `64m`。                                                                                                                                |

### 用户态内存约束

设置用户态内存使用有四种方式：

<table>
  <thead>
    <tr>
      <th>选项</th>
      <th>结果</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="no-wrap">
          <strong>memory=inf, memory-swap=inf</strong>（默认）
      </td>
      <td>
        不对容器施加内存限制。容器可按需使用内存。
      </td>
    </tr>
    <tr>
      <td class="no-wrap"><strong>memory=L&lt;inf, memory-swap=inf</strong></td>
      <td>
        （指定 memory 并将 memory-swap 设为 <code>-1</code>）容器最多可使用 L 字节内存，交换分区按需使用（前提是宿主支持 swap）。
      </td>
    </tr>
    <tr>
      <td class="no-wrap"><strong>memory=L&lt;inf, memory-swap=2*L</strong></td>
      <td>
        （仅指定 memory，未指定 memory-swap）容器最多可使用 L 字节内存；内存加交换分区的总和默认为其两倍。
      </td>
    </tr>
    <tr>
      <td class="no-wrap">
          <strong>memory=L&lt;inf, memory-swap=S&lt;inf, L&lt;=S</strong>
      </td>
      <td>
        （同时指定 memory 与 memory-swap）容器最多可使用 L 字节内存；内存加交换分区的总量受 S 限制。
      </td>
    </tr>
  </tbody>
</table>

示例：

```console
$ docker run -it ubuntu:24.04 /bin/bash
```

未设置任何内存限制，这意味着容器中的进程可按需使用内存与交换分区。

```console
$ docker run -it -m 300M --memory-swap -1 ubuntu:24.04 /bin/bash
```

设定了内存上限，同时不限制交换分区。这表示容器中的进程可以使用 300M 内存，并按需使用交换分区（前提是宿主支持 swap）。

```console
$ docker run -it -m 300M ubuntu:24.04 /bin/bash
```

仅设置了内存上限。这意味着容器中的进程可以使用 300M 内存与 300M 交换分区。默认情况下，总虚拟内存（`--memory-swap`）会被设置为内存的两倍，本例中为 600M，因此可使用 300M 的交换分区。

```console
$ docker run -it -m 300M --memory-swap 1G ubuntu:24.04 /bin/bash
```

同时设置了内存与交换分区上限，因此容器中的进程可以使用 300M 内存与 700M 交换分区。

内存保留（memory reservation）属于软限制机制，有助于在多容器共享内存时实现更好的资源分配。正常情况下，容器可按需使用内存，并仅受 `-m/--memory` 设置的硬限制约束。当设置了保留值后，Docker 在检测到内存争用或系统内存不足时，会强制容器将内存消耗收敛到保留值附近。

务必将内存保留值设置在硬限制之下，否则硬限制会优先生效。保留值为 0 等同于不设置保留。默认（未设置保留）时，保留值等同于硬限制。

请注意，内存保留是软限制，并不保证不会被超出；它尝试在发生内存争用时，依据保留配置来分配内存。

下面的示例将内存上限（`-m`）设为 500M，并将内存保留设为 200M：

```console
$ docker run -it -m 500M --memory-reservation 200M ubuntu:24.04 /bin/bash
```

在此配置下，当容器内存消耗高于 200M 且低于 500M 时，下一次系统内存回收会尝试将容器内存压回到 200M 以下。

下面的示例仅设置 1G 的内存保留，且不设置硬限制：

```console
$ docker run -it --memory-reservation 1G ubuntu:24.04 /bin/bash
```

容器可按需使用内存。内存保留确保容器不会长时间占用过多内存，因为每次内存回收都会将其消耗收敛至保留值。

默认情况下，当发生内存不足（OOM）时，内核会杀死容器内的进程。你可以通过 `--oom-kill-disable` 更改该行为。仅在同时设置了 `-m/--memory` 时才建议禁用 OOM killer；如果未设置 `-m` 就禁用 OOM killer，可能导致宿主机内存耗尽，进而需要杀死宿主上的系统进程来释放内存。

下面的示例将内存限制为 100M，并为该容器禁用 OOM killer：

```console
$ docker run -it -m 100M --oom-kill-disable ubuntu:24.04 /bin/bash
```

下面的示例展示了一个危险的用法：

```console
$ docker run -it --oom-kill-disable ubuntu:24.04 /bin/bash
```

该容器拥有无限内存配额，可能导致宿主机内存耗尽，不得不杀死系统进程以释放内存。你可以通过 `--oom-score-adj` 调整容器在 OOM 时被杀死的优先级；分值为负时越不易被杀死，分值为正时越容易被杀死。

### 内核内存约束

内核内存与用户态内存有本质区别：内核内存无法被换出（swap）。由于不可换出，若容器占用过多内核内存，可能阻塞系统服务。内核内存包括：

 - 栈页（stack pages）
 - slab 页
 - 套接字内存压力（sockets memory pressure）
 - TCP 内存压力（tcp memory pressure）

你可以设置内核内存上限来约束这些内存的使用。例如，每个进程都会消耗一定数量的栈页；限制内核内存可以在使用过高时阻止新进程创建。

内核内存从不完全独立于用户态内存。实际上，内核内存限制是在用户态内存限制的语境下生效。设“U”为用户态内存上限，“K”为内核内存上限。可以有三种设置方式：

<table>
  <thead>
    <tr>
      <th>选项</th>
      <th>结果</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="no-wrap"><strong>U != 0, K = inf</strong>（默认）</td>
      <td>
        这是在未使用内核内存限制前就存在的标准内存限制机制。忽略内核内存限制。
      </td>
    </tr>
    <tr>
      <td class="no-wrap"><strong>U != 0, K &lt; U</strong></td>
      <td>
        内核内存是用户态内存的子集。该配置在按 cgroup 过度分配总内存的部署中有用。但不推荐过度分配内核内存，因为主机仍可能耗尽不可回收的内存。
        在这种情况下，你可以配置 K，使得所有分组的内核内存之和不超过总内存；然后在不影响系统服务质量的前提下自由设置 U。
      </td>
    </tr>
    <tr>
      <td class="no-wrap"><strong>U != 0, K &gt; U</strong></td>
      <td>
        由于内核内存的计费同样会计入用户态内存计数，且对两类内存都可能触发回收，因此该配置为管理员提供了统一的内存视图，也适用于只想跟踪内核内存使用的人群。
      </td>
    </tr>
  </tbody>
</table>

示例：

```console
$ docker run -it -m 500M --kernel-memory 50M ubuntu:24.04 /bin/bash
```

同时设置了总内存与内核内存：容器总内存可达 500M，其中最多 50M 可用于内核内存。

```console
$ docker run -it --kernel-memory 50M ubuntu:24.04 /bin/bash
```

仅设置内核内存、未设置 `-m` 时：容器可按需使用总内存，但内核内存最多为 50M。

### Swappiness（换出）约束

默认情况下，内核可以将容器的匿名页按一定比例换出（swap）。你可以通过 `--memory-swappiness` 将该比例设置为 0 到 100 之间的值。设置为 0 则关闭匿名页换出；设置为 100 则允许所有匿名页可被换出。若未指定 `--memory-swappiness`，容器会继承父级的换出策略。

例如：

```console
$ docker run -it --memory-swappiness=0 ubuntu:24.04 /bin/bash
```

在希望保留工作集、避免换出带来的性能损失时，设置 `--memory-swappiness` 会很有帮助。

### CPU 份额（share）约束

默认情况下，所有容器获得相同比例的 CPU 时间份额。你可以通过调整 CPU 份额权重来改变某个容器相对于其他容器的 CPU 份额。

使用 `-c` 或 `--cpu-shares` 将默认权重 1024 调整为 2 或更高的值。若设置为 0，系统会忽略该值并沿用默认 1024。

该权重仅在存在 CPU 密集型进程时生效；当某个容器任务空闲时，其他容器可以使用剩余的 CPU 时间。实际获得的 CPU 时间与系统上运行的容器数量有关。

例如，假设有三个容器：一个的份额为 1024，另外两个为 512。当三者都尝试使用 100% CPU 时，第一个容器将获得 50% 的总 CPU 时间。若再增加一个份额为 1024 的容器，则第一个容器仅获得 33% 的 CPU，其他容器分别为 16.5%、16.5% 与 33%。

在多核系统上，CPU 时间份额会在所有 CPU 核心间分布。即使容器被限制为少于 100% 的总 CPU 时间，它仍可能在每个核心上各自使用 100% 的时间片。

例如，在一个拥有至少 3 个核心的系统上，如果启动一个容器 `{C0}`（`-c=512`）运行一个进程，另一个容器 `{C1}`（`-c=1024`）运行两个进程，可能出现如下分配：

    PID    container	CPU	CPU share
    100    {C0}		0	CPU0 的 100%
    101    {C1}		1	CPU1 的 100%
    102    {C1}		2	CPU2 的 100%

### CPU 周期（period）约束

默认的 CPU CFS（完全公平调度器）周期为 100ms。你可以使用 `--cpu-period` 设置调度周期，通常配合 `--cpu-quota` 一起使用。

示例：

```console
$ docker run -it --cpu-period=50000 --cpu-quota=25000 ubuntu:24.04 /bin/bash
```

若系统只有 1 个 CPU，这表示容器每 50ms 获得 50% CPU 的运行时间。

除了使用 `--cpu-period` 与 `--cpu-quota` 设置 CPU 周期约束外，也可以通过 `--cpus` 指定浮点数达到同样目的。例如在 1 个 CPU 的情况下，`--cpus=0.5` 等价于 `--cpu-period=50000` 与 `--cpu-quota=25000`（即 50% CPU）。

`--cpus` 的默认值为 `0.000`，表示不限制。

更多信息请参考 [CFS 带宽限制文档](https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt)。

### CPU 集（cpuset）约束

你可以为容器指定允许运行的 CPU 集。

示例：

```console
$ docker run -it --cpuset-cpus="1,3" ubuntu:24.04 /bin/bash
```

表示容器内的进程仅可在 CPU 1 与 CPU 3 上运行。

```console
$ docker run -it --cpuset-cpus="0-2" ubuntu:24.04 /bin/bash
```

表示容器内的进程仅可在 CPU 0、CPU 1 与 CPU 2 上运行。

你也可以指定允许容器使用的内存节点（仅在 NUMA 系统有效）。

示例：

```console
$ docker run -it --cpuset-mems="1,3" ubuntu:24.04 /bin/bash
```

该示例限制容器仅使用内存节点 1 与 3。

```console
$ docker run -it --cpuset-mems="0-2" ubuntu:24.04 /bin/bash
```

该示例限制容器仅使用内存节点 0、1 与 2。

### CPU 配额（quota）约束

`--cpu-quota` 用于限制容器的 CPU 使用率。默认值 0 表示容器可以占用 100% 的单个 CPU 资源（1 个 CPU）。CFS（完全公平调度器）负责进程的资源分配，也是内核默认的 Linux 调度器。将该值设为 `50000` 可将容器限制在 50% 的单核 CPU 资源。对于多 CPU，请根据需要调整 `--cpu-quota`。更多信息请参考 [CFS 带宽限制文档](https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt)。

### 块 IO 带宽（Blkio）约束

默认情况下，所有容器获得相同比例的块 IO 带宽（blkio），默认权重为 500。你可以通过 `--blkio-weight` 修改容器的 blkio 权重，从而相对调整其与其他正在运行容器之间的 blkio 配额。

> [!NOTE]
> blkio 权重仅对直写（direct IO）生效，当前不支持缓冲 IO。

`--blkio-weight` 的取值范围为 10 到 1000。如下示例创建两个 blkio 权重不同的容器：

```console
$ docker run -it --name c1 --blkio-weight 300 ubuntu:24.04 /bin/bash
$ docker run -it --name c2 --blkio-weight 600 ubuntu:24.04 /bin/bash
```

如果同时在两个容器中进行块 IO，例如运行：

```console
$ time dd if=/mnt/zerofile of=test.out bs=1M count=1024 oflag=direct
```

你会发现两者耗时的比例与其 blkio 权重比例一致。

`--blkio-weight-device="DEVICE_NAME:WEIGHT"` 用于设置某个设备的特定权重。`DEVICE_NAME:WEIGHT` 为以冒号分隔的设备名与权重值。例如将 `/dev/sda` 的权重设置为 `200`：

```console
$ docker run -it \
    --blkio-weight-device "/dev/sda:200" \
    ubuntu
```

当同时指定了 `--blkio-weight` 与 `--blkio-weight-device` 时，Docker 会使用 `--blkio-weight` 作为默认权重，并通过 `--blkio-weight-device` 为特定设备覆盖默认值。如下示例设置默认权重为 `300`，并将 `/dev/sda` 的权重覆盖为 `200`：

```console
$ docker run -it \
    --blkio-weight 300 \
    --blkio-weight-device "/dev/sda:200" \
    ubuntu
```

`--device-read-bps` 用于限制从设备的读取速率（字节/秒）。例如，将从 `/dev/sda` 的读取限制为 `1mb/s`：

```console
$ docker run -it --device-read-bps /dev/sda:1mb ubuntu
```

`--device-write-bps` 用于限制写入设备的速率（字节/秒）。例如，将写入 `/dev/sda` 的速率限制为 `1mb/s`：

```console
$ docker run -it --device-write-bps /dev/sda:1mb ubuntu
```

上述两个标志的取值格式均为 `<device-path>:<limit>[unit]`，读写速率都必须为正整数，单位可为 `kb`、`mb`、`gb`。

`--device-read-iops` 用于限制从设备读取的 IOPS。例如，将从 `/dev/sda` 的读取限制为 `1000 IOPS`：

```console
$ docker run -it --device-read-iops /dev/sda:1000 ubuntu
```

`--device-write-iops` 用于限制写入设备的 IOPS。例如，将对 `/dev/sda` 的写入限制为 `1000 IOPS`：

```console
$ docker run -it --device-write-iops /dev/sda:1000 ubuntu
```

这两个标志均采用 `<device-path>:<limit>` 格式，读写 IOPS 必须为正整数。

## 附加用户组（additional groups）

```console
--group-add: Add additional groups to run as
```

默认情况下，容器进程会附带指定用户在系统中查到的补充组（supplementary groups）。如果需要在此基础上再增加更多用户组，可以使用该标志：

```console
$ docker run --rm --group-add audio --group-add nogroup --group-add 777 busybox id

uid=0(root) gid=0(root) groups=10(wheel),29(audio),99(nogroup),777
```

## 运行时特权与 Linux Capabilities

| 选项           | 描述                                                                 |
|:---------------|:--------------------------------------------------------------------|
| `--cap-add`    | 增加 Linux capabilities                                             |
| `--cap-drop`   | 移除 Linux capabilities                                             |
| `--privileged` | 为容器授予扩展权限                                                  |
| `--device=[]`  | 在不使用 `--privileged` 的前提下，将设备映射进容器                   |

默认情况下，Docker 容器是“非特权（unprivileged）”的。例如，容器内默认不能再运行 Docker 守护进程。这是因为默认情况下容器不可访问任何设备；而“特权（privileged）”容器则被授予访问所有设备的权限（详见 [cgroups devices 文档](https://www.kernel.org/doc/Documentation/cgroup-v1/devices.txt)）。

`--privileged` 会为容器授予所有 capabilities。当执行 `docker run --privileged` 时，Docker 会开启对宿主设备的完全访问权限，并根据需要重新配置 AppArmor 或 SELinux，使容器几乎拥有与宿主上普通进程相同的权限。请谨慎使用该标志。关于 `--privileged` 的更多信息，请参见 [`docker run` 参考](https://docs.docker.com/reference/cli/docker/container/run/#privileged)。

如果只想限制性地开放对某些设备的访问，可使用 `--device` 标志，指定一个或多个在容器内可见的设备：

```console
$ docker run --device=/dev/snd:/dev/snd ...
```

默认情况下，容器对这些设备拥有 `read`、`write`、`mknod` 权限。你可以通过在每个 `--device` 后追加第三段 `:rwm` 来覆盖默认权限：

```console
$ docker run --device=/dev/sda:/dev/xvdc --rm -it ubuntu fdisk  /dev/xvdc

Command (m for help): q
$ docker run --device=/dev/sda:/dev/xvdc:r --rm -it ubuntu fdisk  /dev/xvdc
You will not be able to write the partition table.

Command (m for help): q

$ docker run --device=/dev/sda:/dev/xvdc:w --rm -it ubuntu fdisk  /dev/xvdc
    crash....

$ docker run --device=/dev/sda:/dev/xvdc:m --rm -it ubuntu fdisk  /dev/xvdc
fdisk: unable to open /dev/xvdc: Operation not permitted
```

除了 `--privileged` 外，你还可以通过 `--cap-add` 与 `--cap-drop` 精细控制容器的 capabilities。Docker 默认保留一组 capabilities，下面的表格列出了默认允许且可被移除的 capabilities：

| Capability Key        | 能力说明                                                                                                        |
|:----------------------|:----------------------------------------------------------------------------------------------------------------|
| AUDIT_WRITE           | 向内核审计日志写入记录。                                                                                        |
| CHOWN                 | 任意修改文件 UID 与 GID（参见 chown(2)）。                                                                      |
| DAC_OVERRIDE          | 绕过文件读、写、执行权限检查。                                                                                  |
| FOWNER                | 绕过需要进程的文件系统 UID 与文件所有者 UID 匹配的权限检查。                                                    |
| FSETID                | 当文件被修改时，不清除 setuid 与 setgid 位。                                                                    |
| KILL                  | 绕过发送信号时的权限检查。                                                                                      |
| MKNOD                 | 使用 mknod(2) 创建特殊文件。                                                                                    |
| NET_BIND_SERVICE      | 绑定到 1024 以下的特权端口。                                                                                    |
| NET_RAW               | 使用 RAW 与 PACKET 套接字。                                                                                      |
| SETFCAP               | 设置文件 capabilities。                                                                                          |
| SETGID                | 任意操作进程 GID 及附加 GID 列表。                                                                              |
| SETPCAP               | 修改进程 capabilities。                                                                                          |
| SETUID                | 任意操作进程 UID。                                                                                              |
| SYS_CHROOT            | 使用 chroot(2)，修改根目录。                                                                                     |

下表列出了默认不授予、但可以通过 `--cap-add` 添加的 capabilities：

| Capability Key        | 能力说明                                                                                                        |
|:----------------------|:----------------------------------------------------------------------------------------------------------------|
| AUDIT_CONTROL         | 启用/禁用内核审计；修改审计过滤规则；查询审计状态与过滤规则。                                                   |
| AUDIT_READ            | 允许通过多播 netlink socket 读取审计日志。                                                                      |
| BLOCK_SUSPEND         | 允许阻止系统挂起。                                                                                              |
| BPF                   | 允许创建 BPF 映射、加载 BTF、获取 BPF JIT 代码等。                                                               |
| CHECKPOINT_RESTORE    | 允许执行 checkpoint/restore 相关操作（内核 5.9 引入）。                                                          |
| DAC_READ_SEARCH       | 绕过文件读权限检查，以及目录读与执行权限检查。                                                                  |
| IPC_LOCK              | 锁定内存（mlock(2)、mlockall(2)、mmap(2)、shmctl(2)）。                                                          |
| IPC_OWNER             | 绕过对 System V IPC 对象操作的权限检查。                                                                        |
| LEASE                 | 对任意文件建立租约（参见 fcntl(2)）。                                                                           |
| LINUX_IMMUTABLE       | 设置 i-node 标志：FS_APPEND_FL 与 FS_IMMUTABLE_FL。                                                             |
| MAC_ADMIN             | 允许更改 MAC 配置或状态。实现于 Smack LSM。                                                                     |
| MAC_OVERRIDE          | 覆盖强制访问控制（MAC）。实现于 Smack LSM。                                                                      |
| NET_ADMIN             | 执行多种与网络相关的管理操作。                                                                                   |
| NET_BROADCAST         | 允许进行 socket 广播，或监听多播。                                                                              |
| PERFMON               | 允许使用 perf_events、i915_perf 等子系统进行高权限性能/可观测性操作。                                           |
| SYS_ADMIN             | 执行系统管理类操作。                                                                                             |
| SYS_BOOT              | 使用 reboot(2)、kexec_load(2)，重启并加载新内核供后续执行。                                                      |
| SYS_MODULE            | 加载与卸载内核模块。                                                                                             |
| SYS_NICE              | 提升进程 nice 值（nice(2)、setpriority(2)），或修改任意进程的 nice 值。                                         |
| SYS_PACCT             | 使用 acct(2)，开启/关闭进程会计。                                                                                |
| SYS_PTRACE            | 使用 ptrace(2) 跟踪任意进程。                                                                                    |
| SYS_RAWIO             | 执行 I/O 端口操作（iopl(2)、ioperm(2)）。                                                                        |
| SYS_RESOURCE          | 覆盖资源限制。                                                                                                   |
| SYS_TIME              | 设置系统时间（settimeofday(2)、stime(2)、adjtimex(2)）；设置硬件时钟。                                         |
| SYS_TTY_CONFIG        | 使用 vhangup(2)；在虚拟终端上执行特权 ioctl(2) 操作。                                                            |
| SYSLOG                | 执行特权 syslog(2) 操作。                                                                                        |
| WAKE_ALARM            | 触发唤醒系统的事件。                                                                                             |

更多参考请见 [capabilities(7) - Linux man page](https://man7.org/linux/man-pages/man7/capabilities.7.html) 与 [Linux 内核源码](https://github.com/torvalds/linux/blob/124ea650d3072b005457faed69909221c2905a1f/include/uapi/linux/capability.h)。

`--cap-add` 与 `--cap-drop` 都支持值为 `ALL`。例如，仅排除 `MKNOD`：

```console
$ docker run --cap-add=ALL --cap-drop=MKNOD ...
```

`--cap-add` 与 `--cap-drop` 接受带或不带 `CAP_` 前缀的 capability 名称，以下示例等价：

```console
$ docker run --cap-add=SYS_ADMIN ...
$ docker run --cap-add=CAP_SYS_ADMIN ...
```

若需要操作网络栈，不要直接使用 `--privileged`，而应使用 `--cap-add=NET_ADMIN` 来修改网络接口：

```console
$ docker run -it --rm  ubuntu:24.04 ip link add dummy0 type dummy

RTNETLINK answers: Operation not permitted

$ docker run -it --rm --cap-add=NET_ADMIN ubuntu:24.04 ip link add dummy0 type dummy
```

若要挂载基于 FUSE 的文件系统，需要同时结合 `--cap-add` 与 `--device`：

```console
$ docker run --rm -it --cap-add SYS_ADMIN sshfs sshfs sven@10.10.10.20:/home/sven /mnt

fuse: failed to open /dev/fuse: Operation not permitted

$ docker run --rm -it --device /dev/fuse sshfs sshfs sven@10.10.10.20:/home/sven /mnt

fusermount: mount failed: Operation not permitted

$ docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse sshfs

# sshfs sven@10.10.10.20:/home/sven /mnt
The authenticity of host '10.10.10.20 (10.10.10.20)' can't be established.
ECDSA key fingerprint is 25:34:85:75:25:b0:17:46:05:19:04:93:b5:dd:5f:c6.
Are you sure you want to continue connecting (yes/no)? yes
sven@10.10.10.20's password:

root@30aa0cfaf1b5:/# ls -la /mnt/src/docker

total 1516
drwxrwxr-x 1 1000 1000   4096 Dec  4 06:08 .
drwxrwxr-x 1 1000 1000   4096 Dec  4 11:46 ..
-rw-rw-r-- 1 1000 1000     16 Oct  8 00:09 .dockerignore
-rwxrwxr-x 1 1000 1000    464 Oct  8 00:09 .drone.yml
drwxrwxr-x 1 1000 1000   4096 Dec  4 06:11 .git
-rw-rw-r-- 1 1000 1000    461 Dec  4 06:08 .gitignore
....
```

默认的 seccomp 配置会根据所选 capabilities 自动调整，以允许这些能力所需的系统调用，通常无需额外修改。

## 覆盖镜像默认行为

当你通过 [Dockerfile](https://docs.docker.com/reference/dockerfile/) 构建镜像或 commit 镜像时，可以设置若干默认参数，这些参数会在镜像以容器形式启动时生效。运行镜像时，可以通过 `docker run` 的标志覆盖这些默认设置。

- [默认入口（entrypoint）](#default-entrypoint)
- [默认命令与参数](#default-command-and-options)
- [暴露端口](#exposed-ports)
- [环境变量](#environment-variables)
- [健康检查](#healthchecks)
- [用户](#user)
- [工作目录](#working-directory)

### 默认命令与参数

`docker run` 的语法支持为入口（entrypoint）传递可选的命令与参数，分别对应 `[COMMAND]` 与 `[ARG...]`：

```console
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

该命令部分是可选的，因为镜像的创建者可能已通过 Dockerfile 的 `CMD` 指令提供了默认 `COMMAND`。当运行容器时，只需在命令行中指定新的 `COMMAND` 即可覆盖 Dockerfile 中的 `CMD`。

如果镜像同时指定了 `ENTRYPOINT`，那么 `CMD` 或你传入的 `[COMMAND]` 会作为参数追加到 `ENTRYPOINT` 之后。

### 默认入口（entrypoint）

```text
--entrypoint="": 覆盖镜像设置的默认 entrypoint
```

Entrypoint 是指运行容器时默认被调用的可执行文件。镜像的默认 entrypoint 由 Dockerfile 的 `ENTRYPOINT` 指令定义。它与默认命令类似，但差异在于：覆盖 entrypoint 需要显式传入标志，而覆盖默认命令只需使用位置参数。EntryPoint 用于定义容器的默认行为——理想情况下，你可以“像运行该二进制一样”运行容器；当然也可以通过命令行参数传入更多选项。但有时你可能希望在容器中运行其他程序，这时就可以在运行时通过 `--entrypoint` 覆盖默认 entrypoint。

`--entrypoint` 接受一个字符串参数，表示容器启动时要调用的二进制名称或路径。下面示例展示了在默认会自动运行其它二进制（如 `/usr/bin/redis-server`）的镜像中，如何启动一个 Bash：

```console
$ docker run -it --entrypoint /bin/bash example/redis
```

下面的示例展示了如何通过位置参数为自定义 entrypoint 传入额外参数：

```console
$ docker run -it --entrypoint /bin/bash example/redis -c ls -l
$ docker run -it --entrypoint /usr/bin/redis-cli example/redis --help
```

你可以通过传入空字符串来重置容器的 entrypoint，例如：

```console
$ docker run -it --entrypoint="" mysql bash
```

> [!NOTE]
> 传入 `--entrypoint` 会清空镜像中设置的默认命令，即 Dockerfile 中的 `CMD` 会被忽略。

### 暴露端口

默认情况下，运行容器时不会将容器内的任何端口暴露到宿主机。因此，你无法直接访问容器内可能监听的端口。若要从宿主访问容器端口，需要将端口“发布（publish）”。

可以使用 `-P` 或 `-p` 来发布端口：

- `-P`（或 `--publish-all`）会将镜像声明的所有暴露端口映射到宿主机上的随机端口。

  `-P` 只会发布显式声明为“暴露”的端口，这些端口来自 Dockerfile 的 `EXPOSE` 指令或 `docker run` 的 `--expose` 标志。

- `-p`（或 `--publish`）允许你显式地映射容器中的单个端口或端口范围到宿主。

容器内服务监听的端口与对外发布的端口无需相同。例如，容器内的 HTTP 服务可能监听 80 端口，而在运行时被绑定到宿主的 42800 端口。你可以通过 `docker port` 查看宿主端口与容器端口的映射关系。

### 环境变量

在创建 Linux 容器时，Docker 会自动设置一些环境变量；创建 Windows 容器时，Docker 不会自动设置任何环境变量。

Linux 容器中默认设置的环境变量包括：

| 变量       | 值                                                                                                     |
|:-----------|:--------------------------------------------------------------------------------------------------------|
| `HOME`     | 基于 `USER` 的值进行设置                                                                                |
| `HOSTNAME` | 容器的主机名                                                                                            |
| `PATH`     | 包含常见路径，如 `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`                         |
| `TERM`     | 若为容器分配了伪终端，则为 `xterm`                                                                      |

此外，你可以通过一个或多个 `-e` 标志为容器设置任意环境变量。你甚至可以覆盖上述变量，或覆盖镜像在 Dockerfile 中通过 `ENV` 指令定义的变量。

当只指定变量名而不指定值时，宿主机当前的环境变量值会被传递进容器：

```console
$ export today=Wednesday
$ docker run -e "deep=purple" -e today --rm alpine env

PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=d2219b854598
deep=purple
today=Wednesday
HOME=/root
```

```powershell
PS C:\> docker run --rm -e "foo=bar" microsoft/nanoserver cmd /s /c set
ALLUSERSPROFILE=C:\ProgramData
APPDATA=C:\Users\ContainerAdministrator\AppData\Roaming
CommonProgramFiles=C:\Program Files\Common Files
CommonProgramFiles(x86)=C:\Program Files (x86)\Common Files
CommonProgramW6432=C:\Program Files\Common Files
COMPUTERNAME=C2FAEFCC8253
ComSpec=C:\Windows\system32\cmd.exe
foo=bar
LOCALAPPDATA=C:\Users\ContainerAdministrator\AppData\Local
NUMBER_OF_PROCESSORS=8
OS=Windows_NT
Path=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Users\ContainerAdministrator\AppData\Local\Microsoft\WindowsApps
PATHEXT=.COM;.EXE;.BAT;.CMD
PROCESSOR_ARCHITECTURE=AMD64
PROCESSOR_IDENTIFIER=Intel64 Family 6 Model 62 Stepping 4, GenuineIntel
PROCESSOR_LEVEL=6
PROCESSOR_REVISION=3e04
ProgramData=C:\ProgramData
ProgramFiles=C:\Program Files
ProgramFiles(x86)=C:\Program Files (x86)
ProgramW6432=C:\Program Files
PROMPT=$P$G
PUBLIC=C:\Users\Public
SystemDrive=C:
SystemRoot=C:\Windows
TEMP=C:\Users\ContainerAdministrator\AppData\Local\Temp
TMP=C:\Users\ContainerAdministrator\AppData\Local\Temp
USERDOMAIN=User Manager
USERNAME=ContainerAdministrator
USERPROFILE=C:\Users\ContainerAdministrator
windir=C:\Windows
```

### 健康检查（Healthchecks）

`docker run` 提供以下标志用于控制容器健康检查的参数：

| 选项                      | 描述                                                                                 |
|:--------------------------|:--------------------------------------------------------------------------------------|
| `--health-cmd`            | 用于探测健康状况的命令                                                                 |
| `--health-interval`       | 每次探测之间的时间间隔                                                                 |
| `--health-retries`        | 判定为不健康所需的连续失败次数                                                         |
| `--health-timeout`        | 单次探测允许的最长执行时间                                                             |
| `--health-start-period`   | 容器启动后进入健康检查前的“冷启动”时间窗口                                             |
| `--health-start-interval` | 在“冷启动”时间窗口内的探测间隔                                                         |
| `--no-healthcheck`        | 禁用镜像中设置的 `HEALTHCHECK`                                                          |

示例：

```console
$ docker run --name=test -d \
    --health-cmd='stat /etc/passwd || exit 1' \
    --health-interval=2s \
    busybox sleep 1d
$ sleep 2; docker inspect --format='{{.State.Health.Status}}' test
healthy
$ docker exec test rm /etc/passwd
$ sleep 2; docker inspect --format='{{json .State.Health}}' test
{
  "Status": "unhealthy",
  "FailingStreak": 3,
  "Log": [
    {
      "Start": "2016-05-25T17:22:04.635478668Z",
      "End": "2016-05-25T17:22:04.7272552Z",
      "ExitCode": 0,
      "Output": "  File: /etc/passwd\n  Size: 334       \tBlocks: 8          IO Block: 4096   regular file\nDevice: 32h/50d\tInode: 12          Links: 1\nAccess: (0664/-rw-rw-r--)  Uid: (    0/    root)   Gid: (    0/    root)\nAccess: 2015-12-05 22:05:32.000000000\nModify: 2015..."
    },
    {
      "Start": "2016-05-25T17:22:06.732900633Z",
      "End": "2016-05-25T17:22:06.822168935Z",
      "ExitCode": 0,
      "Output": "  File: /etc/passwd\n  Size: 334       \tBlocks: 8          IO Block: 4096   regular file\nDevice: 32h/50d\tInode: 12          Links: 1\nAccess: (0664/-rw-rw-r--)  Uid: (    0/    root)   Gid: (    0/    root)\nAccess: 2015-12-05 22:05:32.000000000\nModify: 2015..."
    },
    {
      "Start": "2016-05-25T17:22:08.823956535Z",
      "End": "2016-05-25T17:22:08.897359124Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    },
    {
      "Start": "2016-05-25T17:22:10.898802931Z",
      "End": "2016-05-25T17:22:10.969631866Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    },
    {
      "Start": "2016-05-25T17:22:12.971033523Z",
      "End": "2016-05-25T17:22:13.082015516Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    }
  ]
}
```

容器的健康状态也会显示在 `docker ps` 的输出中。

### 用户（User）

容器内的默认用户是 `root`（uid = 0）。你可以在 Dockerfile 中通过 `USER` 指令设置默认用户。启动容器时，可通过 `-u` 覆盖该设置。

```text
-u="", --user="": 设置运行指定命令时的用户名或 UID，可选地还可设置组名或 GID。
```

以下形式均为有效写法：

```text
--user=[ user | user:group | uid | uid:gid | user:gid | uid:group ]
```

> [!NOTE]
> 若传入的是数字用户 ID，范围必须在 0-2147483647。若传入用户名，则该用户必须存在于容器内。

### 工作目录（Working directory）

在容器内运行可执行文件时，默认工作目录为根目录（`/`）。镜像的默认工作目录由 Dockerfile 的 `WORKDIR` 设置。你可以使用 `-w`（或 `--workdir`）覆盖镜像的默认工作目录：

```text
$ docker run --rm -w /my/workdir alpine pwd
/my/workdir
```

如果该目录在容器内不存在，将会被自动创建。
