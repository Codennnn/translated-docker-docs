---
title: PKG 安装程序
description: 了解如何使用 PKG 安装程序，并探索额外的配置选项。
keywords: pkg, mac, docker desktop, 安装, 部署, 配置, 管理员, mdm
tags: [admin]
weight: 20
aliases:
 - /desktop/setup/install/enterprise-deployment/pkg-install-and-configure/
---

{{< summary-bar feature_name="PKG installer" >}}

PKG 安装包支持各种 MDM（移动设备管理）解决方案，使其成为批量安装的理想选择，无需个别用户手动设置。使用此安装包，IT 管理员可以确保 Docker Desktop 的标准化、策略驱动的安装，从而提高整个组织的效率和软件管理水平。

## 交互式安装

1. 在 [Docker Home](http://app.docker.com) 中，选择您的组织。
2. 选择 **Admin Console（管理控制台）**，然后选择 **Enterprise deployment（企业部署）**。
3. 在 **macOS** 选项卡中，选择 **Download PKG installer（下载 PKG 安装程序）** 按钮。
4. 下载完成后，双击 `Docker.pkg` 运行安装程序。
5. 按照安装向导的说明授权安装程序并继续安装。
   - **Introduction（介绍）**：选择 **Continue（继续）**。
   - **License（许可协议）**：查看许可协议并选择 **Agree（同意）**。
   - **Destination Select（目标选择）**：此步骤为可选步骤。建议保留默认安装目标（通常为 `Macintosh HD`）。选择 **Continue（继续）**。
   - **Installation Type（安装类型）**：选择 **Install（安装）**。
   - **Installation（安装）**：使用管理员密码或 Touch ID 进行身份验证。
   - **Summary（摘要）**：安装完成后，选择 **Close（关闭）**。

> [!NOTE]
>
> 使用 PKG 安装 Docker Desktop 时，应用内更新会自动禁用。这确保组织可以保持版本一致性并防止未经批准的更新。对于使用 `.dmg` 安装程序安装的 Docker Desktop，应用内更新仍然受支持。
>
> 当有更新可用时，Docker Desktop 会通知您。要更新 Docker Desktop，请从 Docker 管理控制台下载最新的安装程序。导航到 **Enterprise deployment（企业部署）** 页面。
>
> 要了解最新版本发布情况，请查看 [发布说明](/manuals/desktop/release-notes.md) 页面。

## 从命令行安装

1. 在 [Docker Home](http://app.docker.com) 中，选择您的组织。
2. 选择 **Admin Console（管理控制台）**，然后选择 **Enterprise deployment（企业部署）**。
3. 在 **macOS** 选项卡中，选择 **Download PKG installer（下载 PKG 安装程序）** 按钮。
4. 在终端中运行以下命令：

   ```console
   $ sudo installer -pkg "/path/to/Docker.pkg" -target /Applications
   ```

## 其他资源

- 了解如何使用 [Intune](use-intune.md) 或 [Jamf Pro](use-jamf-pro.md) 部署 Mac 版 Docker Desktop
- 探索如何为您的用户[强制登录](/manuals/enterprise/security/enforce-sign-in/methods.md#plist-method-mac-only)