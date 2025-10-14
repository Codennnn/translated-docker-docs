---
description: 了解 Docker Desktop 的网络工作方式，并查看已知限制
keywords: networking, docker desktop, proxy, vpn, Linux, Mac, Windows
title: 探索 Docker Desktop 的网络功能
linkTitle: 网络
aliases:
- /desktop/linux/networking/
- /docker-for-mac/networking/
- /mackit/networking/
- /desktop/mac/networking/
- /docker-for-win/networking/
- /docker-for-windows/networking/
- /desktop/windows/networking/
- /desktop/networking/
weight: 30
---

Docker Desktop 内置了网络功能，可帮助你将容器与主机上的服务互联、实现容器间通信，或通过代理与 VPN 访问网络。

## 适用于所有平台的网络功能

### VPN 透传

Docker Desktop 的网络在连接 VPN 时同样可以工作。其原理是拦截来自容器的流量，并像源自 Docker 应用一样将其注入到主机网络中。

### 端口映射

当你使用 `-p` 参数运行容器时，例如：

```console
$ docker run -p 80:80 -d nginx
```

Docker Desktop 会将容器内运行在 `80` 端口的服务（此处为 `nginx`）映射到主机的 `localhost` 上的 `80` 端口。本例中，主机端口与容器端口相同。

为避免与主机上已占用 `80` 端口的服务冲突：

```console
$ docker run -p 8000:80 -d nginx
```

现在，对 `localhost:8000` 的连接会被转发到容器内的 `80` 端口。

> [!TIP]
>
> `-p` 的语法为 `HOST_PORT:CLIENT_PORT`（宿主机端口:容器端口）。

### HTTP/HTTPS 代理支持

