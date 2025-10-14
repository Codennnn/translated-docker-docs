---
title: Docker Desktop（Mac）的虚拟机管理器（VMM）
linkTitle: 虚拟机管理器（VMM）
keywords: virtualization software, resource allocation, mac, docker desktop, vm monitoring, vm performance, apple silicon
description: 了解 Docker Desktop for Mac 的虚拟机管理器（VMM）选项，包括面向 Apple Silicon 的全新 Docker VMM，带来更高的性能与效率
weight: 110
aliases:
- /desktop/vmm/
---

Docker Desktop 提供多种虚拟机管理器（VMM）来支撑运行容器的 Linux 虚拟机。你可以根据系统架构（Intel 或 Apple Silicon）、性能诉求与功能需求选择最合适的选项。本文概览各可用方案。

要切换 VMM，前往 **Settings** > **General** > **Virtual Machine Manager**。

## Docker VMM（Beta）

{{< summary-bar feature_name="VMM" >}}

Docker VMM 是一款全新的、面向容器优化的虚拟机管理程序（hypervisor）。通过同时优化 Linux 内核与虚拟化层，Docker VMM 在常见开发任务上带来显著的性能提升。

Docker VMM 的关键性能增强包括：
 - Faster I/O operations: With a cold cache, iterating over a large shared filesystem with `find` is 2x faster than when the Apple Virtualization framework is used.
 - Improved caching: With a warm cache, performance can improve by as much as 25x, even surpassing native Mac operations.

这些提升直接改善了依赖频繁文件访问与整体系统响应性的容器化开发体验。Docker VMM 在速度上实现了跃升，使开发流程更顺畅、迭代更高效。

> [!NOTE]
>
> Docker VMM 要求为 Docker Linux VM 至少分配 4GB 内存。在启用 Docker VMM 之前需要提升内存配额，可在 **Settings** 的 **Resources** 选项卡中完成。

### 已知问题

由于 Docker VMM 仍处于 Beta 阶段，存在以下已知限制：

- Docker VMM 目前不支持 Rosetta，因此对 amd64 架构的仿真速度较慢。Docker 正在探索潜在解决方案。
- 在 Docker VMM 下使用 virtiofs 时，某些数据库（如 MongoDB、Cassandra）可能运行失败。该问题预计会在后续版本中修复。

## Apple 虚拟化框架（Apple Virtualization framework）

Apple Virtualization framework 是在 Mac 上管理虚拟机的成熟且稳定的方案，多年来一直是众多 Mac 用户可信赖的选择。对于倾向于采用经验证、性能可靠、兼容性广的方案的开发者而言，这是一个理想选项。

## 适用于 Apple Silicon 的 QEMU（遗留）

> [!NOTE]
>
> 在 4.44 及之后的版本中，QEMU 已被弃用。详情参见[博客公告](https://www.docker.com/blog/docker-desktop-for-mac-qemu-virtualization-option-to-be-deprecated-in-90-days/) 

QEMU 是 Apple Silicon Mac 上的遗留虚拟化选项，主要用于兼容旧有用例。 

Docker 建议迁移到更新的替代方案，例如 Docker VMM 或 Apple Virtualization framework，它们提供更佳的性能与持续支持。尤其是 Docker VMM，具有显著的速度提升与更高效的开发环境，对 Apple Silicon 开发者而言是颇具吸引力的选择。

请注意，这与在[多平台构建](/manuals/build/building/multi-platform.md#qemu)中使用 QEMU 仿真非原生架构无关。

## 适用于 Intel Mac 的 HyperKit（遗留）

> [!NOTE]
>
> HyperKit 将在未来版本中被弃用。

HyperKit 是另一项遗留虚拟化选项，面向 Intel 架构的 Mac。与 QEMU 一样，它仍可用但已被视为弃用。建议迁移到更现代的方案，以获得更好的性能并提升未来可持续性。