---
title: 创建组织
weight: 10
description: 了解如何创建组织。
keywords: docker 组织, 组织, 创建组织, docker 团队, docker 管理控制台, 组织管理
aliases:
  - /docker-hub/orgs/
---

{{< summary-bar feature_name="Admin orgs" >}}

本页面描述如何创建组织。

## 先决条件

在开始创建组织之前：

- 您需要一个 [Docker ID](/accounts/create-account/)
- 查看 [Docker 订阅和功能](../../subscription/details.md)
  以确定为您的组织选择哪种订阅

## 创建组织

有多种方式可以创建组织。您可以：

- 使用管理控制台或 Docker Hub 中的 **创建组织** 选项创建新组织
- 将现有用户账户转换为组织

以下部分包含如何创建新组织的说明。有关将现有用户账户转换为组织的先决条件和详细说明，请参阅
[将账户转换为组织](/manuals/admin/organization/convert-account.md)。

要创建组织：

1. 登录 [Docker Home](https://app.docker.com/) 并导航到组织列表底部。
1. 选择 **创建新组织**。
1. 为您的组织选择订阅、计费周期，并指定您需要的席位数。有关团队版和商业版订阅提供的功能详情，请参阅 [Docker 定价](https://www.docker.com/pricing/)。
1. 选择 **继续到个人资料**。
1. 选择 **创建组织** 来创建新组织。
1. 输入 **组织命名空间**。这是您组织在 Docker Hub 中的官方唯一名称。创建组织后无法更改组织名称。

   > [!NOTE]
   >
   > 您不能为组织和您的 Docker ID 使用相同的名称。如果您想使用您的 Docker ID 作为组织名称，则必须先[将您的账户转换为组织](/manuals/admin/organization/convert-account.md)。

1. 输入您的 **公司名称**。这是您公司的全称。Docker
会在您的组织页面和您发布的任何公共镜像的详细信息中显示公司名称。您可以随时通过导航到
您组织的 **设置** 页面来更新公司名称。
1. 选择 **继续到计费** 继续。
1. 输入您组织的计费信息并选择 **继续到支付** 以继续到计费门户。
1. 提供您的付款详细信息并选择 **购买**。

您现在已经创建了一个组织。

## 查看组织

要在管理控制台中查看组织：

1. 登录 [Docker Home](https://app.docker.com) 并选择您的
组织。
1. 从左侧导航菜单中，选择 **管理控制台**。

管理控制台包含许多选项，让您能够
配置您的组织。

## 合并组织

> [!WARNING]
>
> 如果您要合并组织，建议在计费周期结束时进行。当您合并一个组织并降级另一个组织时，您将失去降级组织上的席位。Docker 不提供降级退款。

如果您有多个组织想要合并为一个，请完成以下步骤：

1. 根据次要组织的席位数，为您想要保留的主组织账户[购买额外席位](../../subscription/manage-seats.md)。
1. 手动将用户添加到主组织并从次要组织中移除现有用户。
1. 手动迁移您的数据，包括所有仓库。
1. 完成所有用户和数据迁移后，将次要账户[降级](../../subscription/change.md)为免费订阅。请注意，Docker 不提供在计费周期中途降级组织的退款。

> [!TIP]
>
> 如果您的组织有通过采购订单购买的 Docker Business 订阅，请联系 Docker 的支持团队或您的客户经理。

## 更多资源

- [视频：Docker Hub 组织](https://www.youtube.com/watch?v=WKlT1O-4Du8)
