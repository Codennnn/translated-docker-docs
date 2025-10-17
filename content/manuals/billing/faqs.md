---
title: 计费常见问题
linkTitle: 常见问题
description: 与计费相关的常见问题
keywords: billing, renewal, payments, faq
tags: [FAQ]
weight: 60
---

### 如果我的订阅付款失败会怎样？

如果订阅付款失败，自到期日（含）起有 15 天宽限期。Docker 将按以下计划重试扣款 3 次：

- 到期日后第 3 天
- 上一次尝试后第 5 天
- 上一次尝试后第 7 天

每次重试失败后，Docker 都会发送主题为 `Action Required - Credit Card Payment Failed` 的邮件通知，并附上未支付的发票。

宽限期结束仍未支付，订阅将降级为免费订阅，所有付费功能会被停用。

### 我可以手动重试失败的付款吗？

不可以。Docker 会按照[重试计划](/manuals/billing/faqs.md#what-happens-if-my-subscription-payment-fails)自动重试失败的付款。

为提高重试成功率，请确认你的默认付款方式为最新。若需更新默认付款方式，参见[管理付款方式](/manuals/billing/payment-method.md#manage-payment-method)。

### Docker 是否会收取销售税或增值税（VAT）？

Docker 在以下地区收取销售税或增值税：

- 美国客户：自 2024 年 7 月 1 日起开始收取销售税。
- 欧洲客户：自 2025 年 3 月 1 日起开始收取增值税（VAT）。
- 英国客户：自 2025 年 5 月 1 日起开始收取增值税（VAT）。

为确保税额评估准确，请确保你的账单信息以及 VAT/税号（若适用）为最新。参见[更新账单信息](/billing/details/)。

如享有免税资格，请参见[登记税务豁免证明](/billing/tax-certificate/)。

### Docker 是否提供学术优惠？

如需学术定价，请联系[Docker 销售团队](https://www.docker.com/company/contact)。
