---
title: 变更你的订阅
description: 升级或降级你的 Docker 订阅，并了解相关计费变更
keywords: 订阅升级, 订阅降级, docker 定价, 订阅变更
aliases:
- /docker-hub/upgrade/
- /docker-hub/billing/upgrade/
- /subscription/upgrade/
- /subscription/downgrade/
- /subscription/core-subscription/upgrade/
- /subscription/core-subscription/downgrade/
- /docker-hub/cancel-downgrade/
- /docker-hub/billing/downgrade/
- /billing/scout-billing/
- /billing/subscription-management/
weight: 30
---

{{% include "tax-compliance.md" %}}

你可以在任何时间升级或降级 Docker 订阅，以适配不断变化的需求。本文介绍如何进行订阅变更，以及计费与功能访问将会如何变化。

> [!NOTE]
>
> 旧版 Docker 订阅用户在进行订阅变更时，界面可能与当前不同。旧版订阅适用于最后一次购买或续订发生在 2024 年 12 月 10 日之前的用户。详情参见《[Announcing Upgraded Docker Plans](https://www.docker.com/blog/november-2024-updated-plans-announcement/)》。

## 升级你的订阅

当你升级 Docker 订阅后，将立即获得新订阅层级的全部功能与权益。功能详情参见 [Docker 定价](https://www.docker.com/pricing)。

{{< tabs >}}
{{< tab name="Docker 订阅" >}}

升级订阅：

1. 登录 [Docker Home](https://app.docker.com/)，选择需要升级的组织。
1. 进入 **Billing**。
1. 可选：如果你从免费 Personal 升级至 Team 且希望保留当前用户名，请先[将用户账号转换为组织](../admin/organization/convert-account.md)。
1. 选择 **Upgrade**。
1. 按照页面指引完成升级。

> [!NOTE]
>
> 如果选择使用美国银行账户付款，你必须先完成账户验证。更多信息参见《[Verify a bank account](manuals/billing/payment-method.md#verify-a-bank-account)》。

{{< /tab >}}
{{< tab name="旧版 Docker 订阅" >}}

如需将旧版 Docker 订阅升级为可访问全部工具的新 Docker 订阅，请联系 [Docker 销售](https://www.docker.com/pricing/contact-sales/)。

{{< /tab >}}
{{< /tabs >}}

## 降级你的订阅

你可以在续费日期之前的任何时间降级 Docker 订阅。已支付但未使用的部分不予退款，但你可在当前计费周期结束前继续使用付费功能。

### 降级注意事项

降级前请注意：

- 团队规模与仓库：根据新订阅的限制，你可能需要减少团队成员数量，并将私有仓库转为公共或删除。
- SSO 与 SCIM：如果从 Docker Business 降级，且组织使用单点登录（SSO），请先移除 SSO 连接与已验证域名。通过 SCIM 自动开通的成员需要重置密码，才能在无 SSO 的情况下登录。
- 私有仓库协作者：Personal 订阅不包含私有仓库协作者。当从 Pro 降级至 Personal 时，所有协作者将被移除，额外的私有仓库将被锁定。

各层级功能限制，参见 [Docker 定价](https://www.docker.com/pricing)。

{{< tabs >}}
{{< tab name="Docker 订阅" >}}

> [!IMPORTANT]
>
> 如果你使用的是[销售协助的 Docker Business 订阅](details.md#sales-assisted)，请联系你的客户经理办理降级。

降级订阅：

1. 登录 [Docker Home](https://app.docker.com/)，选择需要降级的组织。
1. 进入 **Billing**。
1. 点击操作图标，选择 **Cancel subscription**。
1. 填写反馈问卷以继续完成取消。

{{< /tab >}}
{{< tab name="旧版 Docker 订阅" >}}

> [!IMPORTANT]
>
> 如果你使用的是[销售协助的 Docker Business 订阅](details.md#sales-assisted)，请联系你的客户经理办理降级。

降级旧版 Docker 订阅：

1. 登录 [Docker Hub](https://hub.docker.com/billing)。
1. 选择需要降级的组织，然后进入 **Billing**。
1. 要执行降级，你需要先进入升级计划页面，点击 **Upgrade**。
1. 在升级页面的 **Free Team** 计划卡片中选择 **Downgrade**。
1. 按照页面指引完成降级。

降级 Docker Build Cloud 订阅：

1. 登录 [Docker Home](https://app.docker.com)，选择 **Build Cloud**。
1. 进入 **Account settings**，然后选择 **Downgrade**。
1. 为确认降级，在文本框中输入 **DOWNGRADE**，并选择 **Yes, continue**。
1. 账户设置页面将显示通知条，提示你的降级生效日期（下一个计费周期开始时）。

{{< /tab >}}
{{< /tabs >}}

## 订阅暂停策略

订阅无法暂停或延迟。如果订阅发票未在到期日支付，自到期日起将开始 15 天的宽限期。
