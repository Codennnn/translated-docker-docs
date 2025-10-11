---
title: 缓存存储后端
description: |
  通过外部后端管理构建缓存。
  外部缓存可用于创建共享缓存，加速本地内循环与 CI 构建。
keywords: build, buildx, cache, backend, gha, azblob, s3, registry, local
aliases:
  - /build/building/cache/backends/
---

为了保证构建速度，BuildKit 会将构建结果缓存在其自身的内部缓存中。此外，BuildKit 还支持将构建缓存导出到外部位置，便于在后续构建中导入使用。

在 CI/CD 构建环境中，外部缓存几乎是必需品。这类环境的每次运行之间通常几乎没有任何持久化，但我们仍希望尽可能降低镜像构建的耗时。

默认的 `docker` 驱动支持 `inline`、`local`、`registry` 和 `gha` 缓存后端，但前提是你已启用 [containerd 镜像存储](/manuals/desktop/features/containerd.md)。若需使用其他缓存后端，需要选择不同的[驱动](/manuals/build/builders/drivers/_index.md)。

> [!WARNING]
>
> 如果在构建过程中使用机密或凭证，请务必使用专用的
> [`--secret` 选项](/reference/cli/docker/buildx/build.md#secret)进行处理。
> 通过 `COPY` 或 `ARG` 手动管理机密，可能导致凭证泄漏。

## 后端

Buildx 支持以下缓存存储后端：

- `inline`: 将构建缓存内嵌到镜像中。

  内联缓存会与主要构建产物一同推送到相同的位置。
  该方式仅适用于 [`image` 导出器](../../exporters/image-registry.md)。

- `registry`: 将构建缓存封装为独立镜像，并推送到与主要产物分离的专用位置。

- `local`: 将构建缓存写入本地文件系统目录。

- `gha`: 将构建缓存上传到
  [GitHub Actions 缓存](https://docs.github.com/en/rest/actions/cache)（测试版）。

- `s3`: 将构建缓存上传到
  [AWS S3 存储桶](https://aws.amazon.com/s3/)（未发布）。

- `azblob`: 将构建缓存上传到
  [Azure Blob 存储](https://azure.microsoft.com/en-us/services/storage/blobs/)
  （未发布）。

## 命令语法

要使用任一缓存后端，首先在构建时通过
[`--cache-to` 选项](/reference/cli/docker/buildx/build.md#cache-to)
将缓存导出到你选择的存储后端；随后使用
[`--cache-from` 选项](/reference/cli/docker/buildx/build.md#cache-from)
从该后端导入缓存，供本次构建使用。不同于本地 BuildKit 缓存（始终启用），所有缓存存储后端都需要显式导出与显式导入。

示例：使用 `registry` 后端同时导入与导出缓存：

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=registry,ref=<registry>/<cache-image>[,parameters...] \
  --cache-from type=registry,ref=<registry>/<cache-image>[,parameters...] .
```

> [!WARNING]
>
> 一般而言，每个缓存都会写入某个位置。任何位置都不应被写入两次，
> 否则会覆盖此前缓存的数据。若你需要维护多个作用域的缓存
> （例如为每个 Git 分支分别维护缓存），请确保为导出的缓存使用不同的位置。

## 多个缓存

BuildKit 支持多个缓存导出器，你可以将缓存同时推送到多个目的地；
同样也可以从任意多个远程缓存导入。例如，一个常见模式是同时使用当前分支与主分支的缓存。
下面示例展示了使用 registry 缓存后端从多个位置导入缓存：

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=registry,ref=<registry>/<cache-image>:<branch> \
  --cache-from type=registry,ref=<registry>/<cache-image>:<branch> \
  --cache-from type=registry,ref=<registry>/<cache-image>:main .
```

## 配置项

本节介绍在生成缓存导出时可用的一些配置项。这里描述的是至少被两种及以上后端所支持的通用选项。
此外，各后端还支持其特定的参数。请参阅每种后端类型的详细页面，了解适用的配置参数。

常见参数包括：

- [缓存模式](#cache-mode)
- [缓存压缩](#cache-compression)
- [OCI 媒体类型](#oci-media-types)

### 缓存模式 {#cache-mode}

在生成缓存输出时，`--cache-to` 参数接受 `mode` 选项，用于定义要包含在导出缓存中的层。
除 `inline` 缓存外，所有缓存后端均支持此选项。

`mode` 可设置为 `mode=min` 或 `mode=max`。例如，使用 registry 后端并以 `mode=max` 导出缓存：

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=registry,ref=<registry>/<cache-image>,mode=max \
  --cache-from type=registry,ref=<registry>/<cache-image> .
```

该选项仅在使用 `--cache-to` 导出缓存时设置；导入缓存（`--cache-from`）时，相关参数会自动检测。

在 `min` 模式（默认）下，仅缓存会被导入到最终镜像的那些层；
而在 `max` 模式下，所有层都会被缓存，包括中间步骤的层。

`min` 模式的缓存通常更小（有助于加快导入/导出并降低存储成本），
而 `max` 模式更可能获得更多的缓存命中。可根据构建的复杂度与位置进行试验，以找到最适合你的配置。

### 缓存压缩 {#cache-compression}

缓存压缩选项与[导出器的压缩选项](../../exporters/_index.md#compression)相同。
`local` 与 `registry` 缓存后端支持该选项。

例如，使用 `zstd` 压缩 registry 缓存：

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=registry,ref=<registry>/<cache-image>,compression=zstd \
  --cache-from type=registry,ref=<registry>/<cache-image> .
```

### OCI 媒体类型 {#oci-media-types}

缓存的 OCI 相关选项与[导出器的 OCI 选项](../../exporters/_index.md#oci-media-types)相同。
这些选项由 `local` 与 `registry` 缓存后端支持。

例如，若要按 OCI 媒体类型导出缓存，请使用 `oci-mediatypes` 属性：

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=registry,ref=<registry>/<cache-image>,oci-mediatypes=true \
  --cache-from type=registry,ref=<registry>/<cache-image> .
```

该属性仅在 `--cache-to` 标志下有意义。获取缓存时，BuildKit 会自动检测应使用的媒体类型。

默认情况下，按 OCI 媒体类型导出会为缓存镜像生成镜像索引。
部分 OCI 仓库（例如 Amazon ECR）不支持镜像索引媒体类型：`application/vnd.oci.image.index.v1+json`。
如果你将缓存镜像导出到 ECR 或任何不支持镜像索引的仓库，请将 `image-manifest` 设置为 `true`，
以便为缓存镜像生成单一镜像清单（而非镜像索引）：

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=registry,ref=<registry>/<cache-image>,oci-mediatypes=true,image-manifest=true \
  --cache-from type=registry,ref=<registry>/<cache-image> .
```

> [!NOTE]
> 自 BuildKit v0.21 起，`image-manifest` 默认启用。
