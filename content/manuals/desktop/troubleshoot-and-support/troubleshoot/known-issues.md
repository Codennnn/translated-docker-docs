---
description: 查找 Docker Desktop 的已知问题
keywords: mac, troubleshooting, known issues, Docker Desktop
title: 已知问题
tags: [ Troubleshooting ]
weight: 20
aliases:
 - /desktop/troubleshoot/known-issues/
---

{{< tabs >}}
{{< tab name="For Mac with Intel chip" >}}
- Mac 活动监视器报告的 Docker 内存使用量是实际使用量的两倍。这是由于 [macOS 的一个 bug](https://docs.google.com/document/d/17ZiQC1Tp9iH320K-uqVLyiJmk4DHJ3c4zgQetJiKYQM/edit?usp=sharing) 导致的。

- **"Docker.app is damaged" 对话框**：如果在安装或更新过程中看到 "Docker.app is damaged and can't be opened" 对话框，这通常是由于其他应用程序正在使用 Docker CLI 时执行了非原子性复制操作导致的。解决步骤请参阅[修复 macOS 上的 "Docker.app is damaged" 问题](mac-damaged-dialog.md)。

- 从 `.dmg` 运行 `Docker.app` 后强制弹出 `.dmg` 可能会导致鲸鱼图标无响应、Docker 任务在活动监视器中显示为无响应，以及某些进程消耗大量 CPU 资源。重启系统和 Docker 可以解决这些问题。

- Docker Desktop 在 macOS 10.10 Yosemite 及更高版本中使用 `HyperKit` 虚拟机管理程序（https://github.com/docker/hyperkit）。如果您在开发时使用的工具与 `HyperKit` 存在冲突，例如 [Intel Hardware Accelerated Execution Manager (HAXM)](https://software.intel.com/en-us/android/articles/intel-hardware-accelerated-execution-manager/)，当前的解决方法是不要同时运行它们。在使用 HAXM 时，您可以通过临时退出 Docker Desktop 来暂停 `HyperKit`。这样可以让您继续使用其他工具，并防止 `HyperKit` 产生干扰。

- 如果您在使用 [Apache Maven](https://maven.apache.org/) 等需要设置 `DOCKER_HOST` 和 `DOCKER_CERT_PATH` 环境变量的应用程序，请指定这些变量以通过 Unix 套接字连接到 Docker 实例。例如：

  ```console
  $ export DOCKER_HOST=unix:///var/run/docker.sock
  ```

{{< /tab >}}
{{< tab name="For Mac with Apple silicon" >}}

- 某些命令行工具在未安装 Rosetta 2 时无法工作。
  - 旧版本 1.x 的 `docker-compose`。请改用 Compose V2 - 输入 `docker compose`。
  - `docker-credential-ecr-login` 凭据助手。
- 某些镜像不支持 ARM64 架构。您可以添加 `--platform linux/amd64` 参数来使用模拟方式运行（或构建）Intel 镜像。

   但是，在 Apple Silicon 机器上通过模拟运行基于 Intel 的容器可能会崩溃，因为 QEMU 有时无法运行容器。此外，文件系统变更通知 API（`inotify`）在 QEMU 模拟下无法工作。即使容器在模拟下能够正常运行，它们也会比原生版本更慢，并使用更多内存。

   总之，在基于 Arm 的机器上运行基于 Intel 的容器应该仅被视为"尽力而为"。我们建议在 Apple Silicon 机器上尽可能运行 `arm64` 容器，并鼓励容器作者提供 `arm64` 或多架构版本的容器。随着越来越多的镜像被重新构建为[支持多架构](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/)，这个问题会随着时间的推移变得越来越少见。
- 当 TCP 流处于半关闭状态时，用户可能偶尔会遇到数据丢失的情况。

{{< /tab >}}
{{< /tabs >}}
