---
description: 如何将 Docker Scout 与其他系统集成。
keywords: supply chain, security, integrations, registries, ci, environments
title: 将 Docker Scout 集成至其他系统
linkTitle: 集成
weight: 80
---

默认情况下，Docker Scout 会与您的 Docker 组织以及在 Docker Hub 上启用 Docker Scout 的仓库集成。您还可以将 Docker Scout 与更多第三方系统对接，以获取更丰富的洞察信息，包括对正在运行的工作负载的实时可见性。

## 集成类别

根据集成的位置与方式不同，Docker Scout 能提供的洞察也会有所差异。

### 容器仓库（Registry）

将 Docker Scout 集成到第三方容器仓库后，Docker Scout 可以对这些仓库中的镜像执行分析，即使镜像并未托管在 Docker Hub 上，您也能了解其组成与风险。

可用的容器仓库集成：

- [Amazon Elastic Container Registry](./registry/ecr.md)
- [Azure Container Registry](./registry/acr.md)
- [JFrog Artifactory](./registry/artifactory.md)

### 持续集成（CI）

将 Docker Scout 接入持续集成（CI）系统，可以在开发内循环中即时、自动地反馈安全状况。运行在 CI 中的分析还能结合更多上下文信息，帮助您获得更深入的洞察。

可用的 CI 集成：

- [GitHub Actions](./ci/gha.md)
- [GitLab](./ci/gitlab.md)
- [Microsoft Azure DevOps Pipelines](./ci/azure.md)
- [Circle CI](./ci/circle-ci.md)
- [Jenkins](./ci/jenkins.md)

### 环境监控（Environment）

环境监控是指将 Docker Scout 与您的部署环境进行集成，从而获取正在运行的容器工作负载的实时信息。

接入环境后，您可以将生产环境中的工作负载与镜像仓库或其他环境中的版本进行对比分析。

可用的环境监控集成：

- [Sysdig](./environment/sysdig.md)

要了解更多环境集成内容，请参阅：[环境](./environment/_index.md)。

### 代码质量（Code Quality）

将 Docker Scout 与代码分析工具集成，可以在源代码层面进行质量检查，帮助跟踪缺陷、安全问题、测试覆盖率等。配合镜像分析与环境监控，代码质量门禁可让您的供应链管理进一步前移。

启用代码质量集成后，Docker Scout 会在已启用该集成的仓库中，将代码质量评估结果纳入策略评估。

可用的代码质量集成：

- [SonarQube](sonarqube.md)

### 源代码管理（SCM）

将 Docker Scout 与版本控制系统集成，可在代码仓库中直接获得有关镜像分析问题的修复建议。

可用的源代码管理集成：

- [GitHub](source-code-management/github.md) {{< badge color=blue text=Beta >}}

### 团队协作（Collaboration）

该类别的集成用于将软件供应链相关的通知实时推送到团队沟通平台。

可用的团队协作集成：

- [Slack](./team-collaboration/slack.md)
