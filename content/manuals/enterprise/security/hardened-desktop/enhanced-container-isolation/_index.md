---
title: 增强容器隔离
linkTitle: 增强容器隔离
description: 增强容器隔离通过防止恶意容器破坏主机系统，为 Docker Desktop 提供额外的安全性
keywords: 增强容器隔离, 容器安全, sysbox 运行时, linux 用户命名空间, 强化桌面
aliases:
 - /desktop/hardened-desktop/enhanced-container-isolation/
 - /security/for-admins/hardened-desktop/enhanced-container-isolation/
 - /security/hardened-desktop/enhanced-container-isolation/how-eci-works
 - /security/hardened-desktop/enhanced-container-isolation/features-benefits
weight: 10
---

{{< summary-bar feature_name="Hardened Docker Desktop" >}}

增强容器隔离（ECI）防止恶意容器破坏 Docker Desktop 或主机系统。它自动应用高级安全技术，同时保持完全的开发人员生产力与工作流程兼容性。

ECI 加强了容器隔离，并锁定了管理员创建的安全配置，如[仓库访问管理策略](/manuals/enterprise/security/hardened-desktop/registry-access-management.md)和[设置管理](../settings-management/_index.md)控制。

> [!NOTE]
>
> ECI 与其他 Docker 安全功能协同工作，如减少的 Linux 能力、seccomp 和 AppArmor。

## 谁应该使用增强容器隔离？

增强容器隔离适用于：

- 希望防止基于容器的攻击并减少开发环境中安全漏洞的组织
- 需要更强容器隔离而不影响开发工作流程的安全团队
- 在运行不受信任或第三方容器镜像时需要额外保护的企业

## 增强容器隔离如何工作

Docker 使用 [Sysbox 容器运行时](https://github.com/nestybox/sysbox) 实现 ECI，这是标准 OCI runc 运行时的安全增强分支。当开启 ECI 时，通过 `docker run` 或 `docker create` 创建的容器会自动使用 Sysbox 而不是 runc，无需对开发工作流程进行任何更改。

即使使用 `--privileged` 标志的容器也能在增强容器隔离下安全运行，防止它们破坏 Docker Desktop 虚拟机或其他容器。

> [!NOTE]
>
> 当开启 ECI 时，Docker CLI 的 `--runtime` 标志会被忽略。
Docker 的默认运行时仍然是 runc，但所有用户容器都隐式地使用 Sysbox 启动。

## 主要安全特性

### Linux 用户命名空间隔离

使用增强容器隔离时，所有容器都利用 Linux 用户命名空间实现更强的隔离。容器根用户映射到 Docker Desktop VM 中的非特权用户：

```console
$ docker run -it --rm --name=first alpine
/ # cat /proc/self/uid_map
         0     100000      65536
```

此输出显示容器根用户（0）映射到 VM 中的非特权用户 100000，范围是 64K 个用户 ID。每个容器获得独占映射：

```console
$ docker run -it --rm --name=second alpine
/ # cat /proc/self/uid_map
         0     165536      65536
```

没有增强容器隔离时，容器以真正的 root 用户运行：

```console
$ docker run -it --rm alpine
/ # cat /proc/self/uid_map
         0       0     4294967295
```

通过使用 Linux 用户命名空间，ECI 确保容器进程永远不会在 Linux VM 中以有效的用户 ID 运行，将其能力限制在容器内的资源。

### 安全特权容器

特权容器（`docker run --privileged`）通常构成重大安全风险，因为它们提供对 Linux 内核的无限制访问。没有 ECI 时，特权容器可以：

- 以真正的 root 用户运行，拥有所有能力
- 绕过 seccomp 和 AppArmor 限制
- 访问所有硬件设备
- 修改全局内核设置

保护开发环境的组织面临特权容器的挑战，因为它们可以获得 Docker Desktop VM 的控制权并更改安全设置，如仓库访问管理和网络代理。

增强容器隔离通过确保特权容器只能访问其容器边界内的资源来改变特权容器。例如，特权容器无法访问 Docker Desktop 的网络配置：

```console
$ docker run --privileged djs55/bpftool map show
Error: can't get next map: Operation not permitted
```

没有 ECI 时，特权容器可以轻松访问和修改这些设置：

```console
$ docker run --privileged djs55/bpftool map show
17: ringbuf  name blocked_packets  flags 0x0
        key 0B  value 0B  max_entries 16777216  memlock 0B
18: hash  name allowed_map  flags 0x0
        key 4B  value 4B  max_entries 10000  memlock 81920B
```

Docker-in-Docker 和 Kubernetes-in-Docker 等高级容器工作负载仍然可以在 ECI 下工作，但运行更加安全。

> [!NOTE]
>
> ECI 并不阻止用户运行特权容器，而是通过限制其访问来使它们安全。修改全局内核设置（加载内核模块、更改 Berkeley Packet Filter 设置）的特权工作负载会收到"权限被拒绝"错误。

### 命名空间隔离强制执行

增强容器隔离阻止容器与 Docker Desktop VM 共享 Linux 命名空间，保持隔离边界：

**PID 命名空间共享被阻止：**

```console
$ docker run -it --rm --pid=host alpine
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: error in the container spec: invalid or unsupported container spec: sysbox containers can't share namespaces [pid] with the host (because they use the linux user-namespace for isolation): unknown.
```

**网络命名空间共享被阻止：**

```console
$ docker run -it --rm --network=host alpine
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: error in the container spec: invalid or unsupported container spec: sysbox containers can't share a network namespace with the host (because they use the linux user-namespace for isolation): unknown.
```

**用户命名空间覆盖被忽略：**

```console
$ docker run -it --rm --userns=host alpine
/ # cat /proc/self/uid_map
         0     100000      65536
```

使用 `--network-host` 的 Docker 构建操作和 Docker buildx 授权（`network.host`、`security.insecure`）也被阻止。

### 受保护的绑定挂载

增强容器隔离在阻止对敏感 VM 目录访问的同时，保持对标准文件共享的支持：

主机目录挂载继续工作：

```console
$ docker run -it --rm -v $HOME:/mnt alpine
/ # ls /mnt
# 成功列出主目录内容
```

VM 配置挂载被阻止：

```console
$ docker run -it --rm -v /etc/docker/daemon.json:/mnt/daemon.json alpine
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: error in the container spec: can't mount /etc/docker/daemon.json because it's configured as a restricted host mount: unknown
```

这可以防止容器读取或修改 Docker Engine 配置、仓库访问管理设置、代理配置和其他与安全相关的 VM 文件。

> [!NOTE]
>
> 默认情况下，ECI 阻止绑定挂载 Docker Engine 套接字（/var/run/docker.sock），因为这会授予容器对 Docker Engine 的控制权。管理员可以为受信任的容器镜像创建例外。

### 高级系统调用保护

增强容器隔离拦截敏感系统调用，防止容器恶意使用合法能力：

```console
$ docker run -it --rm --cap-add SYS_ADMIN -v $HOME:/mnt:ro alpine
/ # mount -o remount,rw /mnt /mnt
mount: permission denied (are you root?)
```

即使拥有 `CAP_SYS_ADMIN` 能力，容器也无法将只读绑定挂载更改为读写，确保它们无法破坏容器边界。

容器仍然可以在其文件系统内创建内部挂载：

```console
/ # mkdir /root/tmpfs
/ # mount -t tmpfs tmpfs /root/tmpfs
/ # mount -o remount,ro /root/tmpfs /root/tmpfs
/ # findmnt | grep tmpfs
├─/root/tmpfs    tmpfs      tmpfs    ro,relatime,uid=100000,gid=100000
```

ECI 通过仅拦截控制路径系统调用（很少使用）而保持数据路径系统调用不受影响，从而高效地执行系统调用过滤，保持容器性能。

### 自动文件系统用户 ID 映射

增强容器隔离通过自动文件系统映射解决具有不同用户 ID 范围的容器之间的文件共享挑战。

每个容器获得独占的用户 ID 映射，但 Sysbox 使用 Linux 内核 ID 映射挂载（2021 年添加）或替代的 shiftsfs 模块进行文件系统用户 ID 重映射。这将文件系统访问从容器的真实用户 ID 映射到标准范围，从而实现：

- 具有不同用户 ID 范围的容器之间的卷共享
- 无论容器用户 ID 映射如何，文件所有权保持一致
- 无需用户干预的透明文件访问

### 通过文件系统模拟隐藏信息

ECI 在容器内模拟 `/proc` 和 `/sys` 文件系统的部分内容，以隐藏敏感主机信息并提供内核资源的每个容器视图：

```console
$ docker run -it --rm alpine
/ # cat /proc/uptime
5.86 5.86
```

这显示的是容器运行时间而不是 Docker Desktop VM 运行时间，防止系统信息泄漏到容器中。

几个未被 Linux 内核命名空间的 `/proc/sys` 资源在每个容器中被模拟，Sysbox 在编程内核设置时协调值。这使得通常需要特权访问的容器工作负载能够安全运行。

## 性能和兼容性

增强容器隔离保持优化的性能和完全的兼容性：

- 无性能影响：系统调用过滤仅针对控制路径调用，保持数据路径操作不受影响
- 完全工作流程兼容性：现有开发流程、工具和容器镜像无需更改即可工作
- 高级工作负载支持：Docker-in-Docker、Kubernetes-in-Docker 和其他复杂场景安全运行
- 自动管理：用户 ID 映射、文件系统访问和安全策略自动处理
- 标准镜像支持：无需特殊的容器镜像或修改

> [!IMPORTANT]
>
> ECI 保护因 Docker Desktop 版本而异，尚未保护扩展容器。Docker 构建和 Docker Desktop 中的 Kubernetes 根据版本具有不同的保护级别。有关详细信息，请参阅[增强容器隔离限制](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/limitations.md)。