参见 [Proxies](/manuals/desktop/settings-and-maintenance/settings.md#proxies)

### SOCKS5 代理支持

{{< summary-bar feature_name="SOCKS5 proxy support" >}}

SOCKS（Socket Secure）是一种协议，借助代理服务器在客户端与服务端之间转发网络数据包，为用户与应用带来更好的隐私性、安全性与网络性能。

你可以启用 SOCKS 代理支持，以允许拉取镜像等出站请求，并从主机访问 Linux 容器后端的 IP。

启用并配置 SOCKS 代理：

1. 打开 **Settings**，进入 **Resources** 选项卡。
2. 在下拉菜单中选择 **Proxies**。
3. 开启 **Manual proxy configuration** 开关。
4. 在 **Secure Web Server HTTPS** 输入框中粘贴你的 `socks5://host:port` URL。

## Mac 与 Windows 的网络模式与 DNS 行为

自 Docker Desktop 4.42 起，你可以自定义 Docker 处理容器网络与 DNS 解析的方式，以更好地支持从仅 IPv4、双栈到仅 IPv6 等不同环境。这些设置有助于避免由于主机网络不兼容或配置不当导致的超时与连接问题。

> [!NOTE]
>
> 这些设置可通过 CLI 标志或 Compose 文件选项在单个网络层面进行覆盖。

### 默认网络模式

为 Docker 创建新网络时选择默认使用的 IP 协议。这样可以使 Docker 与主机的网络能力或组织要求保持一致，例如强制仅允许 IPv6 访问。

可选项包括：

- **Dual IPv4/IPv6（默认）**：同时支持 IPv4 与 IPv6。灵活性最高，适用于双栈网络环境。
- **仅 IPv4**：仅使用 IPv4 地址。如果主机或网络不支持 IPv6，建议选择此项。
- **仅 IPv6**：仅使用 IPv6 地址。适用于正在向仅 IPv6 迁移或强制使用 IPv6 的环境。

> [!NOTE]
>
> 此设置可通过 CLI 标志或 Compose 文件选项在单个网络层面进行覆盖。

### DNS 解析行为 

控制 Docker 如何过滤返回给容器的 DNS 记录，从而在仅支持 IPv4 或 IPv6 的环境中提升可靠性。该设置可避免应用尝试使用实际不可用的 IP 家族进行连接，从而减少不必要的延迟或失败。

根据所选网络模式，可用选项包括：

- **Auto（推荐）**：Docker 自动检测主机的网络栈，并过滤掉不受支持的 DNS 记录类型（IPv4 的 A 记录、IPv6 的 AAAA 记录）。
- **Filter IPv4（过滤 IPv4，A 记录）**：阻止容器解析 IPv4 地址。仅在双栈模式下可用。
- **Filter IPv6（过滤 IPv6，AAAA 记录）**：阻止容器解析 IPv6 地址。仅在双栈模式下可用。
- **No filtering（不过滤）**：无论主机是否支持，Docker 都返回所有 DNS 记录（A 与 AAAA）。

> [!IMPORTANT]
>
> 切换默认网络模式时，DNS 过滤会被重置为 Auto。

### 使用设置管理（Settings Management）

如果你是管理员，可以使用[设置管理](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md#networking)在开发者机器上统一下发该 Docker Desktop 设置。可从以下代码片段中选择并将其添加到 `admin-settings.json` 文件，或通过 [Admin Console](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md) 进行配置。

{{< tabs >}}
{{< tab name="Networking mode" >}}

双栈 IPv4/IPv6：

```json
{
  "defaultNetworkingMode": {
    "locked": true
    "value": "dual-stack"
  }
}
```

仅 IPv4：

```json
{
  "defaultNetworkingMode": {
    "locked": true
    "value": "ipv4only"
  }
}
```

仅 IPv6：

```json
{
  "defaultNetworkingMode": {
    "locked": true
    "value": "ipv6only"
  }
}
```

{{< /tab >}}
{{< tab name="DNS resolution" >}}

自动过滤：

```json
{
  "dnsInhibition": {
    "locked": true
    "value": "auto"
  }
}
```

过滤 IPv4：

```json
{
  "dnsInhibition": {
    "locked": true
    "value": "ipv4"
  }
}
```

过滤 IPv6：

```json
{
  "dnsInhibition": {
    "locked": true
    "value": "ipv6"
  }
}
```

不进行过滤：

```json
{
  "dnsInhibition": {
    "locked": true
    "value": "none"
  }
}
```

{{< /tab >}}
{{< /tabs >}}

## 适用于 Mac 与 Linux 的网络功能

### SSH agent 转发

Mac 与 Linux 版 Docker Desktop 支持在容器内使用主机的 SSH agent。操作如下：

1. 在 `docker run` 命令中添加以下参数，将 SSH agent 的套接字以绑定挂载的方式注入：

   ```console
   $--mount type=bind,src=/run/host-services/ssh-auth.sock,target=/run/host-services/ssh-auth.sock
   ```

2. 在容器中新增环境变量 `SSH_AUTH_SOCK`：

    ```console
    $ -e SSH_AUTH_SOCK="/run/host-services/ssh-auth.sock"
    ```

如需在 Docker Compose 中启用 SSH agent，请在服务中添加以下配置：

 ```yaml
services:
  web:
    image: nginx:alpine
    volumes:
      - type: bind
        source: /run/host-services/ssh-auth.sock
        target: /run/host-services/ssh-auth.sock
    environment:
      - SSH_AUTH_SOCK=/run/host-services/ssh-auth.sock
 ```

## 已知限制

### 修改内部 IP 地址

可在 **Settings** 中修改 Docker 使用的内部 IP 地址。更改 IP 后，需要重置 Kubernetes 集群，并退出任何活动中的 Swarm。

### 主机上不存在 `docker0` 网桥

由于 Docker Desktop 的网络实现方式，你在主机上看不到 `docker0` 接口。该接口实际上位于虚拟机中。

### 无法 ping 容器

Docker Desktop 无法将流量路由到 Linux 容器。不过，如果你使用的是 Windows，能够 ping Windows 容器。

### 无法为每个容器单独分配 IP

这是因为主机无法直接访问 Docker 的 `bridge` 网络。
但如果你使用的是 Windows，对 Windows 容器可以为每个容器分配独立 IP。

## 用例与替代方案 

### 从容器访问主机上的服务

主机的 IP 可能会变化，或者在无网络访问时没有可用 IP。
Docker 建议使用特殊的 DNS 名称 `host.docker.internal`，它会解析为主机使用的内部 IP 地址。

你也可以通过 `gateway.docker.internal` 访问网关。

如果你的机器安装了 Python，可按下述示例从容器连接到主机上的服务：

1. 运行以下命令，在 8000 端口启动一个简单的 HTTP 服务器。

    `python -m http.server 8000`

    如果安装的是 Python 2.x，请运行 `python -m SimpleHTTPServer 8000`。

2. 随后运行一个容器，安装 `curl`，并使用以下命令尝试连接主机：

    ```console
    $ docker run --rm -it alpine sh
    # apk add curl
    # curl http://host.docker.internal:8000
    # exit
    ```

### 从主机访问容器

面向 `localhost` 的端口转发可正常工作。`--publish`、`-p` 或 `-P` 均可用。
从 Linux 容器暴露的端口会被转发到主机。

Docker 建议你发布一个端口，或从另一个容器进行连接。
即使在 Linux 上，如果容器位于 overlay 网络而非 bridge 网络（不可路由），也需要这样做。

例如，运行一个 `nginx` Web 服务器：

```console
$ docker run -d -p 80:80 --name webserver nginx
```

语法说明：以下两个命令都会将容器的 `80` 端口发布到主机的 `8000` 端口：

```console
$ docker run --publish 8000:80 --name webserver nginx

$ docker run -p 8000:80 --name webserver nginx
```

如需发布所有端口，请使用 `-P` 标志。例如，以下命令会以分离模式启动一个容器，并将其暴露的所有端口发布到主机的随机端口上。

```console
$ docker run -d -P --name webserver nginx
```

或者，你也可以使用[主机网络](/manuals/engine/network/drivers/host.md#docker-desktop)，让容器直接使用主机的网络栈。

关于在 `docker run` 中使用的发布选项，参见 [run 命令](/reference/cli/docker/container/run.md)。
