---
title: 更改你的计费周期
weight: 50
description: 了解如何为 Docker 订阅更改计费周期
keywords: billing, cycle, payments, subscription
---

购买 Docker 订阅时，你可以选择按月或按年计费。如果当前为月度计费，也可以切换为年度计费。

> [!NOTE]
>
> Docker Business 订阅仅提供年度计费。

> [!NOTE]
>
> 不支持从年度计费切换回月度计费。

当你更改计费周期时：

- 下一个计费日期将按照新周期计算。查看下一个计费日期，参见[查看续费日期](history.md#view-renewal-date)。
- 订阅的起始日期会被重置。例如，月度订阅 3 月 1 日开始、4 月 1 日结束；若在 2024 年 3 月 15 日切换计费周期，则新的起始日期为 2024 年 3 月 15 日，结束日期为 2025 年 3 月 15 日。
- 月度订阅未使用的部分将按比例（prorate）折算为年度订阅的抵扣额。例如，若月费为 $10、已使用价值为 $5，切换为年费 $100 时，最终收费为 $95（$100-$5）。

{{% include "tax-compliance.md" %}}

## 将个人账号切换为年度计费

{{< tabs >}}
{{< tab name="Docker subscription" >}}

按以下步骤将 Docker 订阅从月度切换为年度计费：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 选择 **Billing**。
1. 在计划与用量页面选择 **Switch to annual billing**。
1. 核对账单信息。
1. 选择 **Continue to payment**。
1. 核对付款信息并选择 **Upgrade subscription**。

> [!NOTE]
>
> 如果选择使用美国银行账户支付，必须先完成账户验证。详见[验证银行账户](manuals/billing/payment-method.md#verify-a-bank-account)。

完成更改后，“计划与用量”页面会显示更新后的年度订阅详情。

{{< /tab >}}
{{< tab name="Legacy Docker subscription" >}}

按以下步骤将旧版 Docker 订阅从月度切换为年度计费：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择你的组织，然后选择 **Billing**。
1. 在 **Plan** 标签页右下角选择 **Switch to annual billing**。
1. 查看 **Change to an Annual subscription** 页面显示的信息，并选择 **Accept Terms and Purchase** 以确认。

{{< /tab >}}
{{< /tabs >}}

## 将组织切换为年度计费

> [!NOTE]
>
> 只有组织所有者可以更改付款信息。

{{< tabs >}}
{{< tab name="Docker subscription" >}}

按以下步骤将组织的 Docker 订阅从月度切换为年度计费：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 选择 **Billing**。
1. 在计划与用量页面选择 **Switch to annual billing**。
1. 核对账单信息。
1. 选择 **Continue to payment**。
1. 核对付款信息并选择 **Upgrade subscription**。

> [!NOTE]
>
> 如果选择使用美国银行账户支付，必须先完成账户验证。详见[验证银行账户](manuals/billing/payment-method.md#verify-a-bank-account)。

{{< /tab >}}
{{< tab name="Legacy Docker subscription" >}}

按以下步骤将旧版 Docker 组织订阅从月度切换为年度计费：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择你的组织，然后选择 **Billing**。
1. 选择 **Switch to annual billing**。
1. 查看 **Change to an Annual subscription** 页面信息并选择 **Accept Terms and Purchase** 以确认。

{{< /tab >}}
{{< /tabs >}}
