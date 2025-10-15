---
title: 使用图形界面创建例外（Exception）
description: 通过 Docker Scout 控制台或 Docker Desktop，为镜像中的漏洞创建例外。
keywords: Docker, Docker Scout, Docker Desktop, vulnerability, exception, create, GUI
---

Docker Scout 控制台与 Docker Desktop 提供了直观易用的界面，
用于为容器镜像中发现的漏洞创建[例外](/manuals/scout/explore/exceptions.md)。
通过例外，你可以在镜像分析中承认可接受的风险，或标记并处理误报。

## 先决条件

要在 Docker Scout 控制台或 Docker Desktop 中创建例外，你需要使用拥有该镜像的 Docker 组织中，
具备 **Editor** 或 **Owner** 权限的 Docker 账号登录。

## 操作步骤

使用 Docker Scout 控制台或 Docker Desktop 为镜像中的漏洞创建例外：

{{< tabs >}}
{{< tab name="Docker Scout Dashboard" >}}

1. 前往 [Images 页面](https://scout.docker.com/reports/images)。
2. 选择包含目标漏洞的镜像标签（tag）。
3. 打开 **Image layers** 选项卡。
4. 选择包含该漏洞的镜像层（layer）。
5. 在 **Vulnerabilities** 选项卡中，找到要为其创建例外的漏洞。
   漏洞按软件包分组；先找到包含该漏洞的包并展开。
6. 点击漏洞旁的 **Create exception** 按钮。

{{% create_panel.inline %}}
点击 **Create exception** 按钮后，会打开 **Create exception** 侧边面板。
在该面板中，你可以填写例外的详细信息：

- **Exception type**：例外类型。目前支持：

  - **Accepted risk**：因风险可接受、修复成本过高、依赖上游修复等原因，暂不修复该漏洞。
  - **False positive**：基于你的具体使用场景、配置或现有防护措施，该漏洞不构成实际风险（误报）。

    若选择 **False positive**，必须提供为何判定为误报的理由：

- **Additional details**：关于该例外的任何补充说明。

- **Scope**：例外的生效范围，可选：

  - **Image**：仅对当前所选镜像生效。
  - **All images in repository**：对该仓库中的所有镜像生效。
  - **Specific repository**：对指定的仓库中的所有镜像生效。
  - **All images in my organization**：对你所在组织的所有镜像生效。

- **Package scope**：例外在软件包层面的作用范围：

  - **Selected package**：仅对所选软件包生效。
  - **Any packages**：对受该 CVE 影响的所有软件包生效。

填写完所有信息后，点击 **Create** 创建例外。

例外创建完成后，会自动纳入你所选镜像的分析结果中。
你也可以在 Docker Scout 控制台的 [Vulnerabilities 页面 — Exceptions 标签](https://scout.docker.com/reports/vulnerabilities/exceptions)
查看该例外的列表。

{{% /create_panel.inline %}}

{{< /tab >}}
{{< tab name="Docker Desktop" >}}

1. 在 Docker Desktop 中打开 **Images** 视图。
2. 切换到 **Hub** 选项卡。
3. 选择包含目标漏洞的镜像标签（tag）。
4. 选择包含该漏洞的镜像层（layer）。
5. 在 **Vulnerabilities** 选项卡中找到需要创建例外的漏洞。
6. 点击漏洞旁的 **Create exception** 按钮。

{{% create_panel.inline / %}}

{{< /tab >}}
{{< /tabs >}}
