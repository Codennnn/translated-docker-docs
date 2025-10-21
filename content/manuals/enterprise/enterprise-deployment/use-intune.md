---
title: 使用 Intune 部署
description: 使用 Intune（Microsoft 基于云的设备管理工具）部署 Docker Desktop
keywords: microsoft, windows, docker desktop, 部署, mdm, 企业, 管理员, mac, pkg, dmg
tags: [admin]
weight: 40
aliases:
 - /desktop/install/msi/use-intune/
 - /desktop/setup/install/msi/use-intune/
 - /desktop/setup/install/enterprise-deployment/use-intune/
---

{{< summary-bar feature_name="Intune" >}}

了解如何使用 Microsoft Intune 在 Windows 和 macOS 设备上部署 Docker Desktop。内容涵盖应用创建、安装程序配置以及分配给用户或设备。

{{< tabs >}}
{{< tab name="Windows" >}}

1. 登录到您的 Intune 管理中心。
2. 添加新应用。选择 **Apps（应用）**，然后选择 **Windows**，再选择 **Add（添加）**。
3. 对于应用类型，选择 **Windows app (Win32)（Windows 应用 (Win32)）**
4. 选择 `intunewin` 软件包。 
5. 填写所需详细信息，如描述、发布者或应用版本，然后选择 **Next（下一步）**。 
6. 可选：在 **Program（程序）** 选项卡上，您可以更新 **Install command（安装命令）** 字段以满足您的需求。该字段预填充了 `msiexec /i "DockerDesktop.msi" /qn`。有关可进行更改的示例，请参阅 [常见安装场景](msi-install-and-configure.md)。 

   > [!TIP]
   >
   > 建议您配置 Intune 部署，在成功安装后计划重新启动计算机。
   >
   > 这是因为 Docker Desktop 安装程序会根据您选择的引擎安装 Windows 功能，并且还会更新 `docker-users` 本地组的成员身份。
   >
   > 您可能还需要设置 Intune 根据返回代码确定行为，并注意返回代码 `3010`。返回代码 3010 表示安装成功但需要重新启动。

7. 完成剩余选项卡，然后查看并创建应用。 

{{< /tab >}}
{{< tab name="Mac" >}}

首先，上传软件包：

1. 登录到您的 Intune 管理中心。
2. 添加新应用。选择 **Apps（应用）**，然后选择 **macOS**，再选择 **Add（添加）**。
3. 选择 **Line-of-business app（业务线应用）**，然后选择 **Select（选择）**。
4. 上传 `Docker.pkg` 文件并填写所需详细信息。

接下来，分配应用：

1. 添加应用后，在 Intune 中导航至 **Assignments（分配）**。
2. 选择 **Add group（添加组）** 并选择要分配应用的用户组或设备组。
3. 选择 **Save（保存）**。

{{< /tab >}}
{{< /tabs >}}

## 其他资源

- [探索常见问题解答](faq.md)。
- 了解如何为您的用户[强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)。