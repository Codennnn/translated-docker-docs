---
title: 使用 WSL 进行开发
description: 介绍如何在 WSL 2 上使用 Docker 进行开发，并了解 WSL 的 GPU 支持
keywords: wsl, wsl 2, develop, docker desktop, windows
aliases:
- /desktop/wsl/use-wsl/
---

本节介绍如何使用 Docker 与 WSL 2 开始你的应用开发。建议将代码放在默认的 Linux 发行版中，以获得更佳的 Docker + WSL 2 开发体验。启用 Docker Desktop 的 WSL 2 功能后，你可以在 Linux 发行版内处理代码，同时仍在 Windows 上使用 IDE。若使用 [VS Code](https://code.visualstudio.com/download)，该工作流更加顺畅。

## 在 Docker 与 WSL 2 上进行开发

1. 打开 VS Code 并安装 [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) 扩展。该扩展允许你在 Windows 上的 IDE 客户端与 Linux 发行版中的远程环境协同工作。
2. 打开终端并输入：

    ```console
    $ wsl
    ```
3. 进入你的项目目录，然后输入：

    ```console
    $ code .
    ```

    这将打开一个新的 VS Code 窗口，并远程连接到你的默认 Linux 发行版，你可以在 VS Code 窗口的右下角确认连接状态。


或者，你也可以从 **Start** 菜单打开默认的 Linux 发行版，进入项目目录后运行 `code .`。

