---
description: 扩展功能
keywords: Docker Extensions, Docker Desktop, Linux, Mac, Windows,
title: 非市场分发的扩展（Non‑marketplace extensions）
weight: 20
aliases:
 - /desktop/extensions/non-marketplace/
---

## 安装未在市场上架的扩展

> [!WARNING]
>
> 未在扩展市场上架的 Docker 扩展未经过 Docker 的审核流程。
> 扩展可能会安装二进制、执行命令并访问你的本机文件，安装此类扩展请自行承担风险。

扩展市场（Extensions Marketplace）是 Docker Desktop 内安装扩展的官方可信渠道，其中的扩展均已通过 Docker 的审核。但如果你信任扩展作者，也可以在 Docker Desktop 中安装来自其他渠道的扩展。

由于 Docker 扩展本质上是一个 Docker 镜像，你可以在其他平台找到扩展的源代码或镜像来源，例如 GitHub、GitLab，或镜像仓库（如 Docker Hub、GHCR）。
你可以安装社区开发的扩展，或安装你所在公司内部同事开发的扩展；安装来源并不局限于市场。

> [!NOTE]
>
> 确保关闭 **Allow only extensions distributed through the Docker Marketplace**（仅允许通过 Docker Marketplace 分发的扩展）。否则，通过 Extension SDK 工具安装的、未在市场列出的扩展将被阻止。
> 你可以在 **Settings** 中更改该选项。

要安装未在市场上架的扩展，可以使用随 Docker Desktop 一同提供的 Extensions CLI。

在终端中执行 `docker extension install IMAGE[:TAG]`，根据镜像引用（可选附加 TAG）安装扩展。使用 `-f` 或 `--force` 标志可跳过交互式确认。

安装完成后，前往 Docker Desktop 仪表盘查看新扩展。

## 列出已安装的扩展

无论扩展是通过市场安装，还是通过 Extensions CLI 手动安装，你都可以使用 `docker extension ls` 查看已安装扩展的列表。
输出中会包含扩展 ID、提供方、版本、标题，以及是否运行后端容器或是否向主机部署了二进制等信息。例如：

```console
$ docker extension ls
ID                  PROVIDER            VERSION             UI                    VM                  HOST
john/my-extension   John                latest              1 tab(My-Extension)   Running(1)          -
```

前往 Docker Desktop 仪表盘，选择 **Add Extensions**，并在 **Managed** 页签中查看已安装的扩展。
注意会看到 `UNPUBLISHED` 标签，表示该扩展并非通过市场安装。

## 更新扩展 

要更新未在市场上架的扩展，在终端中执行 `docker extension update IMAGE[:TAG]`，其中 `TAG` 应区别于当前已安装版本的 TAG。

例如，如果你之前通过 `docker extension install john/my-extension:0.0.1` 安装了扩展，可通过 `docker extension update john/my-extension:0.0.2` 完成更新。
更新完成后，前往 Docker Desktop 仪表盘查看结果。

> [!NOTE]
>
> 非市场安装的扩展不会在 Docker Desktop 中收到更新通知。

## 卸载扩展

要卸载未在市场上架的扩展，你可以在扩展市场的 **Managed** 页签中点击 **Uninstall**，或在终端执行 `docker extension uninstall IMAGE[:TAG]`。
