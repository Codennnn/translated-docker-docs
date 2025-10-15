---
title: 将 Docker Scout 与 GitHub 集成
linkTitle: GitHub
description: 通过 GitHub 应用集成，在代码仓库中直接获取修复建议
keywords: scout, github, integration, image analysis, supply chain, remediation, source code
---

{{< summary-bar feature_name="Docker Scout GitHub" >}}

通过将 Docker Scout 的 GitHub 应用接入您的 GitHub 源码仓库，Docker Scout 能更好地了解镜像的构建来源，从而为您提供自动化、具备上下文的修复建议。

## 工作原理

启用 GitHub 集成后，Docker Scout 可以将镜像分析结果与源码直接关联。

在分析镜像时，Docker Scout 会检查[溯源声明（provenance attestations）](/manuals/build/metadata/attestations/slsa-provenance.md)，以定位该镜像对应的源码仓库位置。如果找到了源码位置，并且您已启用 GitHub 应用，Docker Scout 会解析用于构建镜像的 Dockerfile。

解析 Dockerfile 可以识别构建所用的基础镜像标签。基于这些标签，Docker Scout 能判断该标签是否已过时（即标签已指向不同的镜像摘要 digest）。例如，您使用 `alpine:3.18` 作为基础镜像，当维护者发布 3.18 的补丁版（包含安全修复）后，您使用的 `alpine:3.18` 可能不再是最新的。

出现这种情况时，Docker Scout 会通过[最新基础镜像策略](/manuals/scout/policy/_index.md#up-to-date-base-images-policy)提示不一致。如果启用了 GitHub 集成，您还会获得自动化的更新建议。了解 Docker Scout 如何帮助您自动改进供应链合规与安全状况，请参见[修复建议](../../policy/remediation.md)。

## 设置

将 Docker Scout 与您的 GitHub 组织集成：

1. 打开 Docker Scout 控制台的 [GitHub 集成页面](https://scout.docker.com/settings/integrations/github/)。
2. 点击 **Integrate GitHub app** 按钮跳转至 GitHub。
3. 选择要集成的组织。
4. 选择要集成 GitHub 组织中的全部仓库，或手动选择部分仓库。
5. 点击 **Install & Authorize** 将 Docker Scout 应用安装并授权到该组织。

   随后系统会重定向回 Docker Scout 控制台，在此可查看已激活的 GitHub 集成。

GitHub 集成现已启用。
