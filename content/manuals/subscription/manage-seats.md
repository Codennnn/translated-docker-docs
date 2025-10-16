---
title: 管理订阅席位
linkTitle: 管理席位
description: 为 Docker Team 与 Business 订阅添加或移除席位
keywords: manage seats, add seats, remove seats, subscription billing, team members
aliases:
- /docker-hub/billing/add-seats/
- /subscription/add-seats/
- /docker-hub/billing/remove-seats/
- /subscription/remove-seats/
- /subscription/core-subscription/add-seats/
- /subscription/core-subscription/remove-seats/
weight: 20
---

你可以随时为 Docker Team 或 Docker Business 订阅添加或移除席位，以适配团队规模变化。若在计费周期中途添加席位，将按比例收取新增席位的费用。

{{% include "tax-compliance.md" %}}

## 为订阅添加席位

{{< tabs >}}
{{< tab name="Docker 订阅" >}}

> [!IMPORTANT]
>
> 如果你使用的是[销售协助的 Docker Business 订阅](details.md#sales-assisted)，请联系你的客户经理以添加席位。

添加席位：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 进入 **Billing**。
1. 选择 **Add seats**，并根据页面提示完成添加。

> [!NOTE]
>
> 如果你选择使用美国银行卡账户支付，需要先完成账户验证。参见[验证银行账户](manuals/billing/payment-method.md#verify-a-bank-account)。

此后你即可为组织添加更多成员。参见[管理组织成员](../admin/organization/members.md)。

{{< /tab >}}
{{< tab name="旧版 Docker 订阅" >}}

> [!IMPORTANT]
>
> 如果你使用的是[销售协助的 Docker Business 订阅](details.md#sales-assisted)，请联系你的客户经理以添加席位。

在旧版 Docker 订阅中添加席位：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择你的组织，然后进入 **Billing**。
1. 在 Billing 页面选择 **Add seats**。
1. 选择要添加的席位数量，随后点击 **Purchase**。

为 Docker Build Cloud 添加席位：

1. 登录 [Docker Home](https://app.docker.com) 并进入 **Build Cloud**。
1. 选择 **Account settings**，然后点击 **Add seats**。
1. 选择要添加的席位数量，并点击 **Add seats**。

{{< /tab >}}
{{< /tabs >}}

## 批量定价

Docker 为 Docker Business 订阅提供 25 个席位起的批量定价。详情请联系 [Docker 销售团队](https://www.docker.com/pricing/contact-sales/)。

## 从订阅中移除席位

你可以随时从 Team 或 Business 订阅中移除席位。变更将于下一个计费周期生效，未使用的部分不予退款。

例如，你每月 8 日为 10 个席位进行结算，若在 15 日移除 2 个席位，则这 2 个席位会保留至下一计费周期开始；从下一周期起你将按 8 个席位计费。

{{< tabs >}}
{{< tab name="Docker 订阅" >}}

> [!IMPORTANT]
>
> 如果你使用的是[销售协助的 Docker Business 订阅](details.md#sales-assisted)，请联系你的客户经理以移除席位。

移除席位：

1. 登录 [Docker Home](https://app.docker.com/)，选择你的组织。
1. 进入 **Billing**。
1. 在 **Seats** 行点击操作图标，选择 **Remove seats**。
1. 根据页面提示完成移除。

在下一计费周期开始前，你可以撤销席位移除操作。点击 **Cancel change** 即可。

{{< /tab >}}
{{< tab name="旧版 Docker 订阅" >}}

> [!IMPORTANT]
>
> 如果你使用的是[销售协助的 Docker Business 订阅](details.md#sales-assisted)，请联系你的客户经理以移除席位。

在旧版 Docker 订阅中移除席位：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择你的组织，然后进入 **Billing**。
1. 在 Billing 页面选择 **Remove seats**。
1. 按页面提示完成移除。

从 Docker Build Cloud 移除席位：

1. 登录 [Docker Home](https://app.docker.com) 并进入 **Build Cloud**。
1. 选择 **Account settings**，然后点击 **Remove seats**。
1. 按页面提示完成移除。

{{< /tab >}}
{{< /tabs >}}