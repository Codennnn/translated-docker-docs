---
title: 将 Docker Scout 与 Sysdig 集成
linkTitle: Sysdig
description: 使用 Sysdig 将运行时环境与 Docker Scout 集成
keywords: scout, sysdig, integration, image analysis, environments, supply chain
---

{{% include "scout-early-access.md" %}}

通过 Sysdig 集成，Docker Scout 能自动发现您当前运行的工作负载所使用的镜像。启用该集成后，您可以实时了解安全状况，并将构建产物与生产环境中实际运行的版本进行对比。

## 工作原理

Sysdig Agent 会采集容器工作负载所使用的镜像。Docker Scout 通过集成 Sysdig API 来发现集群中的镜像。该集成依赖 Sysdig 的 Risk Spotlight 功能。详情参见：[Risk Spotlight Integrations（Sysdig 文档）](https://docs.sysdig.com/en/docs/sysdig-secure/integrations-for-sysdig-secure/risk-spotlight-integrations/)。

> [!TIP]
>
> Sysdig 为 Docker 用户提供了免费试用，便于体验全新的 Docker Scout 集成。
>
> {{< button url=`https://sysdig.com/free-trial-for-docker-customers/` text="Sign up" >}}

每个 Sysdig 集成都对应一个环境。当您启用某个 Sysdig 集成时，需要为该集群指定环境名称（例如 `production` 或 `staging`）。Docker Scout 会将集群中的镜像分配到对应环境。这样，您就可以按环境筛选并查看漏洞状态与策略合规性。

只有经过 Docker Scout 分析的镜像才能被分配到环境。Sysdig 运行时集成本身不会触发镜像分析。若要自动分析镜像，请启用[仓库集成](../_index.md#container-registries)。

镜像分析不必先于运行时集成进行，但只有当 Docker Scout 完成镜像分析后，环境分配才会生效。

## 先决条件

- 在需要集成的集群中安装 Sysdig Agent，参见：[Install Sysdig Agent（Sysdig 文档）](https://docs.sysdig.com/en/docs/installation/sysdig-monitor/install-sysdig-agent/)。
- 在 Sysdig 中为 Risk Spotlight 集成启用 profiling，参见：[Profiling（Sysdig 文档）](https://docs.sysdig.com/en/docs/sysdig-secure/policies/profiling/#enablement)。
- 您必须是组织所有者，才能在 Docker Scout 控制台中启用该集成。

## 集成环境

1. 打开 Docker Scout 控制台的 [Sysdig 集成页面](https://scout.docker.com/settings/integrations/sysdig/)。
2. 在 **How to integrate** 区域，为此次集成输入一个配置名称。Docker Scout 会将该名称用作显示名与 webhook 名称。

3. 选择 **Next**。

4. 输入 Risk Spotlight API Token，并在下拉列表中选择区域（region）。

   Risk Spotlight API Token 是 Docker Scout 集成 Sysdig 所需的 Sysdig 令牌。关于如何生成该令牌，参见：[Risk Spotlight Integrations（Sysdig 文档）](https://docs.sysdig.com/en/docs/sysdig-secure/integrations-for-sysdig-secure/risk-spotlight-integrations/docker-scout/#generate-a-token-for-the-integration)。

   所选区域对应部署 Sysdig Agent 时设置的 `global.sysdig.region` 配置参数。

5. 选择 **Next**。

   点击 **Next** 后，Docker Scout 会连接 Sysdig 并拉取您账户下的集群名称。集群名称对应部署 Sysdig Agents 时设置的 `global.clusterConfig.name` 配置参数。

   若使用提供的令牌连接 Sysdig 失败，将显示错误，且无法继续集成。请返回检查并修正配置。

6. 在下拉列表中选择一个集群名称。

7. 选择 **Next**。

8. 为该集群指定要关联的环境名称。

    您可以复用现有环境，或新建一个环境。

9. 选择 **Enable integration**。

启用集成后，Docker Scout 会自动发现集群中正在运行的镜像，并将这些镜像分配到与该集群关联的环境。关于环境的更多信息，参见：[环境监控](./_index.md)。

> [!NOTE]
>
> Docker Scout 仅会检测已完成分析的镜像。若要触发镜像分析，请启用[仓库集成](../_index.md#container-registries)并将镜像推送到您的仓库。
>
> 如果您为此次集成新建了环境，则在至少有一个镜像完成分析后，该环境才会出现在 Docker Scout 中。

如需集成更多集群，请前往 [Sysdig 集成页面](https://scout.docker.com/settings/integrations/sysdig/) 并选择 **Add** 按钮。
