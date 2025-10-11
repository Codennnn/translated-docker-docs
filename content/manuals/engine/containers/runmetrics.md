---
description: 了解如何观测运行中容器及其各类指标
keywords: docker, metrics, CPU, memory, disk, IO, run, runtime, stats
title: 运行时指标
weight: 50
aliases:
  - /articles/runmetrics/
  - /engine/articles/run_metrics/
  - /engine/articles/runmetrics/
  - /engine/admin/runmetrics/
  - /config/containers/runmetrics/
---

## Docker 统计 {#docker-stats}

使用 `docker stats` 可以实时查看容器的运行指标，包括 CPU、内存使用与上限、网络 IO 等。

以下为 `docker stats` 的示例输出：

```console
$ docker stats redis1 redis2

CONTAINER           CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O
redis1              0.07%               796 KB / 64 MB        1.21%               788 B / 648 B       3.568 MB / 512 KB
redis2              0.07%               2.746 MB / 64 MB      4.29%               1.266 KB / 648 B    12.4 MB / 0 B
```

更多用法参见 [`docker stats`](/reference/cli/docker/container/stats.md) 参考页。

## 控制组（cgroups） {#control-groups}

Linux 容器依赖于 [control groups](https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt)（cgroups）。它不仅用于管理进程组，也会暴露 CPU、内存、块 I/O 等使用指标，还可获取网络使用数据。该机制既适用于“纯” LXC 容器，也适用于 Docker 容器。

cgroups 通过伪文件系统暴露。在现代发行版中，该文件系统位于 `/sys/fs/cgroup`。该目录下会有多个子目录，如 `devices`、`freezer`、`blkio` 等，每个子目录对应一个不同的 cgroup 层级。

在较旧的系统中，cgroups 可能挂载在 `/cgroup`，且不区分层级。此时你将看到一堆文件，以及可能对应已存在容器的一些目录。

要确定 cgroups 的挂载位置，可运行：

```console
$ grep cgroup /proc/mounts
```

### 枚举 cgroups {#enumerate-cgroups}

v1 与 v2 在文件布局上有显著差异。

若系统存在 `/sys/fs/cgroup/cgroup.controllers` 则为 v2，否则为 v1。请根据所用版本阅读对应小节。

以下发行版默认使用 cgroup v2：

- Fedora（自 31 起）
- Debian GNU/Linux（自 11 起）
- Ubuntu（自 21.10 起）

#### cgroup v1

可以查看 `/proc/cgroups`，以了解系统已知的控制组子系统、其所属层级以及包含的组数量。

也可以查看 `/proc/<pid>/cgroup` 以了解某个进程隶属哪些控制组。控制组以相对于层级挂载点根目录的路径形式显示：`/` 表示该进程尚未被分配到任何组；`/lxc/pumpkin` 表示该进程属于名为 `pumpkin` 的容器。

#### cgroup v2

在 cgroup v2 主机上，`/proc/cgroups` 的内容没有意义。可通过 `/sys/fs/cgroup/cgroup.controllers` 查看可用的控制器。

### 切换 cgroup 版本 {#changing-cgroup-version}

更改 cgroup 版本需要重启整个系统。

在基于 systemd 的系统上，可在内核命令行添加 `systemd.unified_cgroup_hierarchy=1` 以启用 cgroup v2。
若要恢复为 v1，设置 `systemd.unified_cgroup_hierarchy=0`。

如果系统提供 `grubby`（如 Fedora），可用以下命令修改内核命令行：

```console
$ sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
```

若无 `grubby`，编辑 `/etc/default/grub` 中的 `GRUB_CMDLINE_LINUX` 行并执行 `sudo update-grub`。

### 在 cgroup v2 上运行 Docker {#running-docker-on-cgroup-v2}

自 Docker 20.10 起支持 cgroup v2。
在 cgroup v2 上运行 Docker 还需满足以下条件：

- containerd：v1.4 或更高
- runc：v1.0.0-rc91 或更高
- 内核：v4.15 或更高（建议 v5.2+）

注意，cgroup v2 模式与 v1 略有不同：

- 默认 cgroup 驱动（`dockerd --exec-opt native.cgroupdriver`）：v2 为 `systemd`，v1 为 `cgroupfs`。
- 默认 cgroup 命名空间模式（`docker run --cgroupns`）：v2 为 `private`，v1 为 `host`。
- `docker run` 标志 `--oom-kill-disable` 与 `--kernel-memory` 在 v2 中被忽略。

