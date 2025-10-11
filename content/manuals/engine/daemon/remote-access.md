---
description:
  通过配置远程访问，让 Docker 除了通过 Unix 套接字外，也能在指定 IP 和端口上侦听，
  从而接受来自远程主机的请求
keywords: configuration, daemon, remote access, engine
title: 为 Docker 守护进程配置远程访问
aliases:
  - /config/daemon/remote-access/
---

默认情况下，Docker 守护进程仅通过 Unix 套接字侦听来自本机客户端的连接。
你可以配置 Docker 让其在指定的 IP 和端口上侦听，从而同时接受远程客户端的请求。

<!-- prettier-ignore -->
> [!WARNING]
>
> 将 Docker 暴露给远程客户端，可能带来对宿主机的未授权访问与其他攻击风险。
>
> 在开启网络访问前，请务必理解其安全影响。如果不加以保护，远程的非 root 用户
> 可能获得宿主机的 root 级权限。
>
> 不使用 TLS 的远程访问**不被推荐**，且将在未来版本中需要显式同意方可启用。
> 如何使用 TLS 证书来保护该连接，请参阅
> [保护 Docker 守护进程套接字](/manuals/engine/security/protect-access.md)。

## 启用远程访问

在使用 systemd 的 Linux 发行版上，可以通过 `docker.service` 的 systemd unit 文件启用远程访问；
如果你的发行版不使用 systemd，也可以通过 `daemon.json` 来配置。

请勿同时在 systemd unit 文件和 `daemon.json` 中配置侦听地址；
同时配置会产生冲突并导致 Docker 无法启动。

### 通过 systemd unit 文件配置远程访问

1. 使用 `sudo systemctl edit docker.service` 打开 `docker.service` 的覆盖配置文件。

2. 添加或修改如下内容（根据需要替换为你的值）：

   ```systemd
   [Service]
   ExecStart=
   ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
   ```

3. 保存文件。

4. 重新加载 `systemctl` 配置：

   ```console
   $ sudo systemctl daemon-reload
   ```

5. 重启 Docker：

   ```console
   $ sudo systemctl restart docker.service
   ```

6. 验证配置已生效：

   ```console
   $ sudo netstat -lntp | grep dockerd
   tcp        0      0 127.0.0.1:2375          0.0.0.0:*               LISTEN      3758/dockerd
   ```

### 通过 `daemon.json` 配置远程访问

1. 在 `/etc/docker/daemon.json` 中设置 `hosts` 数组，同时包含 Unix 套接字与 IP 侦听地址，例如：

   ```json
   {
     "hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
   }
   ```

2. 重启 Docker。

3. 验证配置已生效：

   ```console
   $ sudo netstat -lntp | grep dockerd
   tcp        0      0 127.0.0.1:2375          0.0.0.0:*               LISTEN      3758/dockerd
   ```

### 通过防火墙放行远程 API 访问

如果 Docker 与防火墙运行在同一台主机上，而你希望从其他主机访问 Docker 远程 API，
则必须在防火墙中放行 Docker 端口的入站连接。使用 TLS 加密传输时默认端口为 `2376`，
否则为 `2375`。

常见的两种防火墙守护进程：

- [Uncomplicated Firewall (ufw)](https://help.ubuntu.com/community/UFW)：常见于 Ubuntu 系统。
- [firewalld](https://firewalld.org)：常见于基于 RPM 的系统。

请参考你的操作系统和防火墙文档。下面的示例仅用于入门演示，策略较为宽松；
在生产环境中，你可能需要更严格的安全策略。

- 对于 ufw，在配置中将 `DEFAULT_FORWARD_POLICY="ACCEPT"`。

- 对于 firewalld，在策略中添加类似如下的规则：一条用于入站请求，一条用于出站请求。

  ```xml
  <direct>
    [ <rule ipv="ipv6" table="filter" chain="FORWARD_direct" priority="0"> -i zt0 -j ACCEPT </rule> ]
    [ <rule ipv="ipv6" table="filter" chain="FORWARD_direct" priority="0"> -o zt0 -j ACCEPT </rule> ]
  </direct>
  ```

  请确保接口名称与链（chain）名称正确无误。

## 更多信息

有关守护进程远程访问配置项的更多细节，请参阅
[dockerd CLI 参考](/reference/cli/dockerd/#bind-docker-to-another-hostport-or-a-unix-socket)。
