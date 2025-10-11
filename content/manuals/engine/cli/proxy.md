---
title: 在 Docker CLI 中使用代理服务器
linkTitle: 代理配置
weight: 20
description: 配置 Docker 客户端 CLI 使用代理服务器
keywords: network, networking, proxy, client
aliases:
  - /network/proxy/
---

本文介绍如何通过容器中的环境变量配置 Docker CLI 使用代理。

本文不涉及 Docker Desktop 的代理配置。请参考[在 Docker Desktop 中配置 HTTP/HTTPS 代理](/manuals/desktop/settings-and-maintenance/settings.md#proxies)。

如果你在没有 Docker Desktop 的环境中运行 Docker Engine，且需要为 Docker 守护进程（`dockerd`）配置代理，请参见[为 Docker 守护进程配置代理](/manuals/engine/daemon/proxy.md)。

如果容器需要使用 HTTP、HTTPS 或 FTP 代理，你可以按以下方式配置：

- [通过 Docker 客户端配置](#configure-the-docker-client)
- [通过 CLI 直接设置代理](#set-proxy-using-the-cli)

> [!NOTE]
>
> 目前并没有统一标准规定 Web 客户端应如何处理代理环境变量，或其具体格式。
>
> 如需了解这些变量的历史背景，可参考 GitLab 团队的博文：[We need to talk: Can we standardize NO_PROXY?](https://about.gitlab.com/blog/2021/01/27/we-need-to-talk-no-proxy/)。

## 配置 Docker 客户端

可以在 `~/.docker/config.json` 中以 JSON 形式为 Docker 客户端添加代理配置。构建与容器都会使用该文件中的设置。

```json
{
 "proxies": {
   "default": {
    "httpProxy": "http://proxy.example.com:3128",
    "httpsProxy": "https://proxy.example.com:3129",
     "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
   }
 }
}
```

> [!WARNING]
>
> 代理配置可能包含敏感信息。例如，某些代理需要在 URL 中附带认证信息；或代理地址本身会暴露公司内部的 IP/主机名。
>
> 环境变量会以明文形式存储在容器配置中，因此可能通过远程 API 被查看，或在使用 `docker commit` 时被写入镜像。

保存文件后配置会立即生效，无需重启 Docker。但它只作用于新创建的容器与构建，不会影响已存在的容器。

配置项说明如下：

| Property     | Description                                                                         |
| :----------- | :---------------------------------------------------------------------------------- |
| `httpProxy`  | 设置 `HTTP_PROXY`/`http_proxy` 环境变量与构建参数。   |
| `httpsProxy` | 设置 `HTTPS_PROXY`/`https_proxy` 环境变量与构建参数。 |
| `ftpProxy`   | 设置 `FTP_PROXY`/`ftp_proxy` 环境变量与构建参数。     |
| `noProxy`    | 设置 `NO_PROXY`/`no_proxy` 环境变量与构建参数。       |
| `allProxy`   | 设置 `ALL_PROXY`/`all_proxy` 环境变量与构建参数。     |

上述设置仅用于为容器配置代理相关的环境变量，并不会作为 Docker CLI 或 Docker Engine 本身的代理设置。请参见 [环境变量](/reference/cli/docker/#environment-variables) 与[为 Docker 守护进程配置代理](/manuals/engine/daemon/proxy.md) 以分别配置 CLI 与守护进程的代理。

### 使用代理配置运行容器

启动容器时，其代理相关的环境变量会根据 `~/.docker/config.json` 中的配置自动设置：

例如，基于前文的[示例配置](#configure-the-docker-client)，容器中可看到如下环境变量：

```console
$ docker run --rm alpine sh -c 'env | grep -i  _PROXY'
https_proxy=http://proxy.example.com:3129
HTTPS_PROXY=http://proxy.example.com:3129
http_proxy=http://proxy.example.com:3128
HTTP_PROXY=http://proxy.example.com:3128
no_proxy=*.test.example.com,.example.org,127.0.0.0/8
NO_PROXY=*.test.example.com,.example.org,127.0.0.0/8
```

### 使用代理配置进行构建

执行构建时，构建上下文会根据 Docker 客户端配置文件中的代理设置，自动预填充相关的构建参数：

同样基于前文的[示例配置](#configure-the-docker-client)，构建阶段可见如下输出：

```console
$ docker build \
  --no-cache \
  --progress=plain \
  - <<EOF
FROM alpine
RUN env | grep -i _PROXY
EOF
```

```console
#5 [2/2] RUN env | grep -i _PROXY
#5 0.100 HTTPS_PROXY=https://proxy.example.com:3129
#5 0.100 no_proxy=*.test.example.com,.example.org,127.0.0.0/8
#5 0.100 NO_PROXY=*.test.example.com,.example.org,127.0.0.0/8
#5 0.100 https_proxy=https://proxy.example.com:3129
#5 0.100 http_proxy=http://proxy.example.com:3128
#5 0.100 HTTP_PROXY=http://proxy.example.com:3128
#5 DONE 0.1s
```

### 按守护进程分别配置代理

`~/.docker/config.json` 中 `proxies` 下的 `default` 键会为客户端连接到的所有守护进程提供默认代理设置。若需为某个特定守护进程单独配置，请将该守护进程地址作为键名：

下面的示例同时配置了一个全局默认代理，以及对地址为 `tcp://docker-daemon1.example.com` 的守护进程设置了 no-proxy 覆盖：

```json
{
 "proxies": {
   "default": {
     "httpProxy": "http://proxy.example.com:3128",
     "httpsProxy": "https://proxy.example.com:3129",
     "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
   },
   "tcp://docker-daemon1.example.com": {
     "noProxy": "*.internal.example.net"
   }
 }
}
```

## 使用 CLI 设置代理

除了[在 Docker 客户端中配置](#configure-the-docker-client)，也可以在调用 `docker build` 与 `docker run` 时通过命令行直接指定代理：

命令行方式在构建时使用 `--build-arg`，在运行容器时使用 `--env`：

```console
$ docker build --build-arg HTTP_PROXY="http://proxy.example.com:3128" .
$ docker run --env HTTP_PROXY="http://proxy.example.com:3128" redis
```

可用的代理相关构建参数，详见 [`Dockerfile` 预定义 ARG](/reference/dockerfile.md#predefined-args)。这些值只在构建容器中可用，不会写入最终产物。

## 将代理作为构建期环境变量

不要在 `Dockerfile` 中使用 `ENV` 指令设置构建期代理，应改用构建参数。

把代理作为环境变量写入镜像会导致配置固化；若该代理为内网代理，基于该镜像启动的容器在其他环境中可能无法访问。

此外，把代理配置写入镜像还存在安全风险，因为其中可能包含敏感信息。
