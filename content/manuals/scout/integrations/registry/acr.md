---
description: 将 Azure Container Registry 与 Docker Scout 集成
keywords: docker scout, acr, azure, integration, image analysis, security, cves
title: 将 Docker Scout 与 Azure Container Registry 集成
linkTitle: Azure Container Registry
---

将 Docker Scout 与 Azure Container Registry（ACR）集成后，您可以查看托管在 ACR 仓库中的镜像洞察。在完成 ACR 集成并对目标仓库启用 Docker Scout 后，向该仓库推送镜像会自动触发镜像分析。您可以在 Docker Scout 控制台或使用 `docker scout` CLI 查看分析结果。

## 工作原理

为帮助您将 ACR 与 Docker Scout 集成，我们提供了一份自定义的 Azure Resource Manager（ARM）模板，它会在 Azure 中自动创建所需的基础设施：

- 用于镜像推送与删除事件的 Event Grid 主题与订阅
- 仓库的只读授权 Token，用于列出仓库并拉取镜像

当这些资源在 Azure 中创建完成后，您即可在已集成的 ACR 实例中为镜像仓库启用集成。一旦为某个仓库启用集成，后续推送的新镜像会自动触发分析，分析结果将显示在 Docker Scout 控制台。

如果在启用集成时，仓库中已存在镜像，Docker Scout 会自动拉取并分析最新的镜像版本。

### ARM 模板

下表描述了需要配置的资源：

> [!NOTE]
>
> 在 Azure 账户中创建这些资源会产生少量的持续费用。表格中的 **Cost** 列给出了当集成的 ACR 每日有 100 次镜像推送时的月度费用估算。
>
> 出口（Egress）费用随用量而变化，约为 $0.1/GB，且前 100 GB 免费。

| Azure                   | Resource                                                                                   | Cost                                              |
| ----------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------- |
| Event Grid system topic | 订阅 ACR 事件（镜像推送与删除）                                                             | Free                                              |
| Event subscription      | 通过 Webhook 订阅将 Event Grid 事件发送到 Scout                                             | $0.60/100 万条消息，前 10 万条免费                 |
| Registry Token          | 只读 Token，供 Scout 列出仓库与拉取镜像使用                                                 | Free                                              |

下方 JSON 展示了 Docker Scout 用于在 Azure 中创建资源的 ARM 模板。

{{< accordion title="JSON template" >}}

{{< acr-template.inline >}}
{{ with resources.GetRemote "https://prod-scout-integration-templates.s3.amazonaws.com/latest/acr_token_template.json" }}
{{ $data := .Content | transform.Unmarshal }}

```json
{{ transform.Remarshal "json" $data }}
```

{{ end }}
{{< /acr-template.inline >}}

{{< /accordion >}}

## 集成一个仓库

1. 打开 Docker Scout 控制台的 [ACR 集成页面](https://scout.docker.com/settings/integrations/azure/)。
2. 在 **How to integrate** 区域，输入要集成的 **Registry hostname**。
3. 选择 **Next**。
4. 选择 **Deploy to Azure**，在 Azure 中打开模板部署向导。

   若您尚未登录 Azure，系统会提示先登录。

5. 在模板向导中配置您的部署：

   - **Resource group**：填写与该容器仓库相同的资源组。Docker Scout 相关资源必须部署在与仓库相同的资源组中。

   - **Registry name**：该字段会预填为仓库主机名的子域。

6. 选择 **Review + create**，随后点击 **Create** 部署模板。

7. 等待部署完成。
8. 在 **Deployment details** 区域，点击新创建的 **Container registry token** 资源，为该 Token 生成一个新密码。
    
    或者，您也可以在 Azure 中搜索并进入目标 **Container registry** 资源，为新创建的访问 Token 生成密码。

9. 复制生成的密码，返回 Docker Scout 控制台完成集成。

10. 将生成的密码粘贴到 **Registry token** 字段。
11. 选择 **Enable integration**。

点击 **Enable integration** 后，Docker Scout 会执行连通性测试以验证配置是否正确。验证成功后，您会被重定向到 Azure 仓库摘要页面，其中显示当前组织下的全部 Azure 集成。

接下来，请在[仓库设置](https://scout.docker.com/settings/repos/)中为需要分析的仓库启用 Docker Scout。

启用后，您推送的镜像会被 Docker Scout 分析，结果显示在控制台。如果仓库已存在镜像，Docker Scout 会自动拉取并分析最新版本。

## 移除集成

> [!IMPORTANT]
>
> 在 Docker Scout 控制台移除集成并不会自动删除 Azure 中创建的资源。

移除 ACR 集成：

1. 前往 Docker Scout 控制台的 [ACR 集成页面](https://scout.docker.com/settings/integrations/azure/)。
2. 找到要移除的 ACR 集成并点击 **Remove** 按钮。
3. 在弹出的对话框中选择 **Remove** 确认。
4. 在 Docker Scout 控制台移除集成后，请同步删除 Azure 中与该集成相关的资源：

   - 容器仓库中的 `docker-scout-readonly-token` Token
   - `docker-scout-repository` Event Grid System Topic
