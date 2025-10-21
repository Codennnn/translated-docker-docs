---
title: 使用管理控制台配置设置管理
linkTitle: 使用管理控制台
description: 使用 Docker 管理控制台在整个组织内配置并强制执行 Docker Desktop 设置
keywords: 管理控制台, 设置管理, 策略配置, 企业控制, docker desktop
weight: 20
aliases:
 - /security/for-admins/hardened-desktop/settings-management/configure-admin-console/
---

{{< summary-bar feature_name="Admin Console" >}}

使用 Docker 管理控制台为整个组织的 Docker Desktop 创建和管理设置策略。设置策略让您能够标准化配置、强制执行安全要求，并保持一致的 Docker Desktop 环境。

## 先决条件

在开始之前，请确保您具备以下条件：

- 安装了 [Docker Desktop 4.37.1 或更高版本](/manuals/desktop/release-notes.md)
- [已验证的域](/manuals/enterprise/security/single-sign-on/configure.md#step-one-add-and-verify-your-domain)
- 为您的组织[强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)
- Docker Business 订阅

> [!IMPORTANT]
>
> 您必须将用户添加到已验证的域，设置才能生效。

## 创建设置策略

要创建新的设置策略：

1. 登录 [Docker Home](https://app.docker.com/) 并选择您的组织。
1. 选择**管理控制台**，然后选择**Desktop 设置管理**。
1. 选择**创建设置策略**。
1. 提供名称和可选描述。

      > [!TIP]
      >
      > 您可以上传现有的 `admin-settings.json` 文件来预填充表单。
      管理控制台策略会覆盖本地 `admin-settings.json` 文件。

1. 选择策略应用对象：
   - 所有用户
   - 特定用户

      > [!NOTE]
      >
      > 用户特定策略会覆盖全局默认策略。在组织范围内应用策略之前，请先在小范围内测试。

1. 使用状态配置每个设置：
   - **用户定义**：用户可以更改设置。
   - **始终启用**：设置已开启并锁定。
   - **启用**：设置已开启但可以更改。
   - **始终禁用**：设置已关闭并锁定。
   - **禁用**：设置已关闭但可以更改。

      > [!TIP]
      >
      > 有关可配置设置的完整列表、支持的平台和配置方法，请参阅[设置参考](settings-reference.md)。

1. 选择**创建**以保存您的策略。

## 应用策略

设置策略在 Docker Desktop 重启且用户登录后生效。

对于新安装：

1. 启动 Docker Desktop。
1. 使用您的 Docker 账户登录。

对于现有安装：

1. 完全退出 Docker Desktop。
1. 重新启动 Docker Desktop。

> [!IMPORTANT]
>
> 用户必须完全退出并重新打开 Docker Desktop。从 Docker Desktop 菜单重启是不够的。

Docker Desktop 在启动时和运行期间每 60 分钟检查一次策略更新。

## 验证已应用的设置

应用策略后：

- Docker Desktop 将大多数设置显示为灰色
- 某些设置，特别是增强容器隔离配置，可能不会显示在 GUI 中
- 您可以通过检查系统上的 [`settings-store.json` 文件](/manuals/desktop/settings-and-maintenance/settings.md)来验证所有已应用的设置

## 管理现有策略

在管理控制台的**Desktop 设置管理**页面上，使用**操作**菜单来：

- 编辑或删除现有设置策略
- 将设置策略导出为 `admin-settings.json` 文件
- 将用户特定策略提升为新的全局默认策略

## 回滚策略

要回滚设置策略：

- 完全回滚：删除整个策略。
- 部分回滚：将特定设置设置为**用户定义**。

当您回滚设置时，用户将重新获得对这些设置配置的控制权。
