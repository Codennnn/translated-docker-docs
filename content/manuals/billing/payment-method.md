---
title: 添加或更新付款方式
weight: 20
description: 了解如何在 Docker Hub 中添加或更新付款方式
keywords: payments, billing, subscription, supported payment methods, failed payments, add credit card, bank transfer, Stripe Link, payment failure
aliases:
    - /billing/core-billing/payment-method/
---

本文介绍如何为个人账号或组织添加或更新付款方式。

你可以随时添加或更新账号的现有付款方式。

> [!IMPORTANT]
>
> 如需移除所有付款方式，必须先将订阅降级为免费订阅。参见[降级](../subscription/change.md)。

支持以下付款方式：

- 信用卡（Cards）
  - Visa
  - MasterCard
  - American Express
  - Discover
  - JCB
  - Diners
  - UnionPay
- 钱包（Wallets）
  - Stripe Link
- 银行账户（Bank accounts）
  - 使用已[验证](manuals/billing/payment-method.md#verify-a-bank-account)的美国银行账户进行 ACH 转账

所有费用均以美元（USD）结算。

{{% include "tax-compliance.md" %}}

## 管理付款方式

### 个人账号

{{< tabs >}}
{{< tab name="Docker subscription" >}}

添加付款方式：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 选择 **Billing**。
1. 在左侧菜单选择 **Payment methods**。
1. 选择 **Add payment method**。
1. 输入新的付款信息：
    - 如果添加银行卡：
        - 选择 **Card** 并填写卡信息表单。
    - 如果添加 Link 付款：
        - 选择 **Secure, 1-click checkout with Link**，输入 Link **email address** 与 **phone number**。
        - 若尚未使用 Link，需要先填写卡信息表单以在 Link 中保存卡片。
    - 如果添加银行账户：
        - 选择 **US bank account**。
        - 验证你的 **Email** 与 **Full name**。
        - 若列表中有你的银行，选择其名称；若未列出，选择 **Search for your bank**。
        - 账户验证参见 [Verify a bank account](manuals/billing/payment-method.md#verify-a-bank-account)。
1. 选择 **Add payment method**。
1. 可选：通过 **Set as default** 将其设为默认付款方式。
1. 可选：通过 **Delete** 删除非默认付款方式。

> [!NOTE]
>
> 如果要将美国银行账户设为默认付款方式，必须先完成账户验证。

{{< /tab >}}
{{< tab name="Legacy Docker subscription" >}}

添加付款方式：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择 **Billing**。
1. 点击 **Payment methods** 链接。
1. 选择 **Add payment method**。
1. 输入新的付款信息：
    - 如果添加银行卡：
        - 选择 **Card** 并填写卡信息表单。
    - 如果添加 Link 付款：
        - 选择 **Secure, 1-click checkout with Link**，输入 Link **email address** 与 **phone number**。
        - 若你尚未成为 Link 用户，需要先填写卡信息表单以在 Link 中保存卡片。
1. 选择 **Add**。
1. 点击 **Actions** 图标并选择 **Make default**，以确保新付款方式应用于所有购买与订阅。
1. 可选：点击 **Actions** 图标后选择 **Delete** 删除非默认付款方式。

{{< /tab >}}
{{< /tabs >}}

### 组织

> [!NOTE]
>
> 只有组织所有者可以修改付款信息。

{{< tabs >}}
{{< tab name="Docker subscription" >}}

添加付款方式：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 选择 **Billing**。
1. 在左侧菜单选择 **Payment methods**。
1. 选择 **Add payment method**。
1. 输入新的付款信息：
    - 如果添加银行卡：
        - 选择 **Card** 并填写卡信息表单。
    - 如果添加 Link 付款：
        - 选择 **Secure, 1-click checkout with Link**，输入 Link **email address** 与 **phone number**。
        - 若你尚未成为 Link 用户，需要先填写卡信息表单以在 Link 中保存卡片。
    - 如果添加银行账户：
        - 选择 **US bank account**。
        - 验证你的 **Email** 与 **Full name**。
        - 若列表中有你的银行，选择其名称；若未列出，选择 **Search for your bank**。
        - 账户验证参见 [Verify a bank account](manuals/billing/payment-method.md#verify-a-bank-account)。
1. 选择 **Add payment method**。
1. 可选：通过 **Set as default** 将其设为默认付款方式。
1. 可选：通过 **Delete** 删除非默认付款方式。

> [!NOTE]
>
> 如果要将美国银行账户设为默认付款方式，必须先完成账户验证。

{{< /tab >}}
{{< tab name="Legacy Docker subscription" >}}

To add a payment method:

1. Sign in to [Docker Hub](https://hub.docker.com).
1. Select your organization, then select **Billing**.
1. Select the **Payment methods** link.
1. Select **Add payment method**.
1. Enter your new payment information:
    - If you are adding a card:
        - Select **Card** and fill out the card information form.
    - If you are adding a Link payment:
        - Select **Secure, 1-click checkout with Link** and enter your
        Link **email address** and **phone number**.
        - If you are not an existing Link customer, you must fill out the
        card information form to store a card for Link payments.
1. Select **Add payment method**.
1. Select the **Actions** icon, then select **Make default** to ensure that
your new payment method applies to all purchases and subscriptions.
1. Optional. You can remove non-default payment methods by selecting
the **Actions** icon. Then, select **Delete**.

{{< /tab >}}
{{< /tabs >}}

## 验证银行账户

将银行账户用作付款方式时，有两种验证方式：

- 即时验证：Docker 支持多家主流银行的即时验证。
- 手动验证：其他银行需要手动验证。

{{< tabs >}}
{{< tab name="Instant verification" >}}

### 即时验证

要完成即时验证，需要在 Docker 计费流程中登录你的银行账户：

1. 选择 **US bank account** 作为付款方式。
1. 验证你的 **Email** 与 **Full name**。
1. 若列表中有你的银行，选择其名称；否则选择 **Search for your bank**。
1. 登录你的网上银行并查看条款；此协议允许 Docker 从所连接的银行账户扣款。
1. 选择 **Agree and continue**。
1. 选择要关联并验证的账户，点击 **Connect account**。

验证完成后，你会在弹窗中看到成功提示。

{{< /tab >}}
{{< tab name="Manual verification" >}}

### 手动验证

手动验证需要你填写银行对账单中的“小额存款”金额：

1. 选择 **US bank account** 作为付款方式。
1. 验证你的 **Email** 与 **First and last name**。
1. 选择 **Enter bank details manually instead**。
1. 输入银行信息：**Routing number** 与 **Account number**。
1. 点击 **Submit**。
1. 你将收到一封包含手动验证指引的邮件。

手动验证依赖“小额存款”。1–2 个工作日内，你的银行账户会收到一笔很小的存款（例如 $0.01）。打开验证邮件，输入该金额即可完成验证。

{{< /tab >}}
{{< /tabs >}}

## 付款失败

> [!NOTE]
>
> 你无法手动重试失败的付款。Docker 会根据重试计划自动重试。

如果订阅付款失败，自到期日（含）起有 15 天宽限期。Docker 将按以下计划重试 3 次：

- 到期日后第 3 天
- 上一次尝试后第 5 天
- 上一次尝试后第 7 天

每次重试失败后，会发送主题为 `Action Required - Credit Card Payment Failed` 的邮件，并附上未支付的发票。

宽限期结束仍未支付，订阅将降级为免费订阅，所有付费功能将被停用。