### 查找指定容器的 cgroup {#find-the-cgroup-for-a-given-container}

每个容器在每个层级下都会创建一个 cgroup。在较老系统及较旧 LXC 用户态工具上，cgroup 名称即为容器名。较新版本的 LXC 工具中，cgroup 形式为 `lxc/<container_name>`。

在 Docker 中，cgroup 名称通常是容器的完整 ID（long ID）。若 `docker ps` 显示为 `ae836c95b4c3`，其 long ID 可能形如 `ae836c95b4c3c9e9179e0e91015512da89fdec91612f63cebae57df9a5444c79`。可通过 `docker inspect` 或 `docker ps --no-trunc` 查询。

综合起来，要查看某个 Docker 容器的内存相关指标，可以查看以下路径：

- `/sys/fs/cgroup/memory/docker/<longid>/` on cgroup v1, `cgroupfs` driver
- `/sys/fs/cgroup/memory/system.slice/docker-<longid>.scope/` on cgroup v1, `systemd` driver
- `/sys/fs/cgroup/docker/<longid>/` on cgroup v2, `cgroupfs` driver
- `/sys/fs/cgroup/system.slice/docker-<longid>.scope/` on cgroup v2, `systemd` driver

### 来自 cgroups 的指标：内存、CPU、块 I/O {#metrics-from-cgroups-memory-cpu-block-i-o}

> [!NOTE]
>
> 本节尚未针对 cgroup v2 更新。有关 v2 的更多信息，请参见[内核文档](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)。

针对每个子系统（内存、CPU、块 I/O），都会有一个或多个伪文件提供统计信息。

#### 内存指标：`memory.stat` {#memory-metrics-memory-stat}

内存相关指标位于 `memory` cgroup。该控制组会对主机内存使用进行非常细粒度的记账，因此会带来少量开销，许多发行版默认不启用。通常只需在内核命令行添加如下参数即可启用：
`cgroup_enable=memory swapaccount=1`。

指标位于伪文件 `memory.stat`，示例如下：

    cache 11492564992
    rss 1930993664
    mapped_file 306728960
    pgpgin 406632648
    pgpgout 403355412
    swap 0
    pgfault 728281223
    pgmajfault 1724
    inactive_anon 46608384
    active_anon 1884520448
    inactive_file 7003344896
    active_file 4489052160
    unevictable 32768
    hierarchical_memory_limit 9223372036854775807
    hierarchical_memsw_limit 9223372036854775807
    total_cache 11492564992
    total_rss 1930993664
    total_mapped_file 306728960
    total_pgpgin 406632648
    total_pgpgout 403355412
    total_swap 0
    total_pgfault 728281223
    total_pgmajfault 1724
    total_inactive_anon 46608384
    total_active_anon 1884520448
    total_inactive_file 7003344896
    total_active_file 4489052160
    total_unevictable 32768

前半部分（无 `total_` 前缀）仅包含该 cgroup 内进程的统计（不含子 cgroup）；后半部分（带 `total_` 前缀）则包含子 cgroup 的统计。

部分指标是“仪表值”（可增可减），例如 `swap` 表示 cgroup 成员当前使用的 swap 空间。另一些是“计数值”（只增不减），表示某类事件发生的次数，例如 `pgfault` 表示自 cgroup 创建以来的缺页次数。

`cache`
: 控制组进程用于页缓存的数据量，可与块设备上的数据块一一对应。当读写磁盘文件时该值会增加，包括常规 I/O（`open`/`read`/`write`）与映射文件（`mmap`）。该值也包含 `tmpfs` 挂载使用的内存。

`rss`
: 不对应磁盘内容的内存用量，如栈、堆与匿名映射。

`mapped_file`
: 控制组进程映射的文件内存量；反映“如何使用”，非“使用了多少”。

`pgfault`, `pgmajfault`
: 分别表示触发“缺页”和“重大缺页”的次数。缺页是进程访问不存在或受保护的虚拟内存区域时发生的事件：若访问无效地址会收到 `SIGSEGV` 并被终止；若读取被换出或映射文件对应的内存，内核会从磁盘载入该页并完成访问；写入写时复制区域时，内核会复制页面后再继续写入。“重大缺页”指需要从磁盘实际读取数据的情况；若只是复制现有页或分配空页，则属于普通（次要）缺页。

`swap`
: 当前 cgroup 进程使用的 swap 量。

