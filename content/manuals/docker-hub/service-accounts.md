---
description: Docker 服务账户
keywords: Docker, 服务, 账户, Docker Hub
title: 服务账户
weight: 50
---

{{% include "new-plans.md" %}}

> [!IMPORTANT]
>
> 自 2024 年 12 月 10 日起，增强版 Service Account（服务账号）附加包已不再提供。现有的服务账号协议将在其当前期限内继续有效，但不再支持购买或续订增强版附加包，客户需在新的订阅方案下续约。
>
> Docker 建议迁移至 [Organization Access Tokens（组织访问令牌，OAT）](/manuals/enterprise/security/access-tokens.md)，其可提供类似功能。

服务账号是一种用于自动化管理容器镜像或容器化应用的 Docker ID。服务账号通常用于自动化工作流中，且不会与组织成员共享个人 Docker ID。典型用例包括在 Docker Hub 上进行内容镜像，或在 CI/CD 流程中拉取镜像。

## 增强版服务账号附加包档位

关于增强版服务账号附加包的详细信息，请参阅下表：

| 档位 | 每日拉取次数上限\* |
| ------ | ------ |
| 1 | 5,000-10,000 |
| 2 | 10,000-25,000 |
| 3 | 25,000-50,000 |
| 4 | 50,000-100,000 |
| 5 | 100,000+ |

<sub>*服务账号在全年范围内，最多可在 20 天内超出上述拉取次数上限 25%，且无需额外费用。可按需提供使用量报告。<sub>