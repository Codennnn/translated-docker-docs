---
title: GitHub Actions 缓存
description: 在 CI 中使用 GitHub Actions 缓存管理构建缓存
keywords: build, buildx, cache, backend, gha, github, actions
aliases:
  - /build/building/cache/backends/gha/
---

{{< summary-bar feature_name="GitHub Actions cache" >}}

GitHub Actions 缓存使用由 GitHub 提供的
[actions/cache](https://github.com/actions/cache) 或其他支持 GitHub Actions 缓存协议的服务。
只要你的使用场景符合
[GitHub 规定的容量与使用限制](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy)，
这是在 GitHub Actions 工作流中推荐采用的缓存方式。

默认的 `docker` 驱动不支持该缓存存储后端。
如需使用此功能，请使用其他驱动创建新的构建器；
详情参见[构建驱动](/manuals/build/builders/drivers/_index.md)。

## 概要

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=gha[,parameters...] \
  --cache-from type=gha[,parameters...] .
```

下表描述了可通过 `--cache-to` 与 `--cache-from` 传入的逗号分隔参数（CSV）：

| 名称           | 选项                     | 类型        | 默认值                                         | 说明                                                                 |
|----------------|--------------------------|-------------|------------------------------------------------|----------------------------------------------------------------------|
| `url`          | `cache-to`,`cache-from`  | String      | `$ACTIONS_CACHE_URL` 或 `$ACTIONS_RESULTS_URL` | 缓存服务器 URL，见[认证][1]。当 `version=2` 时忽略。                 |
| `url_v2`       | `cache-to`,`cache-from`  | String      | `$ACTIONS_RESULTS_URL`                         | v2 缓存服务器 URL，见[认证][1]。                                     |
| `token`        | `cache-to`,`cache-from`  | String      | `$ACTIONS_RUNTIME_TOKEN`                       | 访问令牌，见[认证][1]。                                             |
| `scope`        | `cache-to`,`cache-from`  | String      | `buildkit`                                     | 缓存对象所属作用域，见[作用域][2]。                                  |
| `mode`         | `cache-to`               | `min`,`max` | `min`                                          | 导出的缓存层范围，见[缓存模式][3]。                                  |
| `ignore-error` | `cache-to`               | Boolean     | `false`                                        | 忽略因缓存导出失败引发的错误。                                       |
| `timeout`      | `cache-to`,`cache-from`  | String      | `10m`                                          | 导入或导出缓存的最长持续时间，超过即超时。                           |
| `repository`   | `cache-to`               | String      |                                                | 用于存储缓存的 GitHub 仓库。                                         |
| `ghtoken`      | `cache-to`               | String      |                                                | 访问 GitHub API 所需的 GitHub Token。                                |
| `version`      | `cache-to`,`cache-from`  | String      | `1`（若设置 `$ACTIONS_CACHE_SERVICE_V2` 则为 `2`） | 选择 GitHub Actions 缓存版本，见[版本][4]。                          |

[1]: #authentication
[2]: #scope
[3]: _index.md#cache-mode
[4]: #version

## 认证 {#authentication}

如果未显式指定 `url`、`url_v2` 或 `token` 参数，`gha` 缓存后端会回退读取环境变量。
如果你在工作流的内联步骤中手动执行 `docker buildx`，则必须显式暴露这些变量。
建议使用 [`crazy-max/ghaction-github-runtime`](https://github.com/crazy-max/ghaction-github-runtime)
这一 GitHub Action 来辅助暴露相关变量。

## 作用域 {#scope}

作用域（Scope）是用于标识缓存对象的键。默认值为 `buildkit`。
如果你构建多个镜像，每次构建都会覆盖上一次的缓存，最终只保留最后一个缓存。

若要同时保留多次构建的缓存，你可以为该作用域指定特定名称。
下例将作用域设置为镜像名，以确保每个镜像都有独立的缓存：

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=gha,url=...,token=...,scope=image \
  --cache-from type=gha,url=...,token=...,scope=image .
$ docker buildx build --push -t <registry>/<image2> \
  --cache-to type=gha,url=...,token=...,scope=image2 \
  --cache-from type=gha,url=...,token=...,scope=image2 .
```

GitHub 的[缓存访问限制](https://docs.github.com/en/actions/advanced-guides/caching-dependencies-to-speed-up-workflows#restrictions-for-accessing-a-cache)
同样适用。工作流仅能访问当前分支、基准分支与默认分支的缓存。

## 版本 {#version}

如果未显式设置 `version`，默认使用 v1。
若环境变量 `$ACTIONS_CACHE_SERVICE_V2` 被设置为可解释为 `true` 的值（`1`、`true`、`yes`），则自动使用 v2。

同一时间仅有一个 URL 生效：

 - v1 使用 `url`（默认 `$ACTIONS_CACHE_URL`）。
 - v2 使用 `url_v2`（默认 `$ACTIONS_RESULTS_URL`）。

### 搭配 `docker/build-push-action` 使用

使用
[`docker/build-push-action`](https://github.com/docker/build-push-action)
时，`url` 与 `token` 参数会自动填充，无需手动指定或添加额外的变通处理。

示例：

```yaml
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: "<registry>/<image>:latest"
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

## 避免 GitHub Actions 缓存 API 限流

GitHub 的
[使用限制与淘汰策略](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy)
会在一段时间后清理过期的缓存条目。默认情况下，`gha` 缓存后端会通过 GitHub Actions 缓存 API 检查缓存条目的状态。

如果在短时间内发起过多请求，GitHub Actions 缓存 API 会触发速率限制。
当使用 `gha` 缓存后端进行构建时，由于频繁查询缓存键，容易出现这一情况。

```text
#31 exporting to GitHub Actions Cache
#31 preparing build cache for export
#31 preparing build cache for export 600.3s done
#31 ERROR: maximum timeout reached
------
 > exporting to GitHub Actions Cache:
------
ERROR: failed to solve: maximum timeout reached
make: *** [Makefile:35: release] Error 1
Error: Process completed with exit code 2.
```

为缓解该问题，你可以向 BuildKit 提供 GitHub Token，使其改用标准 GitHub API 检查缓存键，
从而减少对缓存 API 的请求次数。

可通过 `ghtoken` 参数提供 GitHub Token，并通过 `repository` 指定用于存储缓存的仓库。
`ghtoken` 需要具备 `repo` 作用域，以访问 GitHub Actions 缓存 API。

当你通过 `docker/build-push-action` 进行构建时，`ghtoken` 会自动使用 `secrets.GITHUB_TOKEN` 的值。
你也可以通过 `github-token` 入参手动设置 `ghtoken`，如下所示：

```yaml
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: "<registry>/<image>:latest"
    cache-from: type=gha
    cache-to: type=gha,mode=max
    github-token: ${{ secrets.MY_CUSTOM_TOKEN }}
```

## 延伸阅读

缓存入门请参阅《[Docker 构建缓存](../_index.md)》。

关于 `gha` 缓存后端的更多信息，请参阅
[BuildKit README](https://github.com/moby/buildkit#github-actions-cache-experimental)。

关于在 Docker 中使用 GitHub Actions，参见
[GitHub Actions 简介](../../ci/github-actions/_index.md)
