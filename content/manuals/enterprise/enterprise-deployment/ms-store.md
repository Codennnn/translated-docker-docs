---
title: 从 Microsoft Store 在 Windows 上安装 Docker Desktop
linkTitle: MS Store
description: 通过 Microsoft Store 安装 Windows 版 Docker Desktop。了解其更新行为和限制。
keywords: microsoft store, windows, docker desktop, 安装, 部署, 配置, 管理员, mdm, intune, winget
tags: [admin]
weight: 30
aliases: 
 - /desktop/setup/install/enterprise-deployment/ms-store/
---

您可以通过 [Microsoft 应用商店](https://apps.microsoft.com/detail/xp8cbj40xlbwkx?hl=en-GB&gl=GB) 部署 Windows 版 Docker Desktop。

Microsoft Store 版本的 Docker Desktop 提供与标准安装程序相同的功能，但根据是由开发人员自行安装还是由 Intune 等 MDM 工具处理安装，其更新行为有所不同。这将在以下章节中详细说明。

请选择最适合您环境要求和管理实践的安装方法。

## 更新行为

### 开发人员管理的安装

对于直接安装 Docker Desktop 的开发人员：

- 对于大多数用户，Microsoft Store 不会自动更新像 Docker Desktop 这样的 Win32 应用程序。
- 只有一部分用户（约 20%）可能会在 Microsoft Store 页面上收到更新通知。
- 大多数用户必须在商店内手动检查并应用更新。

### Intune 管理的安装

在由 Intune 管理的环境中：

- Intune 大约每 8 小时检查一次更新。
- 检测到新版本时，Intune 会触发 `winget` 升级。
- 如果配置了适当的策略，更新可以在没有用户干预的情况下自动进行。
- 更新由 Intune 的管理基础结构处理，而不是由 Microsoft Store 本身处理。

## WSL 注意事项

Windows 版 Docker Desktop 与 WSL 紧密集成。更新从 Microsoft Store 安装的 Docker Desktop 时：

- 确保您已退出 Docker Desktop 且它不再运行，以便更新能够成功完成
- 在某些环境中，虚拟硬盘（VHDX）文件锁定可能会阻止更新完成

## Intune 管理建议

如果使用 Intune 管理 Windows 版 Docker Desktop：

- 确保您的 Intune 策略配置为处理应用程序更新
- 请注意更新过程使用 WinGet API 而不是直接的 Store 机制
- 考虑在受控环境中测试更新过程以验证其功能正常
