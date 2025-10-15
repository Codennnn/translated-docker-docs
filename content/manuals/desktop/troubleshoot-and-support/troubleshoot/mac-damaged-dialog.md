---
description: 修复 macOS 上的 "Docker.app is damaged and can't be opened. You should move it to the Trash" 对话框
keywords: docker desktop mac, damaged app, move to trash, gatekeeper, installation issues, troubleshooting
title: 修复 macOS 上的 "Docker.app is damaged and can't be opened" 问题
linkTitle: MacOS 应用损坏对话框
tags: [Troubleshooting]
weight: 30
---

## 错误消息

当您尝试打开 Docker Desktop 时，macOS 会显示以下对话框：

```text
Docker.app is damaged and can't be opened. You should move it to the Trash.
```

此错误会阻止 Docker Desktop 启动，可能在安装期间或更新后出现。

## 可能原因

此问题是由于拖放安装过程中的非原子性复制操作导致的。当您从 DMG 文件拖放 `Docker.app` 时，如果另一个应用程序（如 VS Code）正在通过符号链接调用 Docker CLI，复制操作可能会被中断，导致应用程序处于部分复制状态，Gatekeeper 会将其标记为"已损坏"。

## 解决方案

按照以下步骤解决此问题：

### 步骤一：退出第三方软件

关闭任何可能在后台调用 Docker 的应用程序：
- Visual Studio Code 和其他 IDE
- 终端应用程序
- 代理应用或开发工具
- 任何使用 Docker CLI 的脚本或进程

### 步骤二：删除任何部分安装

1. 将 `/Applications/Docker.app` 移至废纸篓并清空废纸篓。
2. 如果您使用了 DMG 安装程序，请弹出并重新挂载 Docker DMG。

### 步骤三：重新安装 Docker Desktop

按照 [macOS 安装指南](/manuals/desktop/setup/install/mac-install.md)中的说明重新安装 Docker Desktop。

### 如果对话框仍然出现

如果按照恢复步骤操作后仍然看到"已损坏"对话框：

1. 使用终端收集诊断信息。按照[从终端诊断](/manuals/desktop/troubleshoot-and-support/troubleshoot/_index.md#diagnose-from-the-terminal)中的说明操作。
   - 记下运行诊断后终端中显示的诊断 ID。

2. 获取帮助：
   - 如果您有付费的 Docker 订阅，请[联系支持](/manuals/desktop/troubleshoot-and-support/support.md)并提供您的诊断 ID
   - 对于社区用户，请在 [GitHub 上创建 issue](https://github.com/docker/for-mac/issues) 并提供您的诊断 ID

## 预防措施

要在将来避免此问题：

- 如果您的组织允许，请通过应用内更新流程更新 Docker Desktop
- 在通过 DMG 安装程序拖放方式安装 Docker Desktop 之前，务必退出使用 Docker 的应用程序
- 在受管理的环境中，优先使用 PKG 安装方式而不是 DMG 拖放方式
- 在安装完成之前保持安装程序卷处于挂载状态

## 相关信息

- [在 Mac 上安装 Docker Desktop](/manuals/desktop/setup/install/mac-install.md)
- [PKG 安装程序文档](/manuals/enterprise/enterprise-deployment/pkg-install-and-configure.md)
- [Docker Desktop 故障排查](/manuals/desktop/troubleshoot-and-support/troubleshoot/_index.md)
- [已知问题](/manuals/desktop/troubleshoot-and-support/troubleshoot/known-issues.md)
