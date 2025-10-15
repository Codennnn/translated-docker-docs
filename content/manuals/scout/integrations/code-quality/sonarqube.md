---
title: 将 Docker Scout 与 SonarQube 集成
linkTitle: SonarQube
description: 使用项目中定义的 SonarQube 质量门禁评估镜像
keywords: scout, supply chain, integration, code quality
---

通过 SonarQube 集成，Docker Scout 可在策略评估（Policy Evaluation）中展示 SonarQube 的质量门禁检查，对应于新的[SonarQube 质量门禁策略](/manuals/scout/policy/_index.md#sonarqube-quality-gates-policy)。

## 工作原理

该集成通过 [SonarQube webhooks](https://docs.sonarsource.com/sonarqube/latest/project-administration/webhooks/) 在项目分析完成时通知 Docker Scout。Webhook 被调用后，Docker Scout 会接收分析结果并存储于数据库。

当您向仓库推送新镜像时，Docker Scout 会基于与该镜像关联的 SonarQube 分析记录进行评估。Docker Scout 使用镜像中的 Git 溯源元数据（来自溯源声明或 OCI 注解）来将镜像仓库与 SonarQube 的分析结果关联起来。

> [!NOTE]
>
> Docker Scout 无法访问 SonarQube 的历史分析记录。只有在启用该集成之后产生的分析结果，才会在 Docker Scout 中可用。

同时支持自管理的 SonarQube 实例与 SonarCloud。

## 前置条件

将 Docker Scout 与 SonarQube 集成之前，请确保：

- 镜像仓库已[接入 Docker Scout](../_index.md#container-registries)。
- 镜像在构建时包含[溯源声明（provenance attestations）](/manuals/build/metadata/attestations/slsa-provenance.md)，或包含 `org.opencontainers.image.revision` 注解，用于指明 Git 仓库信息。

## 启用 SonarQube 集成

1. 打开 Docker Scout 控制台的 [SonarQube 集成页面](https://scout.docker.com/settings/integrations/sonarqube/)。
2. 在 **How to integrate** 区域，为此次集成输入配置名称。Docker Scout 将该名称用作显示名，并用于命名 webhook。
3. 选择 **Next**。
4. 输入您的 SonarQube 实例的配置信息。Docker Scout 将基于这些信息创建 SonarQube webhook。

   在 SonarQube 中[生成新的 **User token**](https://docs.sonarsource.com/sonarqube/latest/user-guide/user-account/generating-and-using-tokens/#generating-a-token)。该 token 需要目标项目的 Administer 权限，或全局 Administer 权限。

   输入该 token、SonarQube URL，以及 SonarQube 组织（使用 SonarCloud 时必填）。

5. 选择 **Enable configuration**。

   Docker Scout 会执行连通性测试，以验证配置信息是否正确，以及 token 是否具备所需权限。

6. 连通性测试成功后，系统将跳转到 SonarQube 集成概览页面，列出您所有的 SonarQube 集成及其状态。

在集成概览页面，您可以直接进入 **SonarQube Quality Gates Policy**。该策略初始不会显示结果。要开始看到评估结果，请触发一次项目的 SonarQube 分析，并将对应镜像推送到仓库。更多信息见：[策略说明](../../policy/_index.md#sonarqube-quality-gates)。
