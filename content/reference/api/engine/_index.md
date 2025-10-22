---
title: Docker Engine API
description: 了解如何使用您选择的编程语言的 Docker Engine API 和 SDK。
keywords: 开发, api, Docker Engine API, API 版本, SDK 版本
aliases:
  - /reference/api/docker_remote_api/
  - /reference/api/docker_remote_api_v1.0/
  - /reference/api/docker_remote_api_v1.1/
  - /reference/api/docker_remote_api_v1.2/
  - /reference/api/docker_remote_api_v1.3/
  - /reference/api/docker_remote_api_v1.4/
  - /reference/api/docker_remote_api_v1.5/
  - /reference/api/docker_remote_api_v1.6/
  - /reference/api/docker_remote_api_v1.7/
  - /reference/api/docker_remote_api_v1.8/
  - /reference/api/docker_remote_api_v1.9/
  - /reference/api/docker_remote_api_v1.10/
  - /reference/api/docker_remote_api_v1.11/
  - /reference/api/docker_remote_api_v1.12/
  - /reference/api/docker_remote_api_v1.13/
  - /reference/api/docker_remote_api_v1.14/
  - /reference/api/docker_remote_api_v1.15/
  - /reference/api/docker_remote_api_v1.16/
  - /reference/api/docker_remote_api_v1.17/
  - /engine/reference/api/
  - /engine/reference/api/docker_remote_api/
  - /engine/api/
---

Docker 提供了一个用于与 Docker 守护进程交互的 API（称为 Docker Engine API），以及 Go 和 Python 的 SDK。这些 SDK 让您能够高效地构建和扩展 Docker 应用程序和解决方案。如果 Go 或 Python 不适合您，您可以直接使用 Docker Engine API。

有关 Docker Engine SDK 的信息，请参阅[使用 Docker Engine SDK 进行开发](sdk/_index.md)。

Docker Engine API 是一个 RESTful API，可以通过 HTTP 客户端（如 `wget` 或 `curl`）访问，或者使用大多数现代编程语言内置的 HTTP 库。

## 查看 API 参考

您可以
[查看最新版本 API 的参考文档](/reference/api/engine/version/v{{% param latest_engine_api_version %}}.md)
或者[选择特定版本](/reference/api/engine/version-history/)。

## 版本化 API 和 SDK

您应该使用的 Docker Engine API 版本取决于您的 Docker 守护进程和 Docker 客户端的版本。

给定版本的 Docker Engine SDK 支持特定版本的 Docker Engine API，以及所有早期版本。如果发生破坏性更改，它们会被显著地记录下来。

> [!NOTE]
>
> Docker 守护进程和客户端不一定需要始终保持相同版本。但是，请记住以下几点。
>
> - 如果守护进程比客户端新，客户端将不知道守护进程中的新功能或已弃用的 API 端点。
>
> - 如果客户端比守护进程新，客户端可能会请求守护进程不知道的 API 端点。

当添加新功能时，会发布新版本的 API。Docker API 是向后兼容的，因此您不需要更新使用 API 的代码，除非您需要利用新功能。

要查看您的 Docker 守护进程和客户端支持的最高 API 版本，请使用 `docker version`：

```console
$ docker version
Client: Docker Engine - Community
 Version:           28.5.1
 API version:       1.51
 Go version:        go1.24.8
 Git commit:        e180ab8
 Built:             Wed Oct  8 12:18:19 2025
 OS/Arch:           linux/arm64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          28.5.1
  API version:      1.51 (minimum version 1.24)
  Go version:       go1.24.8
  Git commit:       f8215cc
  Built:            Wed Oct  8 12:18:19 2025
  OS/Arch:          linux/arm64
  ...
```

您可以通过以下任何方式指定要使用的 API 版本：

- 使用 SDK 时，使用最新版本。至少使用包含您所需功能的 API 版本。
- 直接使用 `curl` 时，将版本指定为 URL 的第一部分。例如，如果端点是 `/containers/`，您可以使用 `/v{{% param "latest_engine_api_version" %}}/containers/`。
- 要强制 Docker CLI 或 Docker Engine SDK 使用比 `docker version` 报告的版本更旧的 API 版本，请将环境变量 `DOCKER_API_VERSION` 设置为正确的版本。这适用于 Linux、Windows 或 macOS 客户端。

  {{% apiVersionPrevious.inline %}}
  {{- $version := site.Params.latest_engine_api_version }}
  {{- $parts := strings.Split $version "." }}
  {{- $major := cast.ToInt (index $parts 0) }}
  {{- $minor := cast.ToInt (index $parts 1) }}
  ```console
  $ DOCKER_API_VERSION={{ $major }}.{{ math.Sub $minor 1 }}
  ```
  {{% /apiVersionPrevious.inline %}}

  当设置了环境变量时，即使 Docker 守护进程支持更新的版本，也会使用该版本的 API。此环境变量禁用 API 版本协商，因此只有在必须使用特定版本的 API 或用于调试目的时才应使用它。

