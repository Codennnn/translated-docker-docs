---
title: 导出器概览
linkTitle: 导出器
weight: 90
description: 构建导出器用于定义构建结果的输出格式
keywords: build, buildx, buildkit, exporter, image, registry, local, tar, oci, docker, cacheonly
aliases:
  - /build/building/exporters/
---

导出器会将你的构建结果保存为指定的输出类型。你可以通过
[`--output` 命令行选项](/reference/cli/docker/buildx/build.md#output)
来指定要使用的导出器。
Buildx 支持以下导出器：

- `image`：将构建结果导出为容器镜像。
- `registry`：将构建结果导出为容器镜像，并推送到指定的仓库。
- `local`：将构建结果的根文件系统导出到本地目录。
- `tar`：将构建结果的根文件系统打包为本地 tar 包。
- `oci`：以
  [OCI 镜像布局](https://github.com/opencontainers/image-spec/blob/v1.0.1/image-layout.md)
  的格式导出到本地文件系统。
- `docker`：以
  [Docker Image Specification v1.2.0](https://github.com/moby/moby/blob/v25.0.0/image/spec/v1.2.md)
  的格式导出到本地文件系统。
- `cacheonly`：不导出任何构建输出，仅执行构建并创建缓存。

## 使用导出器

要指定导出器，使用以下命令语法：

```console
$ docker buildx build --tag <registry>/<image> \
  --output type=<TYPE> .
```

大多数常见场景无需显式指定导出器。只有当你打算自定义输出，或想将结果保存到磁盘时，才需要指定。
`--load` 与 `--push` 选项可以让 Buildx 自动推断应使用的导出器及其设置。

例如，当你在 `--tag` 的同时使用 `--push` 时，Buildx 会自动使用 `image` 导出器，
并将导出器配置为把结果推送到指定仓库。

若要充分利用 BuildKit 提供的各类导出器，可以使用 `--output` 标志来配置导出器参数。

## 适用场景

每种导出器均面向不同的使用场景。下面列举一些常见情形，以及如何使用导出器生成所需输出。

### 加载到镜像存储

Buildx 经常用于构建可加载到镜像存储的容器镜像。这时可使用 `docker` 导出器。
以下示例展示如何使用 `docker` 导出器构建镜像，并通过 `--output` 将镜像加载到本地镜像存储：

```console
$ docker buildx build \
  --output type=docker,name=<registry>/<image> .
```

如果你提供了 `--tag` 与 `--load` 选项，Buildx CLI 会自动使用 `docker` 导出器并加载到镜像存储：

```console
$ docker buildx build --tag <registry>/<image> --load .
```

使用 `docker` 驱动进行构建时，镜像会自动加载到本地镜像存储。

加载到镜像存储的镜像在构建完成后即可被 `docker run` 使用，运行 `docker images` 也能看到它们。

### 推送到仓库

要将构建好的镜像推送到容器仓库，你可以使用 `registry` 或 `image` 导出器。

当你向 Buildx CLI 传入 `--push` 选项时，即指示 BuildKit 将构建好的镜像推送到指定仓库：

```console
$ docker buildx build --tag <registry>/<image> --push .
```

其底层使用 `image` 导出器并设置 `push` 参数。等价于使用 `--output` 的长格式：

```console
$ docker buildx build \
  --output type=image,name=<registry>/<image>,push=true .
```

你也可以使用 `registry` 导出器，效果相同：

```console
$ docker buildx build \
  --output type=registry,name=<registry>/<image> .
```

### 将镜像布局导出为文件

你可以使用 `oci` 或 `docker` 导出器，将构建结果以镜像布局的形式保存到本地文件系统。
这两种导出器都会生成包含相应镜像布局的 tar 包。`dest` 参数用于指定输出 tar 包的目标路径。

```console
$ docker buildx build --output type=oci,dest=./image.tar .
[+] Building 0.8s (7/7) FINISHED
 ...
 => exporting to oci image format                                                                     0.0s
 => exporting layers                                                                                  0.0s
 => exporting manifest sha256:c1ef01a0a0ef94a7064d5cbce408075730410060e253ff8525d1e5f7e27bc900        0.0s
 => exporting config sha256:eadab326c1866dd247efb52cb715ba742bd0f05b6a205439f107cf91b3abc853          0.0s
 => sending tarball                                                                                   0.0s
$ mkdir -p out && tar -C out -xf ./image.tar
$ tree out
out
├── blobs
│   └── sha256
│       ├── 9b18e9b68314027565b90ff6189d65942c0f7986da80df008b8431276885218e
│       ├── c78795f3c329dbbbfb14d0d32288dea25c3cd12f31bd0213be694332a70c7f13
│       ├── d1cf38078fa218d15715e2afcf71588ee482352d697532cf316626164699a0e2
│       ├── e84fa1df52d2abdfac52165755d5d1c7621d74eda8e12881f6b0d38a36e01775
│       └── fe9e23793a27fe30374308988283d40047628c73f91f577432a0d05ab0160de7
├── index.json
├── manifest.json
└── oci-layout
```

### 导出文件系统

如果你不想基于构建结果生成镜像，而是直接导出构建出的文件系统，可以使用 `local` 与 `tar` 导出器。

`local` 导出器会将文件系统解包到指定位置的目录结构中；`tar` 导出器会创建一个 tar 包归档文件。

```console
$ docker buildx build --output type=local,dest=<path/to/output> .
```

`local` 导出器在[多阶段构建](../building/multi-stage.md)中很有用，
因为它允许你仅导出最少量的构建工件，例如自包含的二进制文件。

### 仅缓存导出

当你只想执行一次构建而不导出任何输出时，可以使用 `cacheonly` 导出器。
例如用于试运行构建，或先执行构建、随后再通过其它命令创建导出。
`cacheonly` 导出器会创建构建缓存，因此后续构建将会非常快速。

```console
$ docker buildx build --output type=cacheonly
```

如果你没有指定导出器，也没有提供会自动选择导出器的便捷选项（如 `--load`），
Buildx 会默认使用 `cacheonly` 导出器。例外情况是使用 `docker` 驱动进行构建，
此时默认使用 `docker` 导出器。

当默认使用 `cacheonly` 时，Buildx 会记录一条警告：

```console
$ docker buildx build .
WARNING: No output specified with docker-container driver.
         Build result will only remain in the build cache.
         To push result image into registry use --push or
         to load image into docker use --load
```

## 多个导出器

{{< summary-bar feature_name="Build multiple exporters" >}}

在一次构建中你可以多次指定 `--output` 标志来使用多个导出器。
这需要 **Buildx 与 BuildKit** 均为 0.13.0 或更高版本。

下面的示例在一次构建中使用了三种不同的导出器：

- 使用 `registry` 导出器将镜像推送到仓库
- 使用 `local` 导出器把构建结果解出到本地文件系统
- 使用 `--load`（`image` 导出器的简写）将结果加载到本地镜像存储

```console
$ docker buildx build \
  --output type=registry,tag=<registry>/<image> \
  --output type=local,dest=<path/to/output> \
  --load .
```

## 配置选项

本节介绍导出器可用的一些配置选项。

这里描述的选项至少适用于两种或以上的导出器类型。
此外，不同导出器类型还支持各自的特定参数。请参阅各导出器的详细页面，
了解哪些配置参数适用。

这里介绍的通用参数包括：

- [压缩](#压缩)
- [OCI 媒体类型](#oci-媒体类型)

### 压缩

当你导出压缩后的输出时，可以配置具体的压缩算法与级别。
默认值通常能提供良好的开箱体验，但你也可以根据场景在存储与计算成本之间权衡。
调整压缩参数可以减少所需存储空间、提升镜像下载速度，但会增加构建时间。

要选择压缩算法，使用 `compression` 选项。例如，使用 `compression=zstd` 构建 `image`：

```console
$ docker buildx build \
  --output type=image,name=<registry>/<image>,push=true,compression=zstd .
```

在 `compression` 参数旁配合 `compression-level=<value>` 来选择压缩级别（取决于算法支持）：

- `gzip` 与 `estargz`：0-9
- `zstd`：0-22

一般而言，数字越大，生成的文件越小，但压缩所需时间越长。

使用 `force-compression=true` 选项可强制对从先前镜像导入的层重新压缩，
当你请求的压缩算法与先前算法不同时尤为有用。

> [!NOTE]
>
> `gzip` 与 `estargz` 压缩方法使用 [`compress/gzip` 包](https://pkg.go.dev/compress/gzip)，
> `zstd` 使用 [`github.com/klauspost/compress/zstd` 包](https://github.com/klauspost/compress/tree/master/zstd)。

### OCI 媒体类型

`image`、`registry`、`oci` 与 `docker` 导出器都会创建容器镜像。
这些导出器既支持 Docker 媒体类型（默认），也支持 OCI 媒体类型。

要以 OCI 媒体类型导出镜像，请使用 `oci-mediatypes` 属性：

```console
$ docker buildx build \
  --output type=image,name=<registry>/<image>,push=true,oci-mediatypes=true .
```

## 下一步

阅读各导出器的详细说明，了解其工作原理与用法：

- [镜像与仓库导出器](image-registry.md)
- [OCI 与 Docker 导出器](oci-docker.md)
- [本地与 tar 导出器](local-tar.md)
