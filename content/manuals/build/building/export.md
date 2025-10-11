---
title: 导出可执行文件
weight: 50
description: 使用 Docker 构建来生成并导出可执行文件
keywords: build, buildkit, buildx, guide, tutorial, build arguments, arg
aliases:
  - /build/guide/export/
---

你可以使用 Docker 将应用构建为独立的可执行文件。并非所有场景都需要将应用打包并以镜像的形式分发。你可以用 Docker 完成构建，再通过导出器（exporter）将构建产物保存到磁盘。

`docker build` 的默认输出是容器镜像。该镜像会自动加载到本地镜像存储，你可以基于它运行容器，或将其推送到仓库。其底层使用的是默认导出器，即 `docker` 导出器。

如果希望将构建结果导出为文件，可使用 `--output`（或简写 `-o`）参数。该参数允许你改变构建的输出格式。

## 从构建中导出可执行文件 {#export-binaries-from-a-build}

当你为 `docker build --output` 指定路径时，Docker 会在构建结束时将构建容器的内容导出到宿主机文件系统的该路径。这使用的是 `local`
[导出器](/manuals/build/exporters/local-tar.md)。

好处在于你可以利用 Docker 强大的隔离与构建能力来生成独立可执行文件。这对 Go、Rust 等可编译为单个二进制文件的语言尤其适用。

下面的示例会创建一个打印 “Hello, World!” 的简易 Rust 程序，并将其二进制导出到宿主机文件系统。

1. 为本示例创建一个新目录，并进入该目录：

   ```console
   $ mkdir hello-world-bin
   $ cd hello-world-bin
   ```

2. 创建一个包含以下内容的 Dockerfile：

   ```Dockerfile
   # syntax=docker/dockerfile:1
   FROM rust:alpine AS build
   WORKDIR /src
   COPY <<EOT hello.rs
   fn main() {
       println!("Hello World!");
   }
   EOT
   RUN rustc -o /bin/hello hello.rs
   
   FROM scratch
   COPY --from=build /bin/hello /
   ENTRYPOINT ["/hello"]
   ```

   > [!TIP]
   > `COPY <<EOT` 语法是一种[文档内嵌（here-document）](/reference/dockerfile.md#here-documents) 写法，
   > 允许你在 Dockerfile 中书写多行字符串。这里用于直接在 Dockerfile 内内联一个简单的 Rust 程序。

   该 Dockerfile 使用多阶段构建：第一阶段完成编译，第二阶段将二进制复制到 `scratch` 镜像中。
   最终镜像是一个仅包含该二进制的最小化镜像。对于无需完整操作系统即可运行的程序，`scratch` 常被用于生成尽可能小的构建产物。

3. 构建该 Dockerfile，并将二进制导出到当前工作目录：

   ```console
   $ docker build --output=. .
   ```

   该命令会构建 Dockerfile，并将二进制导出到当前工作目录。生成的二进制名为 `hello`，位于当前目录。

## 导出多平台构建

配合[多平台构建](/manuals/build/building/multi-platform.md) 使用 `local` 导出器，可以一次性编译多个架构的二进制，
前提是你的编译器支持目标平台，这些二进制即可在对应架构的机器上运行。

延续上文[从构建中导出可执行文件](#export-binaries-from-a-build)的小节示例 Dockerfile：

```dockerfile
# syntax=docker/dockerfile:1
FROM rust:alpine AS build
WORKDIR /src
COPY <<EOT hello.rs
fn main() {
    println!("Hello World!");
}
EOT
RUN rustc -o /bin/hello hello.rs

FROM scratch
COPY --from=build /bin/hello /
ENTRYPOINT ["/hello"]
```

你可以在 `docker build` 命令中使用 `--platform` 为多个平台构建该 Rust 程序。结合 `--output` 参数，
构建会将每个目标平台的二进制导出到指定目录。

例如，同时为 `linux/amd64` 与 `linux/arm64` 构建：

```console
$ docker build --platform=linux/amd64,linux/arm64 --output=out .
$ tree out/
out/
├── linux_amd64
│   └── hello
└── linux_arm64
    └── hello

3 directories, 2 files
```

## 进一步了解

除了 `local` 导出器外，还有其他可选的导出器。了解可用导出器及其用法，参见
[导出器](/manuals/build/exporters/_index.md) 文档。
