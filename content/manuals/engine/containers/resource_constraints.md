---
title: 资源限制
weight: 30
description: 为容器指定运行时选项
keywords: docker, daemon, configuration, runtime
aliases:
  - /engine/admin/resource_constraints/
  - /config/containers/resource_constraints/
---

默认情况下，容器没有资源限制，可以在宿主机内核调度器允许的范围内尽可能多地使用给定资源。Docker 通过 `docker run` 的运行时标志提供了控制容器可用内存与 CPU 的方式。本节说明何时应该设置这些限制以及设置后的影响。

其中许多功能需要内核支持相应的 Linux 能力。你可以使用 [`docker info`](/reference/cli/docker/system/info.md) 命令检查是否受支持。如果内核禁用了某些能力，你可能会在输出末尾看到类似的警告：

```console
WARNING: No swap limit support
```

请参考所用操作系统文档以启用这些能力。更多信息见 [Docker Engine 故障排查指南](../daemon/troubleshoot.md#kernel-cgroup-swap-limit-capabilities)。

## 内存

## 了解内存耗尽的风险

务必避免运行中的容器占用过多宿主机内存。在 Linux 主机上，如果内核检测到内存不足以执行关键系统功能，会抛出 `OOME`（Out Of Memory Exception）并开始杀死进程以释放内存。任何进程都有可能被杀，包括 Docker 以及其他关键应用。如果被杀的是关键进程，整个系统可能因此宕机。

为降低风险，Docker 会下调 Docker 守护进程的 OOM 被杀优先级，使其比系统上其他进程更不容易被杀；容器自身的 OOM 优先级不会调整。这意味着更可能被杀的是单个容器，而不是 Docker 守护进程或其他系统进程。请不要通过为守护进程或容器设置极端负值的 `--oom-score-adj`，或在容器上设置 `--oom-kill-disable` 来规避这些保护措施。

关于 Linux 内核 OOM 管理的更多信息，参见 [Out of Memory Management](https://www.kernel.org/doc/gorman/html/understand/understand016.html)。

你可以通过以下方式降低 OOME 导致系统不稳定的风险：

- 在上线之前通过测试了解应用的内存需求。
- 确保应用仅在资源充足的主机上运行。
- 按下文所述限制容器可使用的内存量。
- 为 Docker 主机配置交换分区（swap）时要谨慎。Swap 比内存慢，但能在系统内存不足时提供缓冲。
- 考虑将容器改为 [服务](/manuals/engine/swarm/services.md)，并使用服务级约束与节点标签，确保仅在内存充足的主机上运行。

### 限制容器的内存使用

Docker 支持设置硬限制与软限制：

- 硬限制：容器使用的内存不会超过固定上限。
- 软限制：在满足特定条件前（如内核检测到宿主机内存紧张或争用）容器可按需使用更多内存。

部分选项单独使用或组合使用时的效果不同。

多数选项接受一个正整数，并可使用 `b`、`k`、`m`、`g` 后缀表示字节、KB、MB、GB。

| 选项                   | 说明                                                                                                                                                                                                                                                                                                                                                                                           |
| :--------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-m` 或 `--memory=`     | 容器可使用的最大内存。若设置该选项，最小允许值为 `6m`（6 MB），即至少需设置为 6 MB。                                                                                                                                                                                                                                                                                                         |
| `--memory-swap`\*      | 容器允许写入磁盘的 swap 数量。详见 [`--memory-swap` details](#--memory-swap-details)。                                                                                                                                                                                                                                                                                                         |
| `--memory-swappiness`  | 默认情况下，宿主机内核可以将容器使用的部分匿名页换出（swap）。可将 `--memory-swappiness` 设为 0 到 100 之间的值以调节该比例。详见 [`--memory-swappiness` details](#--memory-swappiness-details)。                                                                                                                                                               |
| `--memory-reservation` | 指定一个小于 `--memory` 的软限制；当 Docker 检测到宿主机内存紧张或争用时生效。若使用 `--memory-reservation`，其值必须低于 `--memory` 才会优先生效。由于是软限制，不能保证容器不会超过该值。                                                                                                                                                                                      |
| `--kernel-memory`      | 容器可使用的最大内核内存。最小允许值为 `6m`。由于内核内存不可被换出，当容器的内核内存被限制时，可能阻塞宿主资源，从而影响宿主机及其他容器。详见 [`--kernel-memory` details](#--kernel-memory-details)。                                                                                                                                                                          |
| `--oom-kill-disable`   | 默认情况下发生 OOM 时，内核会在容器内杀死进程。若要改变此行为，使用 `--oom-kill-disable`。仅当同时设置了 `-m/--memory` 时才建议禁用 OOM killer。若未设置 `-m`，宿主机可能耗尽内存，内核可能需要杀死宿主系统进程以释放内存。                                                                                                                                                             |

关于 cgroups 与内存的更多信息，参见 [Memory Resource Controller](https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt)。

### `--memory-swap` details

`--memory-swap` 是一个修饰标志，仅在同时设置了 `--memory` 时才有意义。启用 swap 允许容器在用尽可用内存后，将超额内存需求写入磁盘。频繁发生内存换出会带来性能损失。

不同取值会带来不同效果：

- 若 `--memory-swap` 为正整数，则必须同时设置 `--memory` 与 `--memory-swap`。`--memory-swap` 表示内存+swap 的总量，而 `--memory` 表示物理内存的上限。例如 `--memory="300m"` 且 `--memory-swap="1g"`，则容器可用 300m 内存与 700m（`1g - 300m`）的 swap。

- 若 `--memory-swap` 设为 `0`，该设置会被忽略，等同未设置。

- 若 `--memory-swap` 与 `--memory` 相同，且 `--memory` 为正整数，则**容器无法使用 swap**。见 [Prevent a container from using swap](#prevent-a-container-from-using-swap)。

- 若未设置 `--memory-swap`，且设置了 `--memory`，并且宿主机配置了 swap，则容器可使用与 `--memory` 相同数量的 swap。例如 `--memory="300m"` 且未设置 `--memory-swap`，则容器总计可使用 600m（内存+swap）。

- 若显式设置 `--memory-swap=-1`，容器可使用无限制的 swap（受宿主机可用量上限约束）。

- 在容器内，`free` 等工具显示的是宿主机可用的 swap，而非容器内可用的 swap。不要依赖 `free` 这类工具的输出判断是否存在 swap。

#### Prevent a container from using swap

当 `--memory` 与 `--memory-swap` 设置为相同的值时，容器将无法使用任何 swap。原因在于 `--memory-swap` 指定的是内存与 swap 的合计上限，而 `--memory` 指定的是仅物理内存的上限。

### `--memory-swappiness` details

- 值为 0：关闭匿名页换出。
- 值为 100：将所有匿名页标记为可换出。
- 默认不设置时，继承宿主机的配置。

### `--kernel-memory` details

内核内存限制是相对于容器整体内存配额来表达的。考虑以下场景：

- 不限制总内存，不限制内核内存：默认行为。
- 不限制总内存，限制内核内存：适用于所有 cgroup 需要的内存总量大于宿主机实际内存的情况。可将内核内存限制在宿主机可用范围内，需求更多内核内存的容器需要等待。
- 限制总内存，不限制内核内存：总体内存受限，但内核内存不受限。
- 限制总内存，限制内核内存：同时限制用户态与内核内存有助于定位与内存相关的问题。如果某容器在任一类型的内存上使用异常，将在不影响其他容器或宿主机的情况下触发 OOM。在此设置下，若内核内存上限低于用户态内存上限，则耗尽内核内存会导致 OOM；反之则不会因内核内存限制触发 OOM。

启用内核内存限制后，宿主机会按进程维度跟踪“最高水位线”等统计信息，从而定位是哪些进程（此处即容器）消耗了过多内存。可在宿主机查看 `/proc/<PID>/status` 获取每个进程的数据。

## CPU

默认情况下，每个容器对宿主机 CPU 周期的使用不受限。你可以通过多种约束限制某个容器对 CPU 周期的使用。多数用户会使用并配置[默认 CFS 调度器](#configure-the-default-cfs-scheduler)，也可以配置[实时调度器](#configure-the-real-time-scheduler)。

### Configure the default CFS scheduler

Linux 的 CFS（Completely Fair Scheduler）是常规进程的 CPU 调度器。可以通过若干运行时标志配置容器可用的 CPU 资源额度。设置后，Docker 会相应调整宿主机上该容器 cgroup 的配置。

| 选项                   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| :--------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--cpus=<value>`       | 指定容器可使用的 CPU 资源份额。例如宿主机有 2 个 CPU，设置 `--cpus="1.5"` 则最多保证 1.5 个 CPU 的算力。这等同于设置 `--cpu-period="100000"` 与 `--cpu-quota="150000"`。                                                                                                                                                                                                                                                                                                                                                                                              |
| `--cpu-period=<value>` | 指定 CFS 调度周期，与 `--cpu-quota` 搭配使用。默认 100000 微秒（100 毫秒）。大多数情况下无需修改；对多数用例而言，使用 `--cpus` 更为便利。                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `--cpu-quota=<value>`  | 为容器设置 CFS 配额。表示每个 `--cpu-period` 周期内，在被限流前容器可用的微秒数，因此相当于一个上限。多数用例优先使用更方便的 `--cpus`。                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `--cpuset-cpus`        | 限制容器可使用的 CPU 或核心编号。当有多个 CPU 时，使用逗号分隔列表或连字符范围指定。首个 CPU 编号为 0。例如 `0-3`（使用第 1-4 个 CPU）或 `1,3`（使用第 2 与第 4 个 CPU）。                                                                                                                                                                                                                                                                                                                                                                                             |
| `--cpu-shares`         | 通过设置大于或小于默认值 1024 的权重，提升或降低容器可分得的 CPU 周期比例。仅在 CPU 周期受限时生效；当资源充足时，所有容器按需使用 CPU，因此这是软限制。`--cpu-shares` 不会阻止 Swarm 模式下的调度；它只是为可用的 CPU 周期分配优先级，不保证或预留特定的 CPU 额度。                                                                                                                                                                                                                                                        |

若宿主机仅有 1 个 CPU，以下命令都能保证容器每秒最多使用 50% 的 CPU：

```console
$ docker run -it --cpus=".5" ubuntu /bin/bash
```

这等同于手动指定 `--cpu-period` 与 `--cpu-quota`：

```console
$ docker run -it --cpu-period=100000 --cpu-quota=50000 ubuntu /bin/bash
```

### Configure the real-time scheduler

对于无法使用 CFS 的任务，可以将容器配置为使用实时调度器。在[配置 Docker 守护进程](#configure-the-docker-daemon)或[配置单个容器](#configure-individual-containers)之前，需要先[确保宿主机内核已正确配置](#configure-the-host-machines-kernel)。

> [!WARNING]
>
> CPU 调度与优先级是高级的内核级特性。多数用户无需修改默认值。错误的配置可能导致宿主系统不稳定甚至不可用。

#### Configure the host machine's kernel

通过运行 `zcat /proc/config.gz | grep CONFIG_RT_GROUP_SCHED` 或检查 `/sys/fs/cgroup/cpu.rt_runtime_us` 是否存在，确认 Linux 内核启用了 `CONFIG_RT_GROUP_SCHED`。关于实时调度器的配置方法，请参考所用操作系统的文档。

#### Configure the Docker daemon

若要让容器使用实时调度器，需以 `--cpu-rt-runtime` 参数启动 Docker 守护进程，用于指定每个调度周期为实时任务保留的微秒数。例如在默认 1,000,000 微秒（1 秒）周期下，设置 `--cpu-rt-runtime=950000` 表示实时任务每周期最多运行 950,000 微秒，至少保留 50,000 微秒给非实时任务。若系统使用 `systemd`，可通过创建 `docker` 服务的 unit 文件来持久化该配置。例如，参考如何通过 [systemd unit 文件](../daemon/proxy.md#systemd-unit-file) 为守护进程配置代理的说明。

#### Configure individual containers

在使用 `docker run` 启动容器时，可以通过多个标志控制容器的 CPU 优先级。具体取值请参考操作系统文档或 `ulimit` 命令。

| 选项                       | 说明                                                                                                                                                                                      |
| :------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--cap-add=sys_nice`       | 赋予容器 `CAP_SYS_NICE` 能力，使其可提升进程 nice 值、设置实时调度策略、设置 CPU 亲和性等。                                                                                                     |
| `--cpu-rt-runtime=<value>` | 在守护进程实时调度周期内，容器以实时优先级运行的最大微秒数。需要同时配合 `--cap-add=sys_nice`。                                                                                                  |
| `--ulimit rtprio=<value>`  | 容器允许的最大实时优先级，同样需要 `--cap-add=sys_nice`。                                                                                                                                     |

下面的示例命令在 `debian:jessie` 容器上同时设置上述三个标志：

```console
$ docker run -it \
    --cpu-rt-runtime=950000 \
    --ulimit rtprio=99 \
    --cap-add=sys_nice \
    debian:jessie
```

如果内核或 Docker 守护进程配置不当，将会报错。

## GPU

### 访问 NVIDIA GPU

#### 先决条件

访问 [NVIDIA 驱动下载页面](https://www.nvidia.com/Download/index.aspx) 获取并安装合适的驱动。安装完成后请重启系统。

确认 GPU 正常运行且可访问。

#### 安装 nvidia-container-toolkit

按照 NVIDIA Container Toolkit 的[安装指南](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)进行安装。

#### 暴露可用的 GPU

启动容器时添加 `--gpus` 标志以启用 GPU 资源，并指定使用的 GPU 数量。例如：

```console
$ docker run -it --rm --gpus all ubuntu nvidia-smi
```

上述命令会暴露所有可用 GPU，并输出类似结果：

```bash
+-------------------------------------------------------------------------------+
| NVIDIA-SMI 384.130            	Driver Version: 384.130               	|
|-------------------------------+----------------------+------------------------+
| GPU  Name 	   Persistence-M| Bus-Id    	Disp.A | Volatile Uncorr. ECC   |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M.   |
|===============================+======================+========================|
|   0  GRID K520       	Off  | 00000000:00:03.0 Off |                  N/A      |
| N/A   36C	P0    39W / 125W |  	0MiB /  4036MiB |      0%  	Default |
+-------------------------------+----------------------+------------------------+
+-------------------------------------------------------------------------------+
| Processes:                                                       GPU Memory   |
|  GPU   	PID   Type   Process name                         	Usage  	|
|===============================================================================|
|  No running processes found                                                   |
+-------------------------------------------------------------------------------+
```

也可以使用 `device` 选项按设备指定 GPU，例如：

```console
$ docker run -it --rm --gpus device=GPU-3a23c669-1f69-c64e-cf85-44e9b07e7a2a ubuntu nvidia-smi
```

上例仅暴露该特定 GPU。

```console
$ docker run -it --rm --gpus '"device=0,2"' ubuntu nvidia-smi
```

上例暴露第 1 与第 3 个 GPU。

> [!NOTE]
>
> 仅在运行单个引擎的系统上才能访问 NVIDIA GPU。

#### 设置 NVIDIA 能力

你也可以手动设置能力。例如在 Ubuntu 上可以运行：

```console
$ docker run --gpus 'all,capabilities=utility' --rm ubuntu nvidia-smi
```

这会启用 `utility` 驱动能力，将 `nvidia-smi` 工具加入容器。

包括能力在内的其他配置也可以通过镜像中的环境变量进行设置。可在 [nvidia-container-toolkit 文档](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html)中查看支持的变量，并在 Dockerfile 中进行设置。

也可以直接使用 CUDA 镜像，这些变量会自动设置。参见官方 NGC 目录的 [CUDA 镜像](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/cuda)。