`active_anon`, `inactive_anon`
: 被内核分别标记为“活跃”和“非活跃”的匿名内存（不关联磁盘页），与前述 `rss` 概念相当。严格来说，`rss = active_anon + inactive_anon - tmpfs`（其中 `tmpfs` 是该控制组挂载的 tmpfs 占用）。页面初始为“活跃”，内核会周期性扫描并将部分标记为“非活跃”；再次访问会立即恢复为“活跃”。当系统内存吃紧需要换出时，会优先换出“非活跃”页。

`active_file`, `inactive_file`
: 文件缓存内存，对应“活跃/非活跃”状态。`cache = active_file + inactive_file + tmpfs`。文件页在活跃/非活跃之间移动的规则与匿名内存不同，但原理相似。回收内存时，回收干净页（未修改）成本更低，可立即回收；匿名页或脏页需先写盘。

`unevictable`
: 不可回收的内存，通常由 `mlock` 锁定，用于确保密钥等敏感数据不会被换出到磁盘。

`memory_limit`, `memsw_limit`
: 并非指标，而是该 cgroup 应用的限制：前者为最大物理内存，后者为内存+swap 的上限。

页缓存的内存记账较为复杂。如果不同 cgroup 的两个进程读取同一文件（底层使用相同磁盘块），其对应的内存开销会在 cgroup 之间分摊。这虽可取，但也意味着当某个 cgroup 终止时，另一个 cgroup 的内存占用可能会上升，因为不再分摊这些页面的成本。

### CPU 指标：`cpuacct.stat` {#cpu-metrics-cpuacct-stat}

相较于内存指标，其余内容更为简单。CPU 指标位于 `cpuacct` 控制器。

每个容器都有伪文件 `cpuacct.stat`，记录容器进程累计的 CPU 使用，分为 `user` 与 `system` 时间：

- `user`：进程直接占用 CPU 执行自身代码的时间。
- `system`：内核代表进程执行系统调用所花费的时间。

