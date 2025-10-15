---
title: 将 Docker Scout 与 Slack 集成
linkTitle: Slack
description: |
  将 Docker Scout 接入 Slack，在频道中实时接收漏洞与策略合规更新
keywords: scout, team collaboration, slack, notifications, updates
---

您可以通过创建 Slack webhook 并将其添加到 Docker Scout 控制台，将 Docker Scout 与 Slack 集成。当有新披露的漏洞影响到您的一个或多个镜像时，Docker Scout 会向您发送通知。

![Slack notification from Docker Scout](../../images/scout-slack-notification.png?border=true "Example Slack notification from Docker Scout")

## 工作原理

完成集成配置后，Docker Scout 会将与仓库相关的策略合规性与漏洞暴露变化，发送到与该 webhook 关联的 Slack 频道。

> [!NOTE]
>
> 仅针对每个仓库的“最后一次推送”的镜像标签触发通知。“最后一次推送”指最近被推送到仓库并被 Docker Scout 分析的镜像标签。如果最后一次推送的镜像未受到新披露的 CVE 影响，则不会触发通知。

关于 Docker Scout 通知的更多信息，请参见：[通知设置](/manuals/scout/explore/dashboard.md#notification-settings)

## 设置

添加 Slack 集成：

1. 创建一个 webhook，参见 [Slack 文档](https://api.slack.com/messaging/webhooks)。
2. 前往 Docker Scout 控制台的 [Slack 集成页面](https://scout.docker.com/settings/integrations/slack/)。
3. 在 **How to integrate** 部分输入一个 **Configuration name**。
   Docker Scout 会将其作为该集成的显示名，您可以将默认值改为更有意义的名称，例如 `#channel-name` 或所属团队名称。
4. 将刚创建的 webhook 粘贴到 **Slack webhook** 字段中。

   如需验证连接，可点击 **Test webhook** 按钮。Docker Scout 会向该 webhook 发送一条测试消息。

5. 选择是否为所有已启用 Scout 的镜像仓库开启通知，或仅填写需要发送通知的仓库名。
6. 准备就绪后，点击 **Create** 启用集成。

创建 webhook 后，Docker Scout 将开始把通知发送到与该 webhook 关联的 Slack 频道。

## 移除 Slack 集成

移除 Slack 集成：

1. 前往 Docker Scout 控制台的 [Slack 集成页面](https://scout.docker.com/settings/integrations/slack/)。
2. 为要移除的集成点击 **Remove** 图标。
3. 在确认对话框中再次选择 **Remove** 完成移除。
