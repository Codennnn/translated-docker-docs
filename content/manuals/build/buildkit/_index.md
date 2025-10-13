---
title: BuildKit
weight: 100
description: BuildKit 介绍与概览
keywords: build, buildkit
---

## 概览

[BuildKit](https://github.com/moby/buildkit)
是对传统构建器（legacy builder）的升级后端。自 Docker Desktop 与 Docker Engine 23.0 起，BuildKit 已成为默认构建器。

BuildKit 提供了新的功能，并显著提升构建性能，同时也引入了对更复杂场景的支持：

- 发现并跳过未被使用的构建阶段
- 并行构建彼此独立的构建阶段
- 在多次构建之间，仅增量传输
  [构建上下文](../concepts/context.md) 中的变更文件
- 发现并跳过传输
  [构建上下文](../concepts/context.md) 中未使用的文件
- 使用具备众多新特性的
  [Dockerfile 前端](frontend.md)
- 避免对其余 API（中间镜像与容器）产生副作用
- 为自动清理（prune）设置构建缓存的优先级

除了许多新特性外，BuildKit 主要在性能、存储管理与可扩展性方面改进了当前体验。
在性能层面，一个重要更新是全并发的构建图求解器。它会在可能时并行执行构建步骤，
并优化掉对最终结果无影响的命令。
对本地源文件的访问也经过了优化：通过仅跟踪多次构建之间对这些文件的更新，
无需在工作开始前等待读取或上传本地文件。

## LLB

BuildKit 的核心是
[Low-Level Build（LLB）](https://github.com/moby/buildkit#exploring-llb) 定义格式。LLB 是一种中间二进制格式，
允许开发者扩展 BuildKit。LLB 定义了可寻址内容（content-addressable）的依赖图，
可用于拼装复杂的构建定义。它还支持 Dockerfile 未暴露的能力，如直接数据挂载与嵌套调用。

{{< figure src="../images/buildkit-dag.svg" class="invertible" >}}

关于构建执行与缓存的一切都由 LLB 定义。相较于传统构建器，缓存模型被彻底重写。
LLB 不再依赖启发式比较镜像，而是直接追踪构建图与挂载到特定操作的内容的校验和。
这使其更快、更精确、且可移植。构建缓存甚至可以导出到仓库，后续在任意主机上按需拉取使用。

可以直接使用
[Go 语言客户端包](https://pkg.go.dev/github.com/moby/buildkit/client/llb)
生成 LLB，用 Go 语言原语定义构建操作之间的关系。
这种方式拥有最大的灵活性，但大多数用户并不会如此定义构建；
通常会选择使用前端组件，或通过 LLB 的嵌套调用来运行一组预定义的构建步骤。

## 前端（Frontend）

前端是将人类可读的构建格式转换为 LLB 的组件，以便 BuildKit 执行。
前端可以以镜像的形式分发，用户可以选择某个特定版本的前端，以保证与其定义中使用的特性相匹配。

例如，要用 BuildKit 构建一个 [Dockerfile](/reference/dockerfile.md)，你需要
[使用外部 Dockerfile 前端](frontend.md)。

## 快速开始

BuildKit 是 Docker Desktop 与 Docker Engine v23.0 及以上版本的默认构建器。

如果你已安装 Docker Desktop，无需额外启用 BuildKit。
若你使用的是低于 23.0 的 Docker Engine 版本，可以通过环境变量或在守护进程配置中将 BuildKit 设为默认。

在运行 `docker build` 命令时通过环境变量启用 BuildKit：

```console
$ DOCKER_BUILDKIT=1 docker build .
```

> [!NOTE]
>
> Buildx 始终使用 BuildKit。

要默认启用 Docker BuildKit，编辑 `/etc/docker/daemon.json` 并重启守护进程：

```json
{
  "features": {
    "buildkit": true
  }
}
```

如果不存在 `/etc/docker/daemon.json` 文件，请新建名为 `daemon.json` 的文件，并添加上述内容。然后重启 Docker 守护进程。

## Windows 上的 BuildKit

> [!WARNING]
>
> BuildKit 仅对 Linux 容器提供完整支持。对 Windows 容器的支持仍处于实验阶段。

自 0.13 版本起，BuildKit 对 Windows 容器（WCOW）提供实验性支持。
本节将带你完成一次试用流程。若要反馈，
请[在仓库中提交 issue](https://github.com/moby/buildkit/issues/new)，尤其是与 `buildkitd.exe` 相关的问题。

### 已知限制

关于 Windows 上 BuildKit 的未解决问题与限制，请参阅
[GitHub issues](https://github.com/moby/buildkit/issues?q=is%3Aissue%20state%3Aopen%20label%3Aarea%2Fwindows-wcow)。

### 先决条件

- 架构：`amd64`、`arm64`（二进制可用，但尚未官方验证）。
- 支持的操作系统：Windows Server 2019、Windows Server 2022、Windows 11。
- 基础镜像：`ServerCore:ltsc2019`、`ServerCore:ltsc2022`、`NanoServer:ltsc2022`。
  兼容性矩阵见[此处](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-2019%2Cwindows-11#windows-server-host-os-compatibility)。
- Docker Desktop 版本 4.29 或更高。

### 步骤

> [!NOTE]
>
> 以下命令需要在具备管理员（提升）权限的 PowerShell 终端中执行。

1. 启用 Windows 功能 **Hyper-V** 与 **Containers**。

   ```console
   > Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V, Containers -All
   ```

   如果你看到 `RestartNeeded` 为 `True`，请重启计算机并以管理员身份重新打开 PowerShell 终端；
   否则继续下一步。

2. 在 Docker Desktop 中切换到 Windows 容器模式。

   点击任务栏中的 Docker 图标，选择 **Switch to Windows containers...**。

3. 按照[这里的安装说明](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#installing-containerd-on-windows)安装 containerd 1.7.7 或更高版本。

4. 下载并解压最新的 BuildKit 发行包。

   ```powershell
   $version = "v0.22.0" # 指定发行版本，v0.13+
   $arch = "amd64" # 也有 arm64 二进制
   curl.exe -LO https://github.com/moby/buildkit/releases/download/$version/buildkit-$version.windows-$arch.tar.gz
   # 可能已经存在来自 containerd 指南的另一个 `\.\bin` 目录
   # 你可以将其移动走
   mv bin bin2
   tar.exe xvf .\buildkit-$version.windows-$arch.tar.gz
   ## x bin/
   ## x bin/buildctl.exe
   ## x bin/buildkitd.exe
   ```

5. 将 BuildKit 二进制加入 `PATH`。

   ```powershell
   # 在 bin 目录解压完成后
   # 将其移动到 $Env:PATH 中合适的位置，或：
   Copy-Item -Path ".\bin" -Destination "$Env:ProgramFiles\buildkit" -Recurse -Force
   # 将 `buildkitd.exe` 与 `buildctl.exe` 加入 $Env:PATH
   $Path = [Environment]::GetEnvironmentVariable("PATH", "Machine") + `
       [IO.Path]::PathSeparator + "$Env:ProgramFiles\buildkit"
   [Environment]::SetEnvironmentVariable( "Path", $Path, "Machine")
   $Env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + `
       [System.Environment]::GetEnvironmentVariable("Path","User")
   ```
6. 启动 BuildKit 守护进程。

   ```console
   > buildkitd.exe
   ```
   > [!NOTE]
   > 如果你运行的是由 dockerd 管理的 `containerd` 进程，请改为指定其地址：
   > `buildkitd.exe --containerd-worker-addr "npipe:////./pipe/docker-containerd"`

7. 在另一个具备管理员权限的终端中，创建一个使用本地 BuildKit 守护进程的远程 builder。

   > [!NOTE]
   >
   > 需要 Docker Desktop 版本 4.29 或更高。

   ```console
   > docker buildx create --name buildkit-exp --use --driver=remote npipe:////./pipe/buildkitd
   buildkit-exp
   ```

8. 运行 `docker buildx inspect` 验证与 builder 的连接。

   ```console
   > docker buildx inspect
   ```

   输出应表明该 builder 的平台是 Windows，且其端点为命名管道（named pipe）：

   ```text
   Name:          buildkit-exp
    Driver:        remote
    Last Activity: 2024-04-15 17:51:58 +0000 UTC
    Nodes:
    Name:             buildkit-exp0
    Endpoint:         npipe:////./pipe/buildkitd
    Status:           running
    BuildKit version: v0.13.1
    Platforms:        windows/amd64
   ...
   ```

9. 创建一个 Dockerfile 并构建 `hello-buildkit` 镜像。

   ```console
   > mkdir sample_dockerfile
   > cd sample_dockerfile
   > Set-Content Dockerfile @"
   FROM mcr.microsoft.com/windows/nanoserver:ltsc2022
   USER ContainerAdministrator
   COPY hello.txt C:/
   RUN echo "Goodbye!" >> hello.txt
   CMD ["cmd", "/C", "type C:\\hello.txt"]
   "@
   Set-Content hello.txt @"
   Hello from BuildKit!
   This message shows that your installation appears to be working correctly.
   "@
   ```

10. 构建并将镜像推送到仓库。

    ```console
    > docker buildx build --push -t <username>/hello-buildkit .
    ```

11. 推送到仓库后，使用 `docker run` 运行该镜像。

    ```console
    > docker run <username>/hello-buildkit
    ```