- Docker Go SDK 允许您启用 API 版本协商，自动选择客户端和正在使用的 Docker Engine 都支持的 API 版本。
- 对于 SDK，您还可以以编程方式将 API 版本指定为 `client` 对象的参数。请参阅
  [Go 构造函数](https://pkg.go.dev/github.com/docker/docker/client#NewClientWithOpts)
  或
  [Python SDK 文档中的 `client`](https://docker-py.readthedocs.io/en/stable/client.html)。

### API 版本矩阵

| Docker 版本 | 最高 API 版本                          | 更改日志                                                         |
|:---------------|:---------------------------------------------|:-------------------------------------------------------------------|
| 28.3           | [1.51](/reference/api/engine/version/v1.51/) | [更改](/reference/api/engine/version-history/#v151-api-changes) |
| 28.2           | [1.50](/reference/api/engine/version/v1.50/) | [更改](/reference/api/engine/version-history/#v150-api-changes) |
| 28.1           | [1.49](/reference/api/engine/version/v1.49/) | [更改](/reference/api/engine/version-history/#v149-api-changes) |
| 28.0           | [1.48](/reference/api/engine/version/v1.48/) | [更改](/reference/api/engine/version-history/#v148-api-changes) |
| 27.5           | [1.47](/reference/api/engine/version/v1.47/) | [更改](/reference/api/engine/version-history/#v147-api-changes) |
| 27.4           | [1.47](/reference/api/engine/version/v1.47/) | [更改](/reference/api/engine/version-history/#v147-api-changes) |
| 27.3           | [1.47](/reference/api/engine/version/v1.47/) | [更改](/reference/api/engine/version-history/#v147-api-changes) |
| 27.2           | [1.47](/reference/api/engine/version/v1.47/) | [更改](/reference/api/engine/version-history/#v147-api-changes) |
| 27.1           | [1.46](/reference/api/engine/version/v1.46/) | [更改](/reference/api/engine/version-history/#v146-api-changes) |
| 27.0           | [1.46](/reference/api/engine/version/v1.46/) | [更改](/reference/api/engine/version-history/#v146-api-changes) |
| 26.1           | [1.45](/reference/api/engine/version/v1.45/) | [更改](/reference/api/engine/version-history/#v145-api-changes) |
| 26.0           | [1.45](/reference/api/engine/version/v1.45/) | [更改](/reference/api/engine/version-history/#v145-api-changes) |
| 25.0           | [1.44](/reference/api/engine/version/v1.44/) | [更改](/reference/api/engine/version-history/#v144-api-changes) |
| 24.0           | [1.43](/reference/api/engine/version/v1.43/) | [更改](/reference/api/engine/version-history/#v143-api-changes) |
| 23.0           | [1.42](/reference/api/engine/version/v1.42/) | [更改](/reference/api/engine/version-history/#v142-api-changes) |
| 20.10          | [1.41](/reference/api/engine/version/v1.41/) | [更改](/reference/api/engine/version-history/#v141-api-changes) |
| 19.03          | [1.40](/reference/api/engine/version/v1.40/) | [更改](/reference/api/engine/version-history/#v140-api-changes) |
| 18.09          | [1.39](/reference/api/engine/version/v1.39/) | [更改](/reference/api/engine/version-history/#v139-api-changes) |
| 18.06          | [1.38](/reference/api/engine/version/v1.38/) | [更改](/reference/api/engine/version-history/#v138-api-changes) |
| 18.05          | [1.37](/reference/api/engine/version/v1.37/) | [更改](/reference/api/engine/version-history/#v137-api-changes) |
| 18.04          | [1.37](/reference/api/engine/version/v1.37/) | [更改](/reference/api/engine/version-history/#v137-api-changes) |
| 18.03          | [1.37](/reference/api/engine/version/v1.37/) | [更改](/reference/api/engine/version-history/#v137-api-changes) |
| 18.02          | [1.36](/reference/api/engine/version/v1.36/) | [更改](/reference/api/engine/version-history/#v136-api-changes) |
| 17.12          | [1.35](/reference/api/engine/version/v1.35/) | [更改](/reference/api/engine/version-history/#v135-api-changes) |
| 17.11          | [1.34](/reference/api/engine/version/v1.34/) | [更改](/reference/api/engine/version-history/#v134-api-changes) |
| 17.10          | [1.33](/reference/api/engine/version/v1.33/) | [更改](/reference/api/engine/version-history/#v133-api-changes) |
| 17.09          | [1.32](/reference/api/engine/version/v1.32/) | [更改](/reference/api/engine/version-history/#v132-api-changes) |
| 17.07          | [1.31](/reference/api/engine/version/v1.31/) | [更改](/reference/api/engine/version-history/#v131-api-changes) |
| 17.06          | [1.30](/reference/api/engine/version/v1.30/) | [更改](/reference/api/engine/version-history/#v130-api-changes) |
| 17.05          | [1.29](/reference/api/engine/version/v1.29/) | [更改](/reference/api/engine/version-history/#v129-api-changes) |
| 17.04          | [1.28](/reference/api/engine/version/v1.28/) | [更改](/reference/api/engine/version-history/#v128-api-changes) |
| 17.03.1        | [1.27](/reference/api/engine/version/v1.27/) | [更改](/reference/api/engine/version-history/#v127-api-changes) |
| 17.03          | [1.26](/reference/api/engine/version/v1.27/) | [更改](/reference/api/engine/version-history/#v126-api-changes) |
| 1.13.1         | [1.26](/reference/api/engine/version/v1.26/) | [更改](/reference/api/engine/version-history/#v126-api-changes) |
| 1.13           | [1.25](/reference/api/engine/version/v1.26/) | [更改](/reference/api/engine/version-history/#v125-api-changes) |
| 1.12           | [1.24](/reference/api/engine/version/v1.24/) | [更改](/reference/api/engine/version-history/#v124-api-changes) |

### 已弃用的 API 版本

v1.24 之前的 API 版本已[弃用](/engine/deprecated/#deprecate-legacy-api-versions)。
您可以在 GitHub 上的代码仓库中找到已弃用 API 版本的存档文档：

- [API 版本 1.23 及更早版本的文档](https://github.com/moby/moby/tree/v25.0.0/docs/api)。
- [API 版本 1.17 及更早版本的文档](https://github.com/moby/moby/tree/v1.9.1/docs/reference/api)。
