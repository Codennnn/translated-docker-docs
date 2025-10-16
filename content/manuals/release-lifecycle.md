---
title: Docker 产品发布生命周期 
linkTitle: 产品发布生命周期
description: 介绍功能从 Beta 到 GA 的各生命周期阶段，
  并说明产品的退休（退役）流程。
keywords: beta, GA, Early Access,
params:
  sidebar:
    group: Products
---

本文详细说明 Docker 的产品发布生命周期以及各阶段的定义，并提供产品退休（退役）流程的信息。功能与产品可能会经历其中的部分或全部阶段。 

>[!NOTE]
>
>Our [Subscription Service Agreement](https://www.docker.com/legal/docker-subscription-service-agreement) governs your use of Docker and covers details of eligibility, content, use, payments and billing, and warranties. This document is not a contract and all use of Docker’s services are subject to Docker’s [Subscription Service Agreement](https://www.docker.com/legal/docker-subscription-service-agreement).

## 生命周期阶段

| 生命周期阶段 | 面向客户可用性 | 支持可用性 | 限制 | 退役 |
| --- | --- | ---- | ---| ---|
|[实验](#experimental)| 有限可用 | 社区支持 | 软件可能存在功能缺失、缺陷和/或稳定性问题 | 可在不另行通知的情况下停止 |
|[Beta](#beta) | 面向全部用户或参与 Beta 反馈计划的用户 | 社区支持 | 软件可能存在功能缺失、缺陷和/或稳定性问题 | 可在不另行通知的情况下停止 |
|[早期访问（EA）](#early-access-ea) | 面向全部用户或参与 EA 反馈计划的用户 | 完整支持 | 软件可能存在功能、性能与/或 API 限制，这些限制会在文档中注明。 | 遵循[退役流程](#retirement-process) |
|[正式发布（GA）](#general-availability-ga) | 全部用户 | 完整支持 | 对已支持的用例几乎没有或没有限制 | 遵循[退役流程](#retirement-process) |

### 实验 {#experimental}

实验（Experimental）阶段的条目表示 Docker 正在尝试中的功能。获得实验功能访问权限的用户可以抢先测试、验证并反馈未来能力，帮助我们将精力聚焦在对用户最有价值的方向。

**面向客户可用性：** 实验功能的可用性有限。部分用户可能无法访问，也可能仅能访问部分或多个实验功能。 

**支持：** 实验功能通过社区渠道与论坛提供尽力而为的支持。

**限制：** 实验功能可能存在较大限制，例如功能、性能或 API 的限制。功能与编程接口可能随时变更，恕不另行通知。

**退役：** 在实验期间，Docker 会评估是否让该条目继续推进生命周期。我们有权在任何时间更改范围或停止实验阶段的产品或功能，且无需另行通知，详见我们的[订阅服务协议](https://www.docker.com/legal/docker-subscription-service-agreement)。

### Beta {#beta}

Beta 阶段表示潜在未来产品或功能的初始发布。参与 Beta 计划的用户可测试、验证并反馈未来能力，帮助我们聚焦对用户最有价值的内容。

**面向客户可用性：** 通过邀请参与 Beta，或在产品内使用明确标注为 Beta 的功能。邀请可能是公开或私有的。

**支持：** Beta 功能通过社区渠道与论坛提供尽力而为的支持。

**限制：** Beta 版本可能存在较大限制，例如功能、性能或 API 的限制。功能与编程接口可能随时变更，恕不另行通知。

**退役：** 在 Beta 期间，Docker 会评估是否让该条目继续推进生命周期。我们有权在任何时间更改范围或停止 Beta 阶段的产品或功能，且无需另行通知，详见我们的[订阅服务协议](https://www.docker.com/legal/docker-subscription-service-agreement)。

### 早期访问（EA） {#early-access-ea}

早期访问（EA）阶段的产品或功能可能仍存在一定的功能限制，并以循序渐进的方式面向特定用户群体启用。它们已基本具备向所有用户发布的成熟度，仅需进一步微调。

**面向客户可用性：** EA 功能可以面向全部客户或特定用户分群发布，作为现有功能的补充或替代。

**支持：** EA 阶段与 GA 阶段享有同等级别的支持。

**限制：** EA 版本可能存在较大限制，例如功能、性能或 API 的限制，这些限制会在文档中注明。对功能或编程接口的破坏性变更将遵循下文的[退役流程](#retirement-process)。

**退役：** 若在进入 GA 前需要退役某个 EA 产品，我们将尽力遵循下文所述的[退役流程](#retirement-process)。

### 正式发布（GA） {#general-availability-ga}

正式发布（GA）阶段表示面向所有 Docker 客户开放、功能完备的产品或功能。

**面向客户可用性：** 所有 Docker 用户均可根据其订阅级别访问 GA 项目。

**限制：** 对已支持的用例而言，GA 功能与产品几乎没有或没有限制。

**支持：** 所有 GA 项目均提供完整支持，详见我们的[支持页面](https://www.docker.com/support/)。

**退役：** GA 阶段遵循下文的[退役流程](#retirement-process)。


## 退役流程 {#retirement-process}

关于是否退役或弃用某项功能，Docker 会遵循严格流程进行决策，其中包括评估需求、使用情况、退役影响，尤其是用户反馈。我们的目标是将资源投入到能为最多用户创造价值的方向。

Docker 致力于在与客户沟通时保持清晰、透明与积极，尤其是在涉及平台变更方面。因此，在退役功能时，我们将尽力遵循以下准则：

- **提前通知：** 对于主要功能或产品的退役，我们将尽量至少提前 6 个月通知客户。
- **可行替代方案：** 在退役功能时，Docker 将努力为客户提供可行的替代方案，可能来自 Docker 本身，或为第三方提供商的推荐方案。在可行且合适的情况下，Docker 会自动将客户迁移至替代方案。
- **持续支持：** 在达到退役日期前，Docker 承诺持续为该功能提供支持。

在某些特殊情况下，我们可能需要加速退役时间表，例如为保护平台完整性或保障客户与他人安全而进行的必要变更。在这类场景下，尽快完成变更尤为重要。

同样地，集成的第三方软件或服务也可能因为对方决定变更或退役其方案而需要退役。在此类情况下，退役节奏不由我们控制。

即便如此，我们仍将尽可能提前告知。
