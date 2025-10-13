---
title: 自定义 Dockerfile 语法
description: 深入理解 Dockerfile 前端，并了解如何使用自定义前端
keywords: build, buildkit, dockerfile, frontend
aliases:
  - /build/buildkit/dockerfile-frontend/
  - /build/dockerfile/frontend/
---

## Dockerfile 前端（frontend）

BuildKit 支持从容器镜像中动态加载前端。要使用外部的 Dockerfile 前端，
请在你的 [Dockerfile](/reference/dockerfile.md) 第一行设置指向特定镜像的
[`syntax` 指令](/reference/dockerfile.md#syntax)：

```dockerfile
# syntax=[remote image reference]
```

例如：

```dockerfile
# syntax=docker/dockerfile:1
# syntax=docker.io/docker/dockerfile:1
# syntax=example.com/user/repo:tag@sha256:abcdef...
```

你也可以使用预定义的构建参数 `BUILDKIT_SYNTAX`，在命令行设置前端镜像引用：

```console
$ docker build --build-arg BUILDKIT_SYNTAX=docker/dockerfile:1 .
```

这行语句用于指定用于解析与构建该 Dockerfile 的语法实现位置。
BuildKit 后端可无缝使用以 Docker 镜像分发的外部实现，这些实现会在容器沙箱环境中执行。

自定义 Dockerfile 实现可以让你：

- 无需更新 Docker 守护进程即可自动获得缺陷修复
- 确保所有用户使用相同的实现来构建你的 Dockerfile
- 在无需更新 Docker 守护进程的情况下使用最新特性
- 在集成进 Docker 守护进程之前，尝试新特性或第三方特性
- 使用[替代的构建定义，或自行创建](https://github.com/moby/buildkit#exploring-llb)
- 构建你自己的、带自定义特性的 Dockerfile 前端

> [!NOTE]
>
> BuildKit 自带一个内置的 Dockerfile 前端，但推荐使用外部镜像，
> 以确保 builder 与用户端使用相同版本，并能在无需等待 BuildKit 或 Docker Engine 发布新版本的情况下自动获得修复。

## 官方发布

Docker 在 Docker Hub 的 `docker/dockerfile` 仓库下分发可用于构建 Dockerfile 的官方镜像。
新镜像通过两个渠道发布：`stable` 与 `labs`。

### 稳定（Stable）渠道

`stable` 渠道遵循[语义化版本](https://semver.org)。例如：

- `docker/dockerfile:1`：始终跟随最新的 `1.x.x` 次版本与补丁版本。
- `docker/dockerfile:1.2`：始终跟随最新的 `1.2.x` 补丁版本；当 `1.3.0` 发布后停止更新。
- `docker/dockerfile:1.2.1`：不可变；永不更新。

我们推荐使用 `docker/dockerfile:1`，它始终指向语法版本 1 的最新稳定版，
并在该大版本周期内同时接收“次版本”和“补丁版本”的更新。
BuildKit 在构建时会自动检查语法更新，确保你始终使用最新版本。

如果使用了 `1.2` 或 `1.2.1` 这样的特定版本，要继续接收缺陷修复与新特性，
则需要手动更新 Dockerfile。旧版本 Dockerfile 与新版本 builder 依然兼容。

### Labs 渠道

`labs` 渠道提供尚未进入 `stable` 渠道的 Dockerfile 特性的抢先体验。
`labs` 镜像与稳定版同时发布，遵循相同的版本模式，但带有 `-labs` 后缀，例如：

- `docker/dockerfile:labs`：`labs` 渠道的最新发布。
- `docker/dockerfile:1-labs`：等同于 `dockerfile:1`，但启用了实验性特性。
- `docker/dockerfile:1.2-labs`：等同于 `dockerfile:1.2`，但启用了实验性特性。
- `docker/dockerfile:1.2.1-labs`：不可变；永不更新。等同于 `dockerfile:1.2.1`，但启用了实验性特性。

请选择适合你需求的渠道。如果你希望尽早使用新特性，请选择 `labs` 渠道。
`labs` 渠道包含 `stable` 的全部特性，并额外提供抢先体验的功能。
其中稳定特性遵循[语义化版本](https://semver.org)，但抢先体验特性不遵循；
新版本可能不向后兼容。建议固定版本以避免破坏性变更带来的影响。

## 其他资源

关于 `labs` 特性、master 构建与 nightly 特性发布的文档，参见
[BuildKit 源码仓库的说明](https://github.com/moby/buildkit/blob/master/README.md)。
完整的可用镜像列表，参见 Docker Hub 上的
[`docker/dockerfile` 仓库](https://hub.docker.com/r/docker/dockerfile)，
以及用于开发构建的 [`docker/dockerfile-upstream` 仓库](https://hub.docker.com/r/docker/dockerfile-upstream)。
