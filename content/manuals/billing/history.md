---
title: 查看账单历史
weight: 40
description: 了解如何查看与下载账单历史、管理发票并查看订阅续费日期。
keywords: payments, billing, subscription, invoices, renewals, invoice management, billing administration, download invoices, VAT, billing support, Docker billing
aliases:
    - /billing/core-billing/history/
---

本文介绍如何查看账单历史、管理发票以及确认续费日期。所有月度与年度订阅都会在周期结束时，使用原付款方式自动续订。

{{% include "tax-compliance.md" %}}

## 发票

发票包含以下信息：

- 发票编号
- 开具日期
- 到期日期
- “Bill to”（付款方）信息
- 应付金额（USD）
- 订单描述、（如适用）数量、单价与金额（USD）

发票中 **Bill to** 区域的信息来自你的账单信息，并非所有字段都必填。账单信息包括：

- 名称（必填）：管理员或公司名称
- 邮箱地址（必填）：用于接收该账号所有计费相关邮件的邮箱
- 地址（必填）
- 电话号码
- 税号或 VAT

发票一旦开具便无法更改。更新账单信息不会影响已开具的发票。如果需要变更账单信息，请务必在订阅续费（发票最终生成）之前完成。参见[更新账单信息](details.md)。

### 查看续费日期

{{< tabs >}}
{{< tab name="Docker subscription" >}}

订阅续费时你会收到发票。要查看续费日期，请登录 [Docker Billing](https://app.docker.com/billing)。续费日期与金额会显示在订阅卡片上。


{{< /tab >}}
{{< tab name="Legacy Docker subscription" >}}

订阅续费时你会收到发票。要查看续费日期：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 点击用户头像打开下拉菜单。
1. 选择 **Billing**。
1. 选择用户或组织账号查看计费详情，其中包含续费日期与金额。

{{< /tab >}}
{{< /tabs >}}

### 在发票上包含 VAT 编号

> [!NOTE]
>
> 若页面中未显示 VAT 编号字段，请填写[联系支持表单](https://hub.docker.com/support/contact/)。该字段可能需要人工添加。

{{< tabs >}}
{{< tab name="Docker subscription" >}}

添加或更新 VAT 编号：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 选择 **Billing**。
1. 在左侧菜单选择 **Billing information**。
1. 在账单信息卡片上选择 **Change**。
1. 勾选 **I'm purchasing as a business**。
1. 在 Tax ID 区域输入你的 VAT 编号。

    > [!IMPORTANT]
    >
    > VAT 编号必须包含国家前缀。例如，德国应填写 `DE123456789`。

1. Select **Update**.

你的 VAT 编号将显示在下一张发票上。

{{< /tab >}}
{{< tab name="Legacy Docker subscription" >}}

添加或更新 VAT 编号：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择你的组织，然后选择 **Billing**。
1. 选择 **Billing address** 链接。
1. 在 **Billing Information** 区域选择 **Update information**。
1. 在 Tax ID 区域输入你的 VAT 编号。

    > [!IMPORTANT]
    >
    > VAT 编号必须包含国家前缀。例如，德国应填写 `DE123456789`。

1. Select **Save**.

你的 VAT 编号将显示在下一张发票上。

{{< /tab >}}
{{< /tabs >}}

## 查看账单历史

你可以为个人账号或组织查看账单历史并下载历史发票。

### 个人账号

{{< tabs >}}
{{< tab name="Docker subscription" >}}

查看账单历史：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 选择 **Billing**。
1. 在左侧菜单选择 **Invoices**。
1. 可选：点击 **Invoice number** 查看发票详情。
1. 可选：点击 **Download** 下载发票。

{{< /tab >}}
{{< tab name="Legacy Docker subscription" >}}

查看账单历史：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择你的组织，然后选择 **Billing**。
1. 选择 **Payment methods and billing history** 链接。

历史发票位于 **Invoice History** 区域，可在其中下载。

{{< /tab >}}
{{< /tabs >}}

### 组织

> [!NOTE]
>
> 只有组织所有者可以查看账单历史。

{{< tabs >}}
{{< tab name="Docker subscription" >}}

查看账单历史：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 选择 **Billing**。
1. 在左侧菜单选择 **Invoices**。
1. 可选：点击 **invoice number** 查看发票详情。
1. 可选：点击 **download** 下载发票。

{{< /tab >}}
{{< tab name="Legacy Docker subscription" >}}

查看账单历史：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择你的组织，然后选择 **Billing**。
1. 选择 **Payment methods and billing history** 链接。

历史发票位于 **Invoice History** 区域，可在其中下载。

{{< /tab >}}
{{< /tabs >}}
