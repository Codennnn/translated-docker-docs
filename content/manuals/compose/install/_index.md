---
description: 了解如何安装 Docker Compose。你可以通过 Docker Desktop 原生集成、Docker Engine 插件，或独立工具等方式获取 Compose。
keywords: install docker compose, docker compose plugin, install compose linux, install docker desktop, docker compose windows, standalone docker compose, docker compose not found
title: 安装 Docker Compose 概览
linkTitle: 安装
weight: 20
toc_max: 3
aliases:
- /compose/compose-desktop/
- /compose/install/other/
- /compose/install/compose-desktop/
---

本页概述在不同平台与场景下安装 Docker Compose 的可选方式。

## 安装场景 

### Docker Desktop（推荐）

获取 Docker Compose 最简单、也最推荐的方式是安装 Docker Desktop。

Docker Desktop 自带 Docker Compose，并同时包含 Compose 所需的 Docker Engine 与 Docker CLI。

Docker Desktop 适用于：
- [Linux](/manuals/desktop/setup/install/linux/_index.md)
- [Mac](/manuals/desktop/setup/install/mac-install.md)
- [Windows](/manuals/desktop/setup/install/windows-install.md)

> [!TIP]
>
> 已安装 Docker Desktop？可在 Docker 菜单中选择 **About Docker Desktop** {{< inline-image src="../../desktop/images/whale-x.svg" alt="whale menu" >}} 查看当前的 Compose 版本。

### 插件（仅限 Linux）

> [!IMPORTANT]
>
> 此安装方式仅适用于 Linux。

如果你的环境已安装 Docker Engine 与 Docker CLI，可以通过命令行安装 Docker Compose 插件，方式包括：
- [使用 Docker 官方软件源](linux.md#install-using-the-repository)
- [手动下载并安装](linux.md#install-the-plugin-manually)

### 独立版（旧方案）

> [!WARNING]
>
> 不建议使用此方式，仅为向后兼容保留支持。

你可以在 Linux 或 Windows Server 上[安装独立版 Docker Compose](standalone.md)。

