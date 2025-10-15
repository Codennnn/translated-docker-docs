---
title: Docker Scout
weight: 40
keywords: scout, supply chain, vulnerabilities, packages, cves, scan, analysis, analyze
description:
  了解 Docker Scout，主动提升你的软件供应链安全
aliases:
  - /engine/scan/
params:
  sidebar:
    group: Products
grid:
  - title: 快速开始
    link: /scout/quickstart/
    description: 了解 Docker Scout 能做什么，以及如何上手。
    icon: explore
  - title: 镜像分析
    link: /scout/image-analysis/
    description: 揭示并深入剖析你的镜像构成。
    icon: radar
  - title: 安全公告数据库
    link: /scout/advisory-db-sources/
    description: 了解 Docker Scout 使用的信息来源。
    icon: database
  - title: 集成
    description: |
      将 Docker Scout 连接到你的 CI、镜像仓库及其他第三方服务。
    link: /scout/integrations/
    icon: multiple_stop
  - title: 仪表板
    link: /scout/dashboard/
    description: |
      Docker Scout 的 Web 界面。
    icon: dashboard
  - title: 策略
    link: /scout/policy/
    description: |
      确保你的制品遵循供应链最佳实践。
    icon: policy
  - title: 升级
    link: /subscription/change/
    description: |
      个人订阅包含最多 1 个仓库。如需更多请升级。
    icon: upgrade
---

容器镜像由多层与软件包组成，天然可能包含漏洞。
这些漏洞会影响容器与应用的安全。

Docker Scout 是一套用于主动提升软件供应链安全的解决方案。
通过分析你的镜像，Docker Scout 会汇总其组成清单（SBOM，软件物料清单）。
随后将该 SBOM 与持续更新的漏洞数据库比对，以定位安全弱点。

Docker Scout 是一个独立的服务与平台，你可以通过 Docker Desktop、Docker Hub、Docker CLI 以及 Docker Scout 仪表板进行使用。
它还支持与第三方系统集成，例如容器仓库与 CI 平台。

{{< grid >}}
