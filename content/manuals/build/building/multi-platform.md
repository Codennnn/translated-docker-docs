---
title: 多平台构建
linkTitle: 多平台
weight: 40
description: 介绍什么是多平台构建，以及如何使用 Docker Buildx 执行多平台构建。
keywords: build, buildx, buildkit, multi-platform, cross-platform, cross-compilation, emulation, QEMU, ARM, x86, Windows, Linux, macOS
aliases:
- /build/buildx/multiplatform-images/
- /desktop/multi-arch/
- /docker-for-mac/multi-arch/
- /mackit/multi-arch/
- /build/guide/multi-platform/
---

多平台构建指的是一次构建即可面向多个操作系统或 CPU 架构组合产出镜像。在构建镜像时，这使你能够产出一个可以在多个平台上运行的单一镜像，例如 `linux/amd64`、`linux/arm64` 和 `windows/amd64`。

## 为什么需要多平台构建？

Docker 通过将应用及其依赖打包为容器，解决了“只在我机器上能跑”的问题。这让相同的应用能够在开发、测试与生产等不同环境中一致运行。

但容器化本身只解决了问题的一部分。容器共享宿主机内核，这意味着容器内运行的代码必须与宿主机的架构兼容。这也是为什么在不使用仿真的情况下，无法在 arm64 宿主机上运行 `linux/amd64` 容器，或在 Linux 主机上运行 Windows 容器。

多平台构建通过将同一应用的多个变体打包到一个镜像中来解决这一问题。这样你就可以在不同类型的硬件上运行同一个镜像，例如在本地 x86-64 开发机或云端基于 ARM 的 Amazon EC2 实例上，而无需借助仿真。

### 单平台与多平台镜像的差异 {#difference-between-single-platform-and-multi-platform-images}

多平台镜像与单平台镜像在结构上不同。单平台镜像包含一个清单（manifest），指向一份配置和一组层（layers）。多平台镜像包含一个清单列表（manifest list），指向多个清单；每个清单对应各自的配置与层。

![Multi-platform image structure](/build/images/single-vs-multiplatform-image.svg)

当你将多平台镜像推送到仓库（registry）时，仓库会保存清单列表以及各个子清单。拉取镜像时，仓库返回清单列表，Docker 会根据宿主机架构自动选择正确的变体。例如，在基于 ARM 的树莓派上运行该镜像时，会选择 `linux/arm64` 变体；在一台 x86-64 笔记本上使用 Linux 容器运行同一镜像时，则会选择 `linux/amd64` 变体。

## 先决条件 {#prerequisites}

要构建多平台镜像，首先需要确保 Docker 环境已启用相应支持。通常有两种方式：

- 将“经典（classic）”镜像存储切换为 containerd 镜像存储。
- 创建并使用自定义构建器（builder）。

Docker Engine 的“经典”镜像存储不支持多平台镜像。切换到 containerd 镜像存储后，Docker Engine 才能推送、拉取并构建多平台镜像。

你也可以创建一个支持多平台的构建器驱动（例如 `docker-container` 驱动）的自定义构建器，而无需更改镜像存储。需要注意的是，这种方式虽可构建多平台镜像，但无法将产物加载到 Docker Engine 的镜像存储中；你可以直接使用 `docker build --push` 将其推送到镜像仓库。

{{< tabs >}}
{{< tab name="containerd image store" >}}

启用 containerd 镜像存储的步骤取决于你使用的是 Docker Desktop 还是独立的 Docker Engine：

- 如果使用 Docker Desktop，请在 [Docker Desktop 设置](/manuals/desktop/features/containerd.md) 中启用 containerd 镜像存储。

- 如果使用独立的 Docker Engine，请通过 [守护进程配置文件](/manuals/engine/storage/containerd.md) 启用 containerd 镜像存储。

{{< /tab >}}
{{< tab name="Custom builder" >}}

要创建自定义构建器，可使用 `docker buildx create` 命令创建一个基于 `docker-container` 驱动的构建器：

```console
$ docker buildx create \
  --name container-builder \
  --driver docker-container \
  --bootstrap --use
```

> [!NOTE]
> 使用 `docker-container` 驱动进行的构建不会自动加载到 Docker Engine 的镜像存储中。参见 [构建驱动](/manuals/build/builders/drivers/_index.md) 了解更多信息。

{{< /tab >}}
{{< /tabs >}}

如果你使用的是独立的 Docker Engine，并且需要通过仿真来构建多平台镜像，则还需安装 QEMU，参见[手动安装 QEMU](#install-qemu-manually)。

## 构建多平台镜像 {#build-multi-platform-images}

在触发构建时，使用 `--platform` 标志指定产物的目标平台，例如 `linux/amd64` 与 `linux/arm64`：

```console
$ docker buildx build --platform linux/amd64,linux/arm64 .
```

## 策略 {#strategies}

根据你的场景，你可以通过三种不同策略构建多平台镜像：

1. 使用仿真（通过 [QEMU](#qemu)）
2. 使用包含[多个原生节点](#multiple-native-nodes)的构建器
3. 结合多阶段构建进行[交叉编译](#cross-compilation)

### QEMU {#qemu}

如果你的构建器已支持 QEMU，使用 QEMU 仿真来构建多平台镜像是最容易上手的方式。采用仿真无需修改 Dockerfile，BuildKit 会自动检测可用的仿真架构。

> [!NOTE]
>
> 基于 QEMU 的仿真构建通常比原生构建慢得多，尤其是在编译、压缩/解压缩等计算密集型任务上。
>
> 如有可能，优先考虑使用[多个原生节点](#multiple-native-nodes)或[交叉编译](#cross-compilation)。

Docker Desktop 默认支持在仿真下运行与构建多平台镜像。无需额外配置，构建器会使用 Docker Desktop 虚拟机中内置的 QEMU。

#### 手动安装 QEMU {#install-qemu-manually}

如果你在 Docker Desktop 之外使用构建器（例如 Linux 上的 Docker Engine，或自定义的远程构建器），需要在宿主机上安装 QEMU 并注册可执行类型。安装 QEMU 的前提条件包括：

- Linux 内核版本 4.8 或更高
- `binfmt-support` 版本 2.1.7 或更高
- QEMU 二进制需以静态方式编译，并在注册时带上 `fix_binary` 标志

可以使用 [`tonistiigi/binfmt`](https://github.com/tonistiigi/binfmt) 镜像，通过一条命令在宿主机上安装 QEMU 并注册可执行类型：

```console
$ docker run --privileged --rm tonistiigi/binfmt --install all
```

该命令会安装 QEMU 二进制并通过 [`binfmt_misc`](https://en.wikipedia.org/wiki/Binfmt_misc) 完成注册，使 QEMU 能够以仿真方式执行非原生的二进制格式。

在宿主机安装并注册可执行类型后，QEMU 会在容器内透明生效。你可以检查 `/proc/sys/fs/binfmt_misc/qemu-*` 中的标志是否包含 `F` 来验证注册是否成功。

### 多个原生节点 {#multiple-native-nodes}

使用多个原生节点可以更好地支持 QEMU 无法处理的复杂用例，并显著提升性能。

你可以使用 `--append` 标志向构建器追加节点。

下面的示例基于名为 `node-amd64` 与 `node-arm64` 的 Docker 上下文创建一个多节点构建器。本示例假设你已经预先添加了这些上下文。

```console
$ docker buildx create --use --name mybuild node-amd64
mybuild
$ docker buildx create --append --name mybuild node-arm64
$ docker buildx build --platform linux/amd64,linux/arm64 .
```

尽管这种方式相较仿真有明显优势，但管理多节点构建器也会带来集群搭建与维护的额外开销。你也可以使用 Docker 的托管服务——Docker Build Cloud，它在 Docker 基础设施上提供托管的多节点构建器。借助 Docker Build Cloud，你可以获得原生的多平台 ARM 与 x86 构建能力，而无需自行维护；同时还能受益于共享构建缓存等能力。

注册 Docker Build Cloud 后，将云端构建器添加到本地环境即可开始构建：

```console
$ docker buildx create --driver cloud <ORG>/<BUILDER_NAME>
cloud-<ORG>-<BUILDER_NAME>
$ docker build \
  --builder cloud-<ORG>-<BUILDER_NAME> \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  --tag <IMAGE_NAME> \
  --push .
```

更多信息参见 [Docker Build Cloud](/manuals/build-cloud/_index.md)。

### 交叉编译 {#cross-compilation}

视项目而定，如果所用编程语言对交叉编译支持良好，你可以结合多阶段构建，从构建器的原生架构出发，为目标平台生成二进制。`BUILDPLATFORM` 与 `TARGETPLATFORM` 等特殊构建参数会自动注入，可在 Dockerfile 中直接使用。

在下面的示例中，`FROM` 指令通过 `--platform=$BUILDPLATFORM` 固定为构建器的原生平台，以避免触发仿真。随后在 `RUN` 指令中插入预定义的 `$BUILDPLATFORM` 与 `$TARGETPLATFORM` 构建参数。这里仅用 `echo` 打印到标准输出，实际项目中你会将这些参数传递给编译器以完成交叉编译。

```dockerfile
# syntax=docker/dockerfile:1
FROM --platform=$BUILDPLATFORM golang:alpine AS build
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM" > /log
FROM alpine
COPY --from=build /log /log
```

## 示例

以下示例展示了多平台构建的常见用法：

- [使用仿真的简单多平台构建](#simple-multi-platform-build-using-emulation)
- [使用 Docker Build Cloud 的多平台 Neovim 构建](#multi-platform-neovim-build-using-docker-build-cloud)
- [交叉编译一个 Go 应用](#cross-compiling-a-go-application)

### 使用仿真的简单多平台构建 {#simple-multi-platform-build-using-emulation}

本示例演示如何使用 QEMU 仿真构建一个简单的多平台镜像。该镜像包含一个用于打印容器架构的文件。

先决条件：

- Docker Desktop，或安装了 [QEMU](#install-qemu-manually) 的 Docker Engine
- 已启用 containerd 镜像存储

步骤：

1. Create an empty directory and navigate to it:

   ```console
   $ mkdir multi-platform
   $ cd multi-platform
   ```

2. 创建一个简单的 Dockerfile，用于打印容器的架构：

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM alpine
   RUN uname -m > /arch
   ```

3. 为 `linux/amd64` 与 `linux/arm64` 构建镜像：

   ```console
   $ docker build --platform linux/amd64,linux/arm64 -t multi-platform .
   ```

4. 运行镜像并输出架构：

   ```console
   $ docker run --rm multi-platform cat /arch
   ```

   - 如果你在 x86-64 机器上运行，应看到 `x86_64`。
   - 如果你在 ARM 机器上运行，应看到 `aarch64`。

### 使用 Docker Build Cloud 的多平台 Neovim 构建 {#multi-platform-neovim-build-using-docker-build-cloud}

本示例演示如何使用 Docker Build Cloud 进行多平台构建，以编译并导出 `linux/amd64` 与 `linux/arm64` 平台的 [Neovim](https://github.com/neovim/neovim) 二进制。

Docker Build Cloud 提供托管的多节点构建器，支持原生的多平台构建，无需仿真，能够更快地完成诸如编译等 CPU 密集型任务。

先决条件：

- 你已[注册 Docker Build Cloud 并创建构建器](/manuals/build-cloud/setup.md)

步骤：

1. Create an empty directory and navigate to it:

   ```console
   $ mkdir docker-build-neovim
   $ cd docker-build-neovim
   ```

2. 创建一个用于构建 Neovim 的 Dockerfile：

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM debian:bookworm AS build
   WORKDIR /work
   RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
       --mount=type=cache,target=/var/lib/apt,sharing=locked \
       apt-get update && apt-get install -y \
       build-essential \
       cmake \
       curl \
       gettext \
       ninja-build \
       unzip
   ADD https://github.com/neovim/neovim.git#stable .
   RUN make CMAKE_BUILD_TYPE=RelWithDebInfo
   
   FROM scratch
   COPY --from=build /work/build/bin/nvim /
   ```

3. 使用 Docker Build Cloud 为 `linux/amd64` 与 `linux/arm64` 构建：

   ```console
   $ docker build \
      --builder <cloud-builder> \
      --platform linux/amd64,linux/arm64 \
      --output ./bin .
   ```

   该命令会使用云端构建器进行构建，并将产出的二进制导出到 `bin` 目录。

4. 验证两个平台的二进制均已构建完成。你应该能在 `linux/amd64` 与 `linux/arm64` 目录下看到 `nvim` 可执行文件。

   ```console
   $ tree ./bin
   ./bin
   ├── linux_amd64
   │   └── nvim
   └── linux_arm64
       └── nvim
   
   3 directories, 2 files
   ```

### 交叉编译一个 Go 应用 {#cross-compiling-a-go-application}

本示例展示如何结合多阶段构建，为多个平台交叉编译一个 Go 应用。该应用是一个简单的 HTTP 服务，监听 8080 端口，并返回容器的架构。本示例使用 Go，但同样适用于其他支持交叉编译的编程语言。

在 Docker 构建中进行交叉编译，依赖一组（由 BuildKit 预定义的）构建参数，这些参数提供构建器与目标平台的信息。你可以将这些参数传递给编译器以驱动交叉编译。

在 Go 中，可以通过设置 `GOOS` 与 `GOARCH` 环境变量指定编译的目标平台。

先决条件：

- Docker Desktop 或 Docker Engine

步骤：

1. Create an empty directory and navigate to it:

   ```console
   $ mkdir go-server
   $ cd go-server
   ```

2. 创建一个用于构建 Go 应用的基础 Dockerfile：

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM golang:alpine AS build
   WORKDIR /app
   ADD https://github.com/dvdksn/buildme.git#eb6279e0ad8a10003718656c6867539bd9426ad8 .
   RUN go build -o server .
   
   FROM alpine
   COPY --from=build /app/server /server
   ENTRYPOINT ["/server"]
   ```

   这个 Dockerfile 还不具备通过交叉编译进行多平台构建的能力。如果直接用 `docker build` 来构建并指定多个平台，构建器会尝试通过仿真来完成构建。

3. 若要启用交叉编译支持，请更新 Dockerfile 以使用预定义的 `BUILDPLATFORM` 与 `TARGETPLATFORM` 构建参数。当你在 `docker build` 中使用 `--platform` 标志时，这些参数会自动可用。

   - 使用 `--platform=$BUILDPLATFORM` 将 `golang` 基础镜像固定为构建器的原生平台。
   - 在 Go 编译阶段增加 `ARG` 指令，使 `TARGETOS` 与 `TARGETARCH` 构建参数在该阶段可用。
   - 将 `GOOS` 与 `GOARCH` 分别设置为 `TARGETOS` 与 `TARGETARCH` 的值。Go 编译器会使用这两个变量进行交叉编译。

   {{< tabs >}}
   {{< tab name="Updated Dockerfile" >}}

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM --platform=$BUILDPLATFORM golang:alpine AS build
   ARG TARGETOS
   ARG TARGETARCH
   WORKDIR /app
   ADD https://github.com/dvdksn/buildme.git#eb6279e0ad8a10003718656c6867539bd9426ad8 .
   RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -o server .
   
   FROM alpine
   COPY --from=build /app/server /server
   ENTRYPOINT ["/server"]
   ```

   {{< /tab >}}
   {{< tab name="Old Dockerfile" >}}

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM golang:alpine AS build
   WORKDIR /app
   ADD https://github.com/dvdksn/buildme.git#eb6279e0ad8a10003718656c6867539bd9426ad8 .
   RUN go build -o server .
   
   FROM alpine
   COPY --from=build /app/server /server
   ENTRYPOINT ["/server"]
   ```

   {{< /tab >}}
   {{< tab name="Diff" >}}

   ```diff
   # syntax=docker/dockerfile:1
   -FROM golang:alpine AS build
   +FROM --platform=$BUILDPLATFORM golang:alpine AS build
   +ARG TARGETOS
   +ARG TARGETARCH
   WORKDIR /app
   ADD https://github.com/dvdksn/buildme.git#eb6279e0ad8a10003718656c6867539bd9426ad8 .
   -RUN go build -o server .
   +RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -o server .
   
   FROM alpine
   COPY --from=build /app/server /server
   ENTRYPOINT ["/server"]
   ```

   {{< /tab >}}
   {{< /tabs >}}

4. 为 `linux/amd64` 与 `linux/arm64` 构建镜像：

   ```console
   $ docker build --platform linux/amd64,linux/arm64 -t go-server .
   ```

以上示例演示了如何在 Docker 构建中为多个平台交叉编译 Go 应用。实际的交叉编译步骤因编程语言而异，请参考相应语言的文档以了解在不同平台间交叉编译的细节。

> [!TIP]
> 你也可以了解一下
> [xx - Dockerfile 交叉编译助手](https://github.com/tonistiigi/xx)。
> `xx` 是一个包含实用脚本的 Docker 镜像，可简化在 Docker 构建中的交叉编译流程。
