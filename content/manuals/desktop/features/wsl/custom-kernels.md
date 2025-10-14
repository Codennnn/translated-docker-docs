---
title: 在 WSL 上使用自定义内核 
description: 在 WSL 2 上将 Docker Desktop 与自定义内核搭配使用
keywords: wsl, docker desktop, custom kernel
tags: [Best practices, troubleshooting]
---

Docker Desktop 依赖 Microsoft 分发的默认 WSL 2 Linux 内核中内置的多个内核特性。因此，在 WSL 2 上将 Docker Desktop 与自定义内核搭配使用不在官方支持范围内，可能会影响 Docker Desktop 的启动或运行。

不过，在某些情况下你可能需要使用自定义内核。Docker Desktop 不会阻止其使用，且已有用户报告成功案例。

如果你决定使用自定义内核，建议从 Microsoft 在其[官方代码仓库](https://github.com/microsoft/WSL2-Linux-Kernel)分发的内核代码树出发，并在此基础上增量添加所需特性。

同时，建议你：
- 使用与最新 WSL 2 发布版一致的内核版本。可在终端运行 `wsl.exe --system uname -r` 查询版本。
- 以 Microsoft 在其[仓库](https://github.com/microsoft/WSL2-Linux-Kernel)提供的默认内核配置为起点，再逐步加入所需特性。
- 确保内核构建环境包含 `pahole`，且其版本正确反映在对应的内核配置项（`CONFIG_PAHOLE_VERSION`）中。

