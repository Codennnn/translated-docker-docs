---
description: 了解如何为 Docker 守护进程配置 HTTP 代理
keywords: dockerd, daemon, configuration, proxy, networking, http_proxy, https_proxy, no_proxy, systemd, environment variables
title: 守护进程代理配置
weight: 30
aliases:
  - /articles/host_integration/
  - /articles/systemd/
  - /engine/admin/systemd/
  - /engine/articles/systemd/
  - /config/daemon/systemd/
  - /config/daemon/proxy/
---

<a name="httphttps-proxy"><!-- included for deep-links to old section --></a>

如果你的组织通过代理服务器访问互联网，你可能需要为 Docker 守护进程配置代理。
守护进程会使用代理去访问 Docker Hub 和其他镜像仓库中的镜像，
以及与 Docker Swarm 中的其他节点通信。

本页介绍如何为 Docker 守护进程配置代理。关于为 Docker CLI 配置代理，参见
[为 Docker CLI 配置代理](/manuals/engine/cli/proxy.md)。

> [!IMPORTANT]
> Docker Desktop 会忽略在 `daemon.json` 中设置的代理配置。
> 如果你使用 Docker Desktop，请在
> [Docker Desktop 设置](/manuals/desktop/settings-and-maintenance/settings.md#proxies)
> 中配置代理。

你可以通过两种方式进行配置：

- 通过配置文件或 CLI 标志进行[守护进程配置](#daemon-configuration)
- 在系统上设置[环境变量](#environment-variables)

直接在守护进程中配置的设置优先生效，高于环境变量。

## 守护进程配置

可以在 `daemon.json` 中为守护进程配置代理，
或在 `dockerd` 命令上使用 `--http-proxy`、`--https-proxy` 等 CLI 标志。
推荐使用 `daemon.json` 进行配置。

```json
{
  "proxies": {
    "http-proxy": "http://proxy.example.com:3128",
    "https-proxy": "https://proxy.example.com:3129",
    "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
  }
}
```

修改配置文件后，重启守护进程以使代理配置生效：

```console
$ sudo systemctl restart docker
```

## 环境变量

Docker 守护进程在启动环境中检查以下环境变量，以配置 HTTP/HTTPS 代理行为：

- `HTTP_PROXY`
- `http_proxy`
- `HTTPS_PROXY`
- `https_proxy`
- `NO_PROXY`
- `no_proxy`

### systemd unit 文件

如果你将 Docker 守护进程作为 systemd 服务运行，可以通过创建 systemd drop-in 文件，
为 `docker` 服务设置这些环境变量。

> **Rootless 模式提示**
>
> 当以[无根（rootless）模式](/manuals/engine/security/rootless.md)运行 Docker 时，
> systemd 配置文件的位置不同。此时 Docker 作为用户态 systemd 服务启动，
> 并使用保存在用户主目录中的文件：`~/.config/systemd/<user>/docker.service.d/`。
> 此外，`systemctl` 需要在无 `sudo` 的情况下并带上 `--user` 标志执行。
> 如果你处于 rootless 模式，请切换到“Rootless mode”标签查看对应说明。

{{< tabs >}}
{{< tab name="Regular install" >}}

1. 为 `docker` 服务创建一个 systemd drop-in 目录：

   ```console
   $ sudo mkdir -p /etc/systemd/system/docker.service.d
   ```

2. 创建 `/etc/systemd/system/docker.service.d/http-proxy.conf` 文件，
   写入 `HTTP_PROXY` 环境变量：

   ```systemd
   [Service]
   Environment="HTTP_PROXY=http://proxy.example.com:3128"
   ```

   如果位于 HTTPS 代理之后，设置 `HTTPS_PROXY` 环境变量：

   ```systemd
   [Service]
   Environment="HTTPS_PROXY=https://proxy.example.com:3129"
   ```

   可以同时设置多个环境变量；例如同时设置 HTTP 与 HTTPS 代理：

   ```systemd
   [Service]
   Environment="HTTP_PROXY=http://proxy.example.com:3128"
   Environment="HTTPS_PROXY=https://proxy.example.com:3129"
   ```

   > [!NOTE]
   >
   > 代理字符串中的特殊字符（如 `#?!()[]{}`）必须使用 `%%` 进行双重转义。例如：
   >
   > ```systemd
   > [Service]
   > Environment="HTTP_PROXY=http://domain%%5Cuser:complex%%23pass@proxy.example.com:3128/"
   > ```

3. 如果有内部 Docker 镜像仓库需要直连（不走代理），可以通过 `NO_PROXY` 环境变量指定。

   `NO_PROXY` 为以逗号分隔的主机列表，用于指定不使用代理的目标。可用形式包括：

   - IP 地址或前缀（如 `1.2.3.4`）
   - 域名，或特殊 DNS 通配符（`*`）
   - 域名匹配该域及其所有子域；以点开头（如 `.example.com`）仅匹配子域。
     例如，在存在 `foo.example.com` 与 `example.com` 时：
     - `example.com` 同时匹配 `example.com` 和 `foo.example.com`
     - `.example.com` 仅匹配 `foo.example.com`
   - 单个星号（`*`）表示不使用任何代理
   - 可带端口号，如 IP `1.2.3.4:80` 或域名 `foo.example.com:80`

   示例：

   ```systemd
   [Service]
   Environment="HTTP_PROXY=http://proxy.example.com:3128"
   Environment="HTTPS_PROXY=https://proxy.example.com:3129"
   Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
   ```

4. 刷新配置并重启 Docker：

   ```console
   $ sudo systemctl daemon-reload
   $ sudo systemctl restart docker
   ```

5. 验证配置已加载且与修改一致，例如：

   ```console
   $ sudo systemctl show --property=Environment docker

   Environment=HTTP_PROXY=http://proxy.example.com:3128 HTTPS_PROXY=https://proxy.example.com:3129 NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp
   ```

{{< /tab >}}
{{< tab name="Rootless mode" >}}

1. 为 `docker` 服务创建 systemd drop-in 目录：

   ```console
   $ mkdir -p ~/.config/systemd/user/docker.service.d
   ```

2. 创建 `~/.config/systemd/user/docker.service.d/http-proxy.conf` 文件，
   写入 `HTTP_PROXY` 环境变量：

   ```systemd
   [Service]
   Environment="HTTP_PROXY=http://proxy.example.com:3128"
   ```

   如果位于 HTTPS 代理之后，设置 `HTTPS_PROXY` 环境变量：

   ```systemd
   [Service]
   Environment="HTTPS_PROXY=https://proxy.example.com:3129"
   ```

   可以同时设置多个环境变量；例如同时设置 HTTP 与 HTTPS 代理：

   ```systemd
   [Service]
   Environment="HTTP_PROXY=http://proxy.example.com:3128"
   Environment="HTTPS_PROXY=https://proxy.example.com:3129"
   ```

   > [!NOTE]
   >
   > 代理字符串中的特殊字符（如 `#?!()[]{}`）必须使用 `%%` 进行双重转义。例如：
   >
   > ```systemd
   > [Service]
   > Environment="HTTP_PROXY=http://domain%%5Cuser:complex%%23pass@proxy.example.com:3128/"
   > ```

3. 如果有内部 Docker 镜像仓库需要直连（不走代理），可以通过 `NO_PROXY` 环境变量指定。

   `NO_PROXY` 为以逗号分隔的主机列表，用于指定不使用代理的目标。可用形式包括：

   - IP 地址或前缀（如 `1.2.3.4`）
   - 域名，或特殊 DNS 通配符（`*`）
   - 域名匹配该域及其所有子域；以点开头（如 `.example.com`）仅匹配子域。
     例如，在存在 `foo.example.com` 与 `example.com` 时：
     - `example.com` 同时匹配 `example.com` 和 `foo.example.com`
     - `.example.com` 仅匹配 `foo.example.com`
   - 单个星号（`*`）表示不使用任何代理
   - 可带端口号，如 IP `1.2.3.4:80` 或域名 `foo.example.com:80`

   示例：

   ```systemd
   [Service]
   Environment="HTTP_PROXY=http://proxy.example.com:3128"
   Environment="HTTPS_PROXY=https://proxy.example.com:3129"
   Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
   ```

4. 刷新配置并重启 Docker：

   ```console
   $ systemctl --user daemon-reload
   $ systemctl --user restart docker
   ```

5. 验证配置已加载且与修改一致，例如：

   ```console
   $ systemctl --user show --property=Environment docker

   Environment=HTTP_PROXY=http://proxy.example.com:3128 HTTPS_PROXY=https://proxy.example.com:3129 NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp
   ```

{{< /tab >}}
{{< /tabs >}}
