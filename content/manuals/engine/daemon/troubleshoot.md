---
title: 排查 Docker 守护进程问题
description: 了解如何排查 Docker 守护进程中的错误与错误配置
keywords: |
  docker, daemon, configuration, troubleshooting, error, fail to start,
  networking, dns resolver, ip forwarding, dnsmasq, firewall,
  Cannot connect to the Docker daemon. Is 'docker daemon' running on this host?
aliases:
  - /engine/install/troubleshoot/
  - /storage/troubleshooting_volume_errors/
  - /config/daemon/troubleshooting/
tags: [Troubleshooting]
---

本页介绍在遇到问题时，如何对守护进程进行排查与调试。

你可以为守护进程启用调试，以了解其运行时活动并辅助排障。若守护进程无响应，
可以向 Docker 守护进程发送 `SIGUSR` 信号，
以[强制输出完整的堆栈跟踪](logs.md#force-a-stack-trace-to-be-logged)到守护进程日志。

## 守护进程

### 无法连接到 Docker 守护进程

```text
Cannot connect to the Docker daemon. Is 'docker daemon' running on this host?
```

该错误可能表示：

- Docker 守护进程未在你的系统上运行。请先启动后重试。
- Docker 客户端试图连接到其他主机上的 Docker 守护进程，但该主机不可达。

### 检查 Docker 是否在运行

检查 Docker 是否在运行，跨操作系统的通用方式是执行 `docker info`。

也可以使用系统工具：`sudo systemctl is-active docker`、`sudo status docker`、
`sudo service docker status`，或在 Windows 中查看服务状态。

最后，可以通过 `ps`、`top` 等命令在进程列表中查找 `dockerd` 进程。

#### 检查客户端连接的宿主

查看 `DOCKER_HOST` 环境变量以确定客户端连接的目标主机：

```console
$ env | grep DOCKER_HOST
```

如果该变量存在，表示 Docker 客户端将连接到远端主机上的守护进程；
若未设置，则连接本地主机上的守护进程。若误设，可执行以下命令取消：

```console
$ unset DOCKER_HOST
```

你可能需要编辑 `~/.bashrc`、`~/.profile` 等环境文件，避免 `DOCKER_HOST` 被误设。

如果 `DOCKER_HOST` 设置正确，请确认远端主机上的 Docker 守护进程正在运行，
且没有防火墙或网络中断阻止连接。

### 排查 `daemon.json` 与启动脚本的配置冲突

当你既使用 `daemon.json`，又在手动或启动脚本中为 `dockerd` 传参，若两者冲突，
Docker 会启动失败并报类似错误：

```text
unable to configure the Docker daemon with file /etc/docker/daemon.json:
the following directives are specified both as a flag and in the configuration
file: hosts: (from flag: [unix:///var/run/docker.sock], from file: [tcp://127.0.0.1:2376])
```

若出现类似错误且你是通过标志手动启动守护进程，需要调整命令行标志或 `daemon.json` 来消除冲突。

> [!NOTE]
>
> 如果你遇到关于 `hosts` 的特定错误，请继续阅读
> [下一节](#configure-the-daemon-host-with-systemd) 获取解决方案。

如果通过操作系统的初始化脚本启动 Docker，可能需要根据具体系统覆盖脚本中的默认设置。

#### 使用 systemd 配置守护进程的 host

一个较难排查的典型冲突是：当你想使用非默认的守护进程地址时。
Docker 默认监听 Unix 套接字。在基于 `systemd` 的 Debian/Ubuntu 上，启动 `dockerd` 时总会带上 `-H` 标志。
如果你又在 `daemon.json` 中设置了 `hosts`，就会产生冲突并导致守护进程启动失败。

可通过创建 `/etc/systemd/system/docker.service.d/docker.conf` 并写入以下内容，
来移除默认用于启动守护进程的 `-H` 参数：

```systemd
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

在其他场景下你也可能需要结合 `systemd` 与 Docker 的配置，例如
[配置 HTTP/HTTPS 代理](./proxy.md)。

> [!NOTE]
>
> 如果覆盖了该选项，但既没有在 `daemon.json` 中设置 `hosts`，也没有在手动启动时传入 `-H`，
> 则 Docker 将无法启动。

在启动 Docker 前运行 `sudo systemctl daemon-reload`。若启动成功，
Docker 将监听 `daemon.json` 中 `hosts` 指定的 IP 地址，而非套接字。

> [!IMPORTANT]
>
> 在 Docker Desktop for Windows/Mac 上，不支持在 `daemon.json` 中设置 `hosts`。

### 内存不足（OOM）问题

当容器尝试使用超过系统可用内存的资源时，可能触发 OOM 异常，容器或守护进程会被内核 OOM killer 终止。
为避免此情况，请确保应用运行在内存充足的宿主机上，并参考
[了解内存耗尽的风险](../containers/resource_constraints.md#understand-the-risks-of-running-out-of-memory)。

### 内核兼容性

如果内核版本低于 3.10，或缺失必要的内核模块，Docker 可能无法正常运行。
你可以下载并运行脚本
[`check-config.sh`](https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh)
来检查内核兼容性。

```console
$ curl https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh > check-config.sh

$ bash ./check-config.sh
```

该脚本仅适用于 Linux。

### 内核 cgroup 交换空间（swap）限制能力

在 Ubuntu 或 Debian 宿主机上操作镜像时，可能会看到如下提示：

```text
WARNING: Your kernel does not support swap limit capabilities. Limitation discarded.
```

如果不需要该能力，可以忽略此警告。

可在 Ubuntu 或 Debian 上按以下步骤启用该能力。
注意：即使 Docker 未运行，启用内存与 swap 计费会带来约 1% 的内存开销及约 10% 的整体性能下降。

1. 以具备 `sudo` 权限的用户登录 Ubuntu 或 Debian 宿主机。

2. 编辑 `/etc/default/grub`，在 `GRUB_CMDLINE_LINUX` 行中添加：

   ```text
   GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
   ```

   保存并关闭文件。

3. 更新 GRUB 引导：

   ```console
   $ sudo update-grub
   ```

   若 GRUB 配置语法有误会报错，请返回重试步骤 2 和 3。

   重启系统后生效。

## 网络

### IP 转发问题

如果你在 systemd 219+ 的系统上使用 `systemd-network` 手动配置网络，Docker 容器可能无法访问网络。
从 systemd 220 开始，某个网络接口的转发选项（`net.ipv4.conf.<interface>.forwarding`）默认关闭。
该设置会阻止 IP 转发，并与 Docker 在容器内启用 `net.ipv4.conf.all.forwarding` 的行为相冲突。

在 RHEL、CentOS 或 Fedora 上，可编辑 Docker 宿主机上的 `/usr/lib/systemd/network/` 下的 `<interface>.network`，
例如 `/usr/lib/systemd/network/80-container-host0.network`。

在 `[Network]` 段中添加如下配置：

```systemd
[Network]
...
IPForward=kernel
# OR
IPForward=true
```

该配置允许容器按预期进行 IP 转发。

### DNS 解析器问题

```console
DNS resolver found in resolv.conf and containers can't use it
```

许多 Linux 桌面环境会运行网络管理器程序，通过往 `/etc/resolv.conf` 中写入来配合 `dnsmasq` 缓存 DNS 请求。
`dnsmasq` 通常监听在 `127.0.0.1`、`127.0.1.1` 等回环地址上，既能加速 DNS 查询，也能提供 DHCP 服务。
但在 Docker 容器内，这种配置并不可行。容器使用自身的网络命名空间，会将 `127.0.0.1` 等回环地址解析为容器自身，
而容器自身通常并未运行 DNS 服务。

如果 Docker 发现 `/etc/resolv.conf` 中引用的地址并非可用的 DNS 服务器，会出现如下警告：

```text
WARNING: Local (127.0.0.1) DNS resolver found in resolv.conf and containers
can't use it. Using default external servers : [8.8.8.8 8.8.4.4]
```

看到该警告时，先检查是否使用了 `dnsmasq`：

```console
$ ps aux | grep dnsmasq
```

如果容器需要解析内网主机名，公网 DNS 并不适用。你有两种选择：

- 为 Docker 指定 DNS 服务器。
- 关闭 `dnsmasq`。

  关闭 `dnsmasq` 会将真实的 DNS 服务器地址写入 `/etc/resolv.conf`，但也会失去 `dnsmasq` 带来的缓存与管理能力。

以上两种方式任选其一。

### 为 Docker 指定 DNS 服务器

默认配置文件位置为 `/etc/docker/daemon.json`。你可以通过 `--config-file` 标志修改配置文件位置。
以下示例假设配置文件位于 `/etc/docker/daemon.json`。

1. 创建或编辑 Docker 守护进程配置文件（默认 `/etc/docker/daemon.json`）。

   ```console
   $ sudo nano /etc/docker/daemon.json
   ```

2. 添加 `dns` 键，并设置一个或多个 DNS 服务器 IP：

   ```json
   {
     "dns": ["8.8.8.8", "8.8.4.4"]
   }
   ```

   如果文件已有内容，仅需新增或编辑 `dns` 行。
   若你的内网 DNS 无法解析公网地址，请至少添加一个可解析公网的 DNS，以保证能连接到 Docker Hub，
   并让容器解析互联网域名。

   Save and close the file.

3. 重启 Docker 守护进程。

   ```console
   $ sudo service docker restart
   ```

4. 验证 Docker 能否解析外部地址，例如尝试拉取镜像：

   ```console
   $ docker pull hello-world
   ```

5. 如有需要，验证容器能否解析内网主机名：

   ```console
   $ docker run --rm -it alpine ping -c4 <my_internal_host>

   PING google.com (192.168.1.2): 56 data bytes
   64 bytes from 192.168.1.2: seq=0 ttl=41 time=7.597 ms
   64 bytes from 192.168.1.2: seq=1 ttl=41 time=7.635 ms
   64 bytes from 192.168.1.2: seq=2 ttl=41 time=7.660 ms
   64 bytes from 192.168.1.2: seq=3 ttl=41 time=7.677 ms
   ```

### 关闭 `dnsmasq`

{{< tabs >}}
{{< tab name="Ubuntu" >}}

如果你不想修改 Docker 守护进程的 DNS 配置，可按以下步骤在 NetworkManager 中关闭 `dnsmasq`：

1. Edit the `/etc/NetworkManager/NetworkManager.conf` file.

2. Comment out the `dns=dnsmasq` line by adding a `#` character to the beginning
   of the line.

   ```text
   # dns=dnsmasq
   ```

   Save and close the file.

3. Restart both NetworkManager and Docker. As an alternative, you can reboot
   your system.

   ```console
   $ sudo systemctl restart network-manager
   $ sudo systemctl restart docker
   ```

{{< /tab >}}
{{< tab name="RHEL, CentOS, or Fedora" >}}

在 RHEL、CentOS 或 Fedora 上关闭 `dnsmasq`：

1. Turn off the `dnsmasq` service:

   ```console
   $ sudo systemctl stop dnsmasq
   $ sudo systemctl disable dnsmasq
   ```

2. Configure the DNS servers manually using the
   [Red Hat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/configuring-the-order-of-dns-servers_configuring-and-managing-networking).

{{< /tab >}}
{{< /tabs >}}

### Docker 网络消失问题

如果某个 Docker 网络（如 `docker0` bridge 或自定义网络）随机消失或异常工作，
可能是其他服务干扰或修改了 Docker 的网络接口。宿主机上的网络管理工具有时会不恰当地修改 Docker 接口。

根据宿主机所用的网络管理工具，参考以下章节将 Docker 接口标记为不受管控（un-managed）：

- If `netscript` is installed, consider [uninstalling it](#uninstall-netscript)
- Configure the network manager to [treat Docker interfaces as un-managed](#un-manage-docker-interfaces)
- If you're using Netplan, you may need to [apply a custom Netplan configuration](#prevent-netplan-from-overriding-network-configuration)

#### 卸载 `netscript`

如果系统安装了 `netscript`，卸载它通常可以解决该问题。例如在基于 Debian 的系统上：

```console
$ sudo apt-get remove netscript-2.4
```

#### 将 Docker 接口标记为不受管控

In some cases, the network manager will attempt to manage Docker interfaces by
default. You can try to explicitly flag Docker networks as un-managed by
editing your system's network configuration settings.

{{< tabs >}}
{{< tab name="NetworkManager" >}}

如使用 `NetworkManager`，请在 `/etc/network/interfaces` 下编辑系统网络配置：

1. Create a file at `/etc/network/interfaces.d/20-docker0` with the following
   contents:

   ```text
   iface docker0 inet manual
   ```

   注意：此示例仅将默认的 `docker0` bridge 设为不受管控，不包含自定义网络。

2. Restart `NetworkManager` for the configuration change to take effect.

   ```console
   $ systemctl restart NetworkManager
   ```

3. Verify that the `docker0` interface has the `unmanaged` state.

   ```console
   $ nmcli device
   ```

{{< /tab >}}
{{< tab name="systemd-networkd" >}}

如在使用 `systemd-networkd` 的系统上运行 Docker，可在 `/etc/systemd/network` 下创建配置，
将 Docker 接口设为不受管控：

1. Create `/etc/systemd/network/docker.network` with the following contents:

   ```ini
   # 确保将 Docker 接口设为不受管控

   [Match]
   Name=docker0 br-* veth*

   [Link]
   Unmanaged=yes

   ```

2. Reload the configuration.

   ```console
   $ sudo systemctl restart systemd-networkd
   ```

3. Restart the Docker daemon.

   ```console
   $ sudo systemctl restart docker
   ```

4. Verify that the Docker interfaces have the `unmanaged` state.

   ```console
   $ networkctl
   ```

{{< /tab >}}
{{< /tabs >}}

### 防止 Netplan 覆盖网络配置

在通过 [`cloud-init`](https://cloudinit.readthedocs.io/en/latest/index.html)
使用 [Netplan](https://netplan.io/) 的系统上，可能需要应用自定义配置，
避免 `netplan` 覆盖网络管理器的配置：

1. Follow the steps in [Un-manage Docker interfaces](#un-manage-docker-interfaces)
   for creating the network manager configuration.
2. Create a `netplan` configuration file under `/etc/netplan/50-cloud-init.yml`.

   下列示例仅作为起点，请按需调整为你希望设为不受管控的接口。
   配置不当可能导致网络连通性问题。

   ```yaml {title="/etc/netplan/50-cloud-init.yml"}
   network:
     ethernets:
       all:
         dhcp4: true
         dhcp6: true
         match:
           # edit this filter to match whatever makes sense for your system
           name: en*
     renderer: networkd
     version: 2
   ```

3. 应用新的 Netplan 配置。

   ```console
   $ sudo netplan apply
   ```

4. 重启 Docker 守护进程：

   ```console
   $ sudo systemctl restart docker
   ```

5. 验证 Docker 接口是否处于 `unmanaged` 状态。

   ```console
   $ networkctl
   ```

## 卷（Volumes）

### 无法移除文件系统

```text
Error: Unable to remove filesystem
```

一些基于容器的工具（如 [Google cAdvisor](https://github.com/google/cadvisor)）
会将 Docker 的系统目录（如 `/var/lib/docker/`）挂载进容器。其文档中示例类似：

```console
$ sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```

当你绑定挂载 `/var/lib/docker/` 时，等于把所有正在运行的容器资源作为文件系统挂载到该容器内。
此时尝试删除其他容器可能失败，并出现如下错误：

```text
Error: Unable to remove filesystem for
74bef250361c7817bee19349c93139621b272bc8f654ae112dd4eb9652af9515:
remove /var/lib/docker/containers/74bef250361c7817bee19349c93139621b272bc8f654ae112dd4eb9652af9515/shm:
Device or resource busy
```

当绑定挂载 `/var/lib/docker/` 的容器对其中的文件系统句柄使用了 `statfs` 或 `fstatfs` 且未关闭时，就会出现上述问题。

通常不建议以这种方式绑定挂载 `/var/lib/docker`，
但 `cAdvisor` 的核心功能需要这样做。

如果不确定是哪个进程占用了错误信息中的路径并阻止删除，可以使用 `lsof` 查找对应进程，例如：

```console
$ sudo lsof /var/lib/docker/containers/74bef250361c7817bee19349c93139621b272bc8f654ae112dd4eb9652af9515/shm
```

解决方法：先停止那个绑定挂载了 `/var/lib/docker` 的容器，再尝试删除其他容器。
