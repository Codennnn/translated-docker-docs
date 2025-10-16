---
title: 快速开始
description: 快速构建扩展的实用指南
keywords: quickstart, extensions
aliases:
 - desktop/extensions-sdk/tutorials/initialize/
 - /desktop/extensions-sdk/quickstart/
weight: 20
---

按照本指南创建一个基础的 Docker 扩展。快速开始会为你自动生成脚手架文件。

## 先决条件

- [Docker Desktop](/manuals/desktop/release-notes.md)
- [NodeJS](https://nodejs.org/)
- [Go](https://go.dev/dl/)

> [!NOTE]
>
> 只有在使用本快速开始创建扩展时才需要 NodeJS 与 Go。该流程会使用 `docker extension init` 命令自动生成脚手架文件，该命令基于 ReactJS 与 Go 的模板项目。

在 Docker Desktop 设置中，确保你可以安装正在开发的扩展。你可能需要进入 **Extensions** 选项卡，取消勾选 **Allow only extensions distributed through the Docker Marketplace**。

## 步骤一：初始化目录

使用 `init` 子命令并提供扩展名称来初始化目录。

```console
$ docker extension init <my-extension>
```

该命令会询问扩展的名称、描述以及 Hub 仓库名等信息，以便 CLI 生成一套脚手架文件，并将其存放在 `my-extension` 目录中。

自动生成的扩展包含：

- `backend` 目录下的 Go 后端服务，通过 socket 提供服务，包含一个返回 JSON 的 `/hello` 端点。
- `frontend` 目录下的 React 前端，可调用后端并展示其响应。

关于构建 UI 的更多建议与规范，请参阅 [设计与样式](design/design-guidelines.md)。

## 步骤二：构建扩展

切换到新创建的目录并执行：

```console
$ docker build -t <name-of-your-extension> .
```

`docker build` 会构建扩展并生成与所选 Hub 仓库名一致的镜像。例如，如果你在以下问题中输入了 `john/my-extension`：

```console
? Hub repository (eg. namespace/repository on hub): john/my-extension`
```

则会生成名为 `john/my-extension` 的镜像。

## 步骤三：安装并预览扩展

在 Docker Desktop 中安装扩展：

```console
$ docker extension install <name-of-your-extension>
```

安装完成后，你应当能在 **Extensions** 菜单下看到一个 **Quickstart** 项。点击即可打开扩展的前端界面进行预览。

> [!TIP]
>
> 在进行 UI 开发时，建议使用热重载以便在无需重建整个扩展的情况下验证变更。参见 [开发 UI 时的预览](dev/test-debug.md#hot-reloading-whilst-developing-the-ui)。

你可能还希望查看属于该扩展的容器。默认情况下，它们在 Docker Dashboard 中是隐藏的。你可以在 **Settings** 中更改此行为，详见[如何显示扩展容器](dev/test-debug.md#show-the-extension-containers)。

## 步骤四：提交并发布到扩展市场

如果希望让所有 Docker Desktop 用户都能使用你的扩展，可以提交发布到扩展市场。更多信息请参见 [发布](extensions/_index.md)。

## 清理

如需卸载扩展，执行：

```console
$ docker extension rm <name-of-your-extension>
```

## 下一步

- 为扩展构建更[高级的前端](build/frontend-extension-tutorial.md)。
- 了解如何[测试与调试](dev/test-debug.md)扩展。
- 了解如何[为扩展配置 CI](dev/continuous-integration.md)。
- 进一步了解扩展的[架构](architecture/_index.md)。
- 进一步了解[UI 设计](design/design-guidelines.md)。