这些时间以“ticks”（每秒 100 份，俗称 jiffies）表示；在 x86 上 `USER_HZ=100`。历史上与调度器“tick”一致，但更高频的调度与[tickless 内核](https://lwn.net/Articles/549580/)已使该数值不再关键。

#### 块 I/O 指标 {#block-i-o-metrics}

块 I/O 由 `blkio` 控制器负责记账。不同指标分散在不同文件中。详见内核文档中的 [blkio-controller](https://www.kernel.org/doc/Documentation/cgroup-v1/blkio-controller.txt)，这里列出最常用的条目：

`blkio.sectors`
: 以 512 字节扇区为单位，按设备统计读写的总扇区数（读写合并计数）。

`blkio.io_service_bytes`
: 记录该 cgroup 的读写字节数。每个设备有 4 个计数器（同步/异步 × 读/写）。

`blkio.io_serviced`
: 执行的 I/O 操作次数（与大小无关）。同样每设备 4 个计数器。

`blkio.io_queued`
: 当前排队的 I/O 操作数量。若该 cgroup 没有 I/O，该值为 0；反之不成立：无排队不代表空闲，可能在使用同步读且设备空闲无需排队。队列大小是相对量，也会受其他设备负载影响。

### 网络指标 {#network-metrics}

网络指标不会由 cgroup 直接提供，原因在于网络接口存在于“网络命名空间”的上下文中。内核可以统计某进程组的收发包与字节数，但这并不实用：通常需要按接口维度查看（本地 `lo` 的流量常不计），而同一 cgroup 的进程可能属于多个网络命名空间，导致出现多个 `lo`、多个 `eth0` 等，进而难以解释。因此无法通过 cgroup 简单获取网络指标。

因此需要从其他来源获取网络指标。

#### iptables {#iptables}

iptables（准确地说是其背后的 netfilter 框架）可以进行较为精细的统计。

例如，可设置规则统计 Web 服务器的出站 HTTP 流量：

```console
$ iptables -I OUTPUT -p tcp --sport 80
```

未使用 `-j` 或 `-g`，该规则仅对匹配的数据包计数并继续后续规则。

之后可通过以下命令查看计数器的值：

```console
$ iptables -nxvL OUTPUT
```

严格来说 `-n` 不是必须，但可避免 iptables 进行反向 DNS 查询，在此场景中通常没有意义。

计数器包括数据包数与字节数。若要按容器流量统计，可在 `FORWARD` 链中通过 `for` 循环为每个容器 IP 添加两条 `iptables` 规则（入站与出站各一条）。需要注意，这只能统计经过 NAT 层的流量；还需额外统计经由用户态代理转发的流量。

随后需要定期检查这些计数器。如果你使用 `collectd`，可以借助其[插件](https://collectd.org/wiki/index.php/Table_of_Plugins)自动采集 iptables 计数。

#### 接口级计数器 {#interface-level-counters}

每个容器都有一个虚拟以太网接口，你可以直接查看该接口的发送（TX）与接收（RX）计数。宿主机中容器对应的虚拟接口名称类似 `vethKk8Zqi`。遗憾的是，很难直接判断接口与容器的对应关系。

更现实的做法是从“容器内部”查看指标。可以通过所谓的 “ip-netns 魔法” 在容器的网络命名空间内，从宿主运行任意可执行程序。

`ip-netns exec` 能在当前可见的任意网络命名空间内执行宿主上的任意程序。宿主可以进入容器的网络命名空间，但容器无法访问宿主或其他并列容器（容器与其子容器间可互通）。

命令格式：

```console
$ ip netns exec <nsname> <command...>
```

示例：

```console
$ ip netns exec mycontainer netstat -i
```

`ip netns` 通过命名空间伪文件定位名为 `mycontainer` 的容器。每个进程属于一个网络命名空间、一个 PID 命名空间、一个 `mnt` 命名空间等，这些命名空间在 `/proc/<pid>/ns/` 下以伪文件形式呈现。例如 PID 42 的网络命名空间为 `/proc/42/ns/net`。

执行 `ip netns exec mycontainer ...` 时，要求 `/var/run/netns/mycontainer` 为上述伪文件之一（可用符号链接）。

换言之，要在容器的网络命名空间中执行命令，我们需要：

- Find out the PID of any process within the container that we want to investigate;
- Create a symlink from `/var/run/netns/<somename>` to `/proc/<thepid>/ns/net`
- Execute `ip netns exec <somename> ....`

结合[枚举 cgroups](#enumerate-cgroups) 的方法，找到容器内目标进程所属的 cgroup。随后查看该 cgroup 下名为 `tasks` 的伪文件（包含所有 PID），任选其一。

综合以上步骤，若容器的短 ID 存在环境变量 `$CID` 中，可执行：

```console
$ TASKS=/sys/fs/cgroup/devices/docker/$CID*/tasks
$ PID=$(head -n 1 $TASKS)
$ mkdir -p /var/run/netns
$ ln -sf /proc/$PID/ns/net /var/run/netns/$CID
$ ip netns exec $CID netstat -i
```

## 高性能指标采集提示 {#tips-for-high-performance-metric-collection}

每次采集指标都启动一个新进程的成本相对较高。若需要高频采集，或要在大量容器（单机上千）上收集指标，尽量避免每次都 fork 新进程。

可使用“单进程”采集方案。用 C（或可执行底层系统调用的语言）编写采集器，借助 `setns()` 让当前进程进入任意命名空间。该调用需要打开命名空间伪文件（位于 `/proc/<pid>/ns/net`）。

注意不要长时间保持该文件描述符处于打开状态。否则当控制组内最后一个进程退出时，该命名空间不会被销毁，其网络资源（如容器虚拟接口）会一直存在，直到关闭该描述符。

正确做法是记录每个容器的首个 PID，并在每次需要时重新打开对应的命名空间伪文件。

## 在容器退出时采集指标 {#collect-metrics-when-a-container-exits}

有时你不关注实时采集，而是希望在容器退出时了解其累计的 CPU、内存等使用情况。

这在 Docker 中较难，因为它依赖 `lxc-start` 并会在退出时清理现场。通常以固定间隔采集更简单，`collectd` 的 LXC 插件即采用这种方式。

如果仍希望在容器停止时抓取一次统计，可以按以下方式操作：

为每个容器启动一个采集进程，并将其 PID 写入需监控的 cgroup 的 `tasks` 文件，使其归属于该 cgroup。采集进程应定期重新读取 `tasks` 文件，检查自己是否是该控制组内最后一个进程。（若还需采集前文所述的网络指标，应将其移动到对应的网络命名空间。）

当容器退出时，`lxc-start` 会尝试删除控制组，但由于仍在使用会失败，这没关系。此时你的采集进程应检测到自己是组内唯一进程，此刻正是收集全部指标的时机。

最后，采集进程应将自己移回根控制组，并删除容器的控制组目录。删除控制组只需对其目录执行 `rmdir`。尽管目录仍包含文件，这在伪文件系统中是允许的。清理完成后，采集进程即可安全退出。
