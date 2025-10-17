---
title: 在 Docker 计费中使用 3D Secure 认证
linkTitle: 3D Secure 认证
description: Docker 计费支持 3D Secure（3DS）进行安全的支付验证。了解 3DS 在 Docker 订阅中的工作方式。
keywords: billing, renewal, payments, subscriptions, 3DS, credit card verification, secure payments, Docker billing security
weight: 40
---

Docker 支持 3D Secure（3DS），这是一层额外的身份验证，某些信用卡支付会要求启用。若你的银行或发卡机构要求 3DS，你可能需要在付款完成前先完成身份验证。

## 工作原理

当在结账过程中触发 3DS 检查时，银行或发卡机构可能会要求你验证身份。常见方式包括：

- 输入发送到手机的一次性密码
- 在手机银行应用中确认扣款
- 回答安全问题或使用生物识别

具体验证步骤以你的金融机构要求为准。

## 何时需要验证

在进行以下操作时，你可能会被要求验证身份：

- 开始[付费订阅](../subscription/setup.md)
- 将[计费周期](/billing/cycle/)从月度改为年度
- [升级订阅](../subscription/change.md)
- 为现有订阅[添加席位](../subscription/manage-seats.md)

如果需要 3DS 且你的付款方式支持，验证提示会在结账时显示。

## 支付验证问题排查

如果因 3DS 无法完成支付：

1. 重试交易。请确保在同一浏览器标签页内完成验证提示。
1. 使用其他付款方式。部分卡片可能不完全支持 3DS 或被拦截。
1. 联系你的银行。银行可能拦截了付款或 3DS 验证尝试。

> [!NOTE]
>
> 关闭广告拦截器或屏蔽弹窗的浏览器扩展，有助于 3DS 验证提示正常显示。
