---
title: 可选容器运行时
description: |
  Docker Engine 默认使用 runc 作为容器运行时，你也可以通过 CLI 或配置守护进程
  来指定其他运行时
keywords: engine, runtime, containerd, runtime v2, shim
aliases:
  - /engine/alternative-runtimes/
---

Docker Engine 使用 containerd 管理容器生命周期，
包括创建、启动与停止容器。默认情况下，containerd 使用 runc 作为容器运行时。

## 我可以使用哪些运行时？

任何实现了 containerd
[shim API](https://github.com/containerd/containerd/blob/main/core/runtime/v2/README.md)
的运行时都可与 Docker 协同使用。
这类运行时自带 containerd shim，通常无需额外配置即可使用。
参见[使用 containerd shim](#use-containerd-shims)。

以下运行时实现了各自的 containerd shim：

- [Wasmtime](https://wasmtime.dev/)
- [gVisor](https://github.com/google/gvisor)
- [Kata Containers](https://katacontainers.io/)

你也可以使用作为 runc 即插即用替代品（drop-in replacement）设计的运行时。
这类运行时依赖 runc 的 containerd shim 来调用运行时二进制。
需要在守护进程配置中手动注册此类运行时。

[youki](https://github.com/youki-dev/youki)
即为可作为 runc 替代品的一个示例。设置方法见[youki 示例](#youki)。

## 使用 containerd shim

containerd shim 允许你在无需修改 Docker 守护进程配置的情况下使用其他运行时。
要使用某个 shim，请将其可执行文件安装到运行 Docker 守护进程的系统 `PATH` 中。

在 `docker run` 中使用 shim 时，通过 `--runtime` 标志传入运行时的完整名称：

```console
$ docker run --runtime io.containerd.kata.v2 hello-world
```

### 在未安装到 PATH 的情况下使用 containerd shim

也可以不将 shim 安装到 `PATH`，此时需要在守护进程配置中注册该 shim，如下所示：

```json
{
  "runtimes": {
    "foo": {
      "runtimeType": "/path/to/containerd-shim-foobar-v1"
    }
  }
}
```

使用时，传入你为该 shim 指定的名称：

```console
$ docker run --runtime foo hello-world
```

### 配置 shim

如需为 containerd shim 传递额外配置，可在守护进程配置文件中使用 `runtimes` 选项。

1. 编辑守护进程配置文件，为目标 shim 添加一个 `runtimes` 条目。

   - 在 `runtimeType` 中指定运行时的完整名称
   - 在 `options` 中添加运行时的相关配置

   ```json
   {
     "runtimes": {
       "gvisor": {
         "runtimeType": "io.containerd.runsc.v1",
         "options": {
           "TypeUrl": "io.containerd.runsc.v1.options",
           "ConfigPath": "/etc/containerd/runsc.toml"
         }
       }
     }
   }
   ```

2. 重新加载守护进程配置。

   ```console
   # systemctl reload docker
   ```

3. 在 `docker run` 中通过 `--runtime` 标志使用该自定义运行时。

   ```console
   $ docker run --runtime gvisor hello-world
   ```

有关 containerd shim 配置项的更多信息，参见
[配置 containerd shim](/reference/cli/dockerd.md#configure-containerd-shims)。

## 示例

以下示例展示如何在 Docker Engine 中配置并使用其他容器运行时。

- [youki](#youki)
- [Wasmtime](#wasmtime)

### youki

youki 是用 Rust 编写的容器运行时。
相较 runc，youki 号称速度更快、内存占用更低，
适用于资源受限的环境。

youki 可作为 runc 的即插即用替代品，这意味着它依赖 runc shim 来调用运行时二进制。
当注册作为 runc 替代品的运行时时，需要配置运行时可执行文件的路径，
以及可选的运行时参数。详见
[配置 runc 的替代运行时](/reference/cli/dockerd.md#configure-runc-drop-in-replacements)。

将 youki 添加为容器运行时：

1. 安装 youki 及其依赖。

   安装步骤参见
   [官方指南](https://youki-dev.github.io/youki/user/basic_setup.html)。

2. 通过编辑 Docker 守护进程配置文件（默认位于 `/etc/docker/daemon.json`），
   将 youki 注册为 Docker 运行时。

   `path` 键应指向 youki 的安装路径。

   ```console
   # cat > /etc/docker/daemon.json <<EOF
   {
     "runtimes": {
       "youki": {
         "path": "/usr/local/bin/youki"
       }
     }
   }
   EOF
   ```

3. 重新加载守护进程配置。

   ```console
   # systemctl reload docker
   ```

现在即可使用 youki 作为运行时来运行容器。

```console
$ docker run --rm --runtime youki hello-world
```

### Wasmtime

{{< summary-bar feature_name="Wasmtime" >}}

Wasmtime 是
[Bytecode Alliance](https://bytecodealliance.org/)
项目下的 Wasm 运行时，可运行 Wasm 容器。
在 Docker 中运行 Wasm 容器可获得两层安全防护：
一方面具备容器隔离带来的益处，另一方面享有 Wasm 运行时环境提供的额外沙箱。

将 Wasmtime 添加为容器运行时的步骤：

1. 在守护进程配置文件中开启
   [containerd 镜像存储](/manuals/engine/storage/containerd.md)。

   ```json
   {
     "features": {
       "containerd-snapshotter": true
     }
   }
   ```

2. 重启 Docker 守护进程。

   ```console
   # systemctl restart docker
   ```

3. 安装 Wasmtime 的 containerd shim 到 `PATH`。

   下面的命令会通过一个内联 Dockerfile 从源码构建 Wasmtime 二进制，
   并导出为 `./containerd-shim-wasmtime-v1`：

   ```console
   $ docker build --output . - <<EOF
   FROM rust:latest as build
   RUN cargo install \
       --git https://github.com/containerd/runwasi.git \
       --bin containerd-shim-wasmtime-v1 \
       --root /out \
       containerd-shim-wasmtime
   FROM scratch
   COPY --from=build /out/bin /
   EOF
   ```

   将该二进制放置到 `PATH` 中的目录：

   ```console
   $ mv ./containerd-shim-wasmtime-v1 /usr/local/bin
   ```

现在即可使用 Wasmtime 作为运行时来运行容器。

```console
$ docker run --rm \
 --runtime io.containerd.wasmtime.v1 \
 --platform wasi/wasm32 \
 michaelirwin244/wasm-example
```

## 相关信息

- 关于容器运行时的更多配置项，参见
  [配置容器运行时](/reference/cli/dockerd.md#configure-container-runtimes)。
- 你可以配置守护进程的默认运行时，参见
  [配置默认容器运行时](/reference/cli/dockerd.md#configure-the-default-container-runtime)。
