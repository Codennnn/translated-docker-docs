---
title: Wasm 工作负载 
weight: 90
description: 如何在 Docker Desktop 上运行 Wasm 工作负载
keywords: Docker, WebAssembly, wasm, containerd, engine
toc_max: 3
aliases: 
- /desktop/wasm/
params:
  sidebar:
    badge:
      color: blue
      text: Beta
---

{{< summary-bar feature_name="Wasm workloads" >}}

WebAssembly（Wasm）是一种快速、轻量的运行时选择，可作为 Linux 与 Windows 容器的替代或补充。在 Docker Desktop 中，你可以让 Wasm 工作负载与传统容器并行运行。

本文介绍如何在 Docker 中与 Linux 容器并行运行 Wasm 应用。

> [!TIP]
>
> 想进一步了解 Wasm 的使用场景与取舍，请参阅 [Docker Wasm 技术预览博文](https://www.docker.com/blog/docker-wasm-technical-preview/)。

## 启用 Wasm 工作负载

运行 Wasm 工作负载需要启用 [containerd 镜像存储](containerd.md)。如果你当前未使用 containerd 镜像存储，现有镜像与容器将暂不可见。

1. 打开 Docker Desktop 的 **Settings**。
2. 在 **General** 选项卡勾选 **Use containerd for pulling and storing images**。
3. 前往 **Features in development**，勾选 **Enable Wasm**。
4. 点击 **Apply** 保存设置。
5. 在确认对话框中选择 **Install** 以安装 Wasm 运行时。

Docker Desktop 将下载并安装以下运行时： 
- `io.containerd.slight.v1`
- `io.containerd.spin.v2`
- `io.containerd.wasmedge.v1`
- `io.containerd.wasmtime.v1`
- `io.containerd.lunatic.v1`
- `io.containerd.wws.v1`
- `io.containerd.wasmer.v1`

## 使用示例

### 使用 `docker run` 运行 Wasm 应用

以下 `docker run` 命令会在你的系统上启动一个 Wasm 容器：

```console
$ docker run \
  --runtime=io.containerd.wasmedge.v1 \
  --platform=wasi/wasm \
  secondstate/rust-example-hello
```

运行后，可访问 [http://localhost:8080/](http://localhost:8080/) 查看示例模块输出的 "Hello world"。

若遇到错误信息，请参见[故障排查](#troubleshooting)。

注意该命令中使用了 `--runtime` 与 `--platform` 标志：

- `--runtime=io.containerd.wasmedge.v1`：告诉 Docker 引擎使用 Wasm 的 containerd shim，而不是标准的 Linux 容器运行时
- `--platform=wasi/wasm`：指定要使用的镜像架构。借助 Wasm 架构，你无需为不同硬件架构分别构建镜像；Wasm 运行时会完成将 Wasm 二进制转换为机器指令的最后一步。

### 使用 Docker Compose 运行 Wasm 应用

可使用以下 Docker Compose 文件运行同样的应用：

```yaml
services:
  app:
    image: secondstate/rust-example-hello
    platform: wasi/wasm
    runtime: io.containerd.wasmedge.v1
```

使用常规的 Docker Compose 命令启动应用：

   ```console
   $ docker compose up
   ```

### 运行包含 Wasm 的多服务应用

网络工作方式与 Linux 容器一致，你可以灵活地将 Wasm 应用与其他容器化工作负载（如数据库）组合到同一应用栈中。

以下示例中，Wasm 应用依赖于在容器中运行的 MariaDB 数据库。

1. Clone the repository.

   ```console
   $ git clone https://github.com/second-state/microservice-rust-mysql.git
   Cloning into 'microservice-rust-mysql'...
   remote: Enumerating objects: 75, done.
   remote: Counting objects: 100% (75/75), done.
   remote: Compressing objects: 100% (42/42), done.
   remote: Total 75 (delta 29), reused 48 (delta 14), pack-reused 0
   Receiving objects: 100% (75/75), 19.09 KiB | 1.74 MiB/s, done.
   Resolving deltas: 100% (29/29), done.
   ```

2. 进入克隆的项目目录，并使用 Docker Compose 启动项目。

   ```console
   $ cd microservice-rust-mysql
   $ docker compose up
   [+] Running 0/1
   ⠿ server Warning                                                                                                  0.4s
   [+] Building 4.8s (13/15)
   ...
   microservice-rust-mysql-db-1      | 2022-10-19 19:54:45 0 [Note] mariadbd: ready for connections.
   microservice-rust-mysql-db-1      | Version: '10.9.3-MariaDB-1:10.9.3+maria~ubu2204'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
   ```

   If you run `docker image ls` from another terminal window, you can see the
   Wasm image in your image store.

   ```console
   $ docker image ls
   REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
   server       latest    2c798ddecfa1   2 minutes ago   3MB
   ```

   查看镜像可见其平台为 `wasi/wasm`，即一组操作系统与架构的组合：

   ```console
   $ docker image inspect server | grep -A 3 "Architecture"
           "Architecture": "wasm",
           "Os": "wasi",
           "Size": 3001146,
           "VirtualSize": 3001146,
   ```

3. 在浏览器打开 `http://localhost:8090`，创建一些示例订单。上述操作均与 Wasm 服务交互。

4. 完成后，在启动应用的终端按 `Ctrl+C` 结束所有进程。

### 构建并推送 Wasm 模块

1. Create a Dockerfile that builds your Wasm application.

   具体步骤因使用的编程语言而异。

2. In a separate stage in your `Dockerfile`, extract the module and set it as
   the `ENTRYPOINT`.

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM scratch
   COPY --from=build /build/hello_world.wasm /hello_world.wasm
   ENTRYPOINT [ "/hello_world.wasm" ]
   ```

3. 指定 `wasi/wasm` 架构进行构建与推送。借助 Buildx，可通过单条命令完成：

   ```console
   $ docker buildx build --platform wasi/wasm -t username/hello-world .
   ...
   => exporting to image                                                                             0.0s
   => => exporting layers                                                                            0.0s
   => => exporting manifest sha256:2ca02b5be86607511da8dc688234a5a00ab4d58294ab9f6beaba48ab3ba8de56  0.0s
   => => exporting config sha256:a45b465c3b6760a1a9fd2eda9112bc7e3169c9722bf9e77cf8c20b37295f954b    0.0s
   => => naming to docker.io/username/hello-world:latest                                            0.0s
   => => unpacking to docker.io/username/hello-world:latest                                         0.0s
   $ docker push username/hello-world
   ```

## 故障排查

本节包含常见问题的解决方法。

### Unknown runtime specified

如果在未启用[containerd 镜像存储](./containerd.md)的情况下运行 Wasm 容器，会出现类似如下错误：

```text
docker: Error response from daemon: Unknown runtime specified io.containerd.wasmedge.v1.
```

请在 Docker Desktop 设置中[启用 containerd 功能](./containerd.md#enable-the-containerd-image-store)后重试。

### Failed to start shim: failed to resolve runtime path

如果使用的 Docker Desktop 版本较旧且不支持运行 Wasm 工作负载，会看到如下错误：

```text
docker: Error response from daemon: failed to start shim: failed to resolve runtime path: runtime "io.containerd.wasmedge.v1" binary not installed "containerd-shim-wasmedge-v1": file does not exist: unknown.
```

请将 Docker Desktop 更新到最新版本后重试。

## Known issues

- 中断时 Docker Compose 可能不会完全退出。可临时通过向 `docker-compose` 进程发送 SIGKILL（`killall -9 docker-compose`）进行清理。
- 即便通过 Docker Desktop 登录，推送到 Docker Hub 仍可能报错 `server message: insufficient_scope: authorization failed`。可在 CLI 中运行 `docker login` 作为临时解决方案。
