---
title: 操作指南
description: 使用 Docker 加固镜像的分步指导，从发现到调试。
weight: 20
params:
  grid_howto:
    - title: 探索 Docker 加固镜像
      description: 学习如何在 Docker Hub 的 DHI 目录中查找和评估镜像仓库、变体、元数据和证明。
      icon: travel_explore
      link: /dhi/how-to/explore/
    - title: 镜像 Docker 加固镜像仓库
      description: 学习如何将镜像镜像到您组织的命名空间，并可选择推送到另一个私有仓库。
      icon: compare_arrows
      link: /dhi/how-to/mirror/
    - title: 自定义 Docker 加固镜像
      description: 学习如何根据您组织的需求自定义 DHI。
      icon: settings
      link: /dhi/how-to/customize/
    - title: 使用 Docker 加固镜像
      description: 学习如何在 Dockerfile、CI 流水线和标准开发工作流中拉取、运行和引用 Docker 加固镜像。
      icon: play_arrow
      link: /dhi/how-to/use/
    - title: 在 Kubernetes 中使用 Docker 加固镜像
      description: 学习如何在 Kubernetes 部署中使用 Docker 加固镜像。
      icon: play_arrow
      link: /dhi/how-to/k8s/
    - title: 管理 Docker 加固镜像
      description: 学习如何在您的组织中管理镜像和自定义的 Docker 加固镜像。
      icon: reorder
      link: /dhi/how-to/manage/
    - title: 将现有应用迁移到 Docker 加固镜像
      description: 跟随分步指南更新您的 Dockerfile，采用 Docker 加固镜像实现安全、最小化和生产就绪的构建。
      icon: directions_run
      link: /dhi/how-to/migrate/
    - title: 验证 Docker 加固镜像
      description: 使用 Docker Scout 或 cosign 验证 Docker 加固镜像的签名证明，如 SBOM、来源和漏洞数据。
      icon: check_circle
      link: /dhi/how-to/verify/
    - title: 扫描 Docker 加固镜像
      description: 学习如何使用 Docker Scout、Grype 或 Trivy 扫描 Docker 加固镜像中的已知漏洞。
      icon: bug_report
      link: /dhi/how-to/scan/
    - title: 通过策略强制使用 Docker 加固镜像
      description: 学习如何对 Docker 加固镜像使用镜像策略与 Docker Scout。
      icon: policy
      link: /dhi/how-to/policies/
    - title: 调试 Docker 加固镜像
      description: 使用 Docker Debug 检查基于加固镜像的运行中容器，无需修改镜像。
      icon: terminal
      link: /dhi/how-to/debug/
---

本节提供使用 Docker 加固镜像（DHI）的实用分步指导。无论您是首次评估 DHI 还是将其集成到生产 CI/CD 流水线中，这些主题都会引导您完成从发现到调试的采用旅程的每个阶段。

为了帮助您入门并保持安全，主题围绕使用 DHI 的典型生命周期进行组织。

## 生命周期流程

1. 在 DHI 目录中探索可用的镜像和元数据。
2. 将受信任的镜像镜像到您的命名空间或仓库。
3. 通过拉取、在开发和 CI 中使用，以及将现有应用迁移到使用安全、最小化的基础镜像，在您的工作流中采用 DHI。
4. 通过验证签名、SBOM 和来源，以及扫描漏洞来分析镜像。
5. 执行策略以维护安全和合规性。
6. 调试基于 DHI 的容器，无需修改镜像。

以下每个主题都与此生命周期中的一个步骤相对应，因此您可以自信地逐步完成探索、实施和持续维护。

## 分步主题

{{< grid
  items="grid_howto"
>}}