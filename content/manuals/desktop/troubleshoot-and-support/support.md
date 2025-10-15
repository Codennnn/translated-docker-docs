---
description: 了解 Docker Desktop 提供哪些支持
keywords: Support, Docker Desktop, Linux, Mac, Windows
title: 获取 Docker Desktop 支持
weight: 20
aliases:
 - /desktop/support/
 - /support/
---

> [!NOTE]
> 
> Docker Desktop 为拥有 [Pro、Team 或 Business 订阅](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade_desktop_support)的开发者提供支持。

### 如何获取 Docker Desktop 支持？

> [!TIP]
>
> 在寻求支持之前，请按照故障排查文档中的相应[诊断步骤](/manuals/desktop/troubleshoot-and-support/troubleshoot/_index.md#diagnose)进行操作。

如果您有付费的 Docker 订阅，可以[联系支持团队](https://hub.docker.com/support/contact/)。

所有 Docker 用户都可以通过以下资源寻求支持，Docker 或社区会尽力响应。
- [Docker Desktop for Windows GitHub 仓库](https://github.com/docker/for-win) 
- [Docker Desktop for Mac GitHub 仓库](https://github.com/docker/for-mac)
- [Docker Desktop for Linux GitHub 仓库](https://github.com/docker/desktop-linux)
- [Docker 社区论坛](https://forums.docker.com/)
- [Docker 社区 Slack](http://dockr.ly/comm-slack)


### 我可以获得什么支持？

- 账户管理相关问题
- 自动构建
- 基本的产品"使用方法"问题
- 账单或订阅问题
- 配置问题
- Desktop 安装问题
   - 安装崩溃
   - 首次运行时 Docker Desktop 无法启动
- Desktop 更新问题
- 命令行界面和 Docker Hub 用户界面中的登录问题
- 推送或拉取问题，包括速率限制
- 使用问题
   - 关闭软件时崩溃
   - Docker Desktop 行为不符合预期

对于 Windows 用户，您还可以请求以下支持：
- 在 BIOS 中启用虚拟化
- 启用 Windows 功能
- 在[特定 VM 或 VDI 环境](/manuals/desktop/setup/vm-vdi.md)中运行（仅限 Docker Business 客户）

### 不支持哪些内容？

Docker Desktop 不支持以下类型的问题：

- 在适用文档中指定的硬件或软件之外或与其结合使用
- 在不受支持的操作系统上运行，包括操作系统的 beta/预览版本
- 使用模拟运行不同架构的容器
- 对 Docker Engine、Docker CLI 或其他捆绑的 Linux 组件的支持
- 对 Kubernetes 的支持
- 标记为实验性的功能
- 系统/服务器管理活动
- 将 Desktop 作为生产运行时支持
- Desktop 的规模部署/多机安装
- 日常产品维护（数据备份、清理磁盘空间和配置日志轮换）
- Docker 未提供的第三方应用程序
- 更改或修改过的 Docker 软件
- 由于硬件故障、滥用或使用不当导致的 Docker 软件缺陷
- 除最新版本之外的任何版本的 Docker 软件
- 报销和用于 Docker 未提供的第三方服务的费用
- Docker 支持不包括培训、定制和集成
- 在单台机器上运行多个 Docker Desktop 实例

> [!NOTE]
>
> 对[在 VM 或 VDI 环境中运行 Docker Desktop](/manuals/desktop/setup/vm-vdi.md) 的支持仅适用于 Docker Business 客户。

### 支持哪些版本？

对于 Docker Business 客户，Docker 提供对比最新版本早最多六个月的版本的支持，尽管任何修复都将在最新版本上进行。

对于 Pro 和 Team 客户，Docker 仅为最新版本的 Docker Desktop 提供支持。如果您运行的是旧版本，Docker 可能会要求您在调查支持请求之前进行更新。

### 我可以在多少台机器上获得 Docker Desktop 支持？

作为 Pro 用户，您可以在单台机器上获得 Docker Desktop 支持。
作为 Team 用户，您可以为与订阅中席位数量相等的机器数量获得 Docker Desktop 支持。

### 支持哪些操作系统？

Docker Desktop 适用于 Mac、Linux 和 Windows。支持的版本信息可在以下页面找到：

- [Mac 系统要求](/manuals/desktop/setup/install/mac-install.md#system-requirements)
- [Windows 系统要求](/manuals/desktop/setup/install/windows-install.md#system-requirements)
- [Linux 系统要求](/manuals/desktop/setup/install/linux/_index.md#system-requirements)

### 获取支持时，Docker Desktop 如何处理个人诊断数据？

上传诊断信息以帮助 Docker 调查问题时，上传的诊断包可能包含用户名和 IP 地址等个人数据。诊断包仅供直接参与诊断 Docker Desktop 问题的 Docker, Inc. 员工访问。

默认情况下，Docker, Inc. 将在 30 天后删除上传的诊断包。您也可以通过指定诊断 ID 或通过您的 GitHub ID（如果诊断 ID 在 GitHub issue 中提到）来请求删除诊断包。Docker, Inc. 只会使用诊断包中的数据来调查特定的用户问题，但可能会从中得出高级别（非个人）指标，例如问题发生率。

有关更多信息，请参阅 [Docker 数据处理协议](https://www.docker.com/legal/data-processing-agreement)。
