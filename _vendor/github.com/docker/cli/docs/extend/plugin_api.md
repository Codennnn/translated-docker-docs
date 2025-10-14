---
title: Docker 插件 API
description: "如何编写 Docker 插件扩展"
keywords: "API, Usage, plugins, documentation, developer"
---

Docker 插件是运行在进程外（out-of-process）的扩展，用于为 Docker Engine 增加功能。

本文描述 Docker Engine 的插件 API。若需了解由 Docker Engine 托管的插件，请参阅《[Docker Engine 插件系统](_index.md)》。

本页面向希望开发自定义 Docker 插件的读者。若你只是想了解或使用插件，请查看[这里](legacy_plugins.md)。

## 什么是插件

插件是与 Docker 守护进程（Docker daemon）运行在同一主机或不同主机上的一个进程。它通过在守护进程主机的插件目录中放置文件来完成自注册，插件目录见[插件发现](#plugin-discovery)。

插件名称具有可读性，通常是简短的小写字符串，例如 `flocker` 或 `weave`。

插件既可以在容器内运行，也可以在容器外运行；目前推荐在容器外运行。

## 插件发现 {#plugin-discovery}

当用户或容器按名称尝试使用某个插件时，Docker 会在插件目录中查找该插件以进行发现。

插件目录中可以放置三类文件：

- `.sock` 文件：Unix 域套接字。
- `.spec` 文件：包含 URL 的文本文件，例如 `unix:///other.sock` 或 `tcp://localhost:8080`。
- `.json` 文件：包含插件完整 JSON 规范的文本文件。

带 Unix 域套接字（`.sock`）的插件必须与 Docker 守护进程运行在同一主机上。
若使用 `.spec` 或 `.json`，在指定远程 URL 的情况下，插件可以运行在其他主机上。

Unix 域套接字文件必须位于 `/run/docker/plugins` 下；而 spec 文件可以位于 `/etc/docker/plugins` 或 `/usr/lib/docker/plugins`。

插件名称由文件名（不含扩展名）决定。

例如，`flocker` 插件可能会创建一个 Unix 套接字：`/run/docker/plugins/flocker.sock`。

如果希望相互隔离定义，也可以为每个插件放在独立子目录中。例如，可将 `flocker` 的套接字放在 `/run/docker/plugins/flocker/flocker.sock`，并仅将 `/run/docker/plugins/flocker` 挂载到 `flocker` 容器内。

Docker 总是首先在 `/run/docker/plugins` 下查找 Unix 套接字；若未找到对应套接字，才会去 `/etc/docker/plugins` 与 `/usr/lib/docker/plugins` 下查找 spec 或 json 文件。一旦找到给定名称的第一个插件定义，即停止继续扫描。

### JSON 规范

以下是插件 JSON 规范示例：

```json
{
  "Name": "plugin-example",
  "Addr": "https://example.com/docker/plugin",
  "TLSConfig": {
    "InsecureSkipVerify": false,
    "CAFile": "/usr/shared/docker/certs/example-ca.pem",
    "CertFile": "/usr/shared/docker/certs/example-cert.pem",
    "KeyFile": "/usr/shared/docker/certs/example-key.pem"
  }
}
```

`TLSConfig` 字段是可选的，仅当该配置存在时才启用 TLS 校验。

## 插件生命周期

插件应在 Docker 启动之前启动，并在 Docker 停止之后再停止。例如，在支持 `systemd` 的平台上打包插件时，可以使用[`systemd` 的依赖关系](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#Before=)来管理启动与停止的先后顺序。

升级插件时，应先停止 Docker 守护进程，升级插件后再重新启动 Docker。

## 插件激活

当首次引用某个插件时——无论是用户按名称引用（例如 `docker run --volume-driver=foo`），还是启动了一个已配置为使用该插件的容器——Docker 会在插件目录中查找该插件，并通过一次“握手”进行激活。参见下文“握手 API”。

插件不会在 Docker 守护进程启动时自动激活，而是按需（惰性）激活，仅在需要时才会激活。

## systemd 套接字激活

插件也可以通过 `systemd` 的套接字进行激活。官方的[插件辅助库](https://github.com/docker/go-plugins-helpers)原生支持套接字激活。要启用套接字激活，需要提供一个 `service` 文件和一个 `socket` 文件。

`service` 文件（例如 `/lib/systemd/system/your-plugin.service`）：

```systemd
[Unit]
Description=Your plugin
Before=docker.service
After=network.target your-plugin.socket
Requires=your-plugin.socket docker.service

[Service]
ExecStart=/usr/lib/docker/your-plugin

[Install]
WantedBy=multi-user.target
```

`socket` 文件（例如 `/lib/systemd/system/your-plugin.socket`）：

```systemd
[Unit]
Description=Your plugin

[Socket]
ListenStream=/run/docker/plugins/your-plugin.sock

[Install]
WantedBy=sockets.target
```

借此，当 Docker 守护进程连接到插件监听的套接字时（例如守护进程首次使用该插件，或插件意外退出后），插件才会被实际启动。

## API 设计

插件 API 采用基于 HTTP 的 RPC 风格 JSON，类似于 webhooks。

请求从 Docker 守护进程发往插件。插件需要实现一个 HTTP 服务器，并将其绑定到“插件发现”一节所述的 Unix 套接字上。

所有请求均为 HTTP `POST` 请求。

API 的版本通过 Accept 头进行约定，目前固定为 `application/vnd.docker.plugins.v1+json`。

## 握手 API

插件通过以下“握手” API 调用完成激活。

### /Plugin.Activate

请求：空请求体

响应：

```json
{
    "Implements": ["VolumeDriver"]
}
```

响应为该插件所实现的 Docker 子系统列表。激活完成后，插件会接收来自这些子系统的事件。

可能的取值包括：

- [`authz`](plugins_authorization.md)
- [`NetworkDriver`](plugins_network.md)
- [`VolumeDriver`](plugins_volume.md)

## 插件调用重试

当调用插件的方法失败时，Docker 会以指数退避方式重试，最长持续 30 秒。这对于将插件打包为容器的场景很有帮助，因为它为插件容器提供了在失败前完成启动的时间，从而避免影响依赖它们的用户容器。

## 插件辅助库

为简化插件开发，Docker 为当前支持的各类插件提供了 `SDK`：参见 [docker/go-plugins-helpers](https://github.com/docker/go-plugins-helpers)。
