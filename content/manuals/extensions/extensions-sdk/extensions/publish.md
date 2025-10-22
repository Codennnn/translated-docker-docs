---
title: 在市场发布扩展
description: Docker 扩展分发
keywords: Docker, extensions, publish
aliases: 
 - /desktop/extensions-sdk/extensions/publish/
weight: 50
---

## 将扩展提交到市场

Docker Desktop 在 [Docker Desktop](https://open.docker.com/extensions/marketplace) 和 [Docker Hub](https://hub.docker.com/search?q=&type=extension) 的扩展市场中显示已发布的扩展。
扩展市场是一个供开发者发现扩展以提升开发体验，并提交自己的扩展供所有桌面用户使用的平台。

当您准备好在市场[发布扩展](DISTRIBUTION.md)时，可以[自行发布扩展](https://github.com/docker/extensions-submissions/issues/new?assignees=&labels=&template=1_automatic_review.yaml&title=%5BSubmission%5D%3A+)

> [!NOTE]
>
> 随着扩展市场不断为扩展用户和发布者添加新功能，您需要随时间推移维护您的扩展，以确保它在市场中保持可用状态。

> [!IMPORTANT]
>
> 目前，Docker 对扩展的手动审核流程已暂停。请通过[自动化提交流程](https://github.com/docker/extensions-submissions/issues/new?assignees=&labels=&template=1_automatic_review.yaml&title=%5BSubmission%5D%3A+)提交您的扩展。

### 提交前准备

在提交扩展之前，它必须通过[验证](validate.md)检查。

强烈建议您在提交扩展之前，确保扩展遵循本节概述的指南。如果您向 Docker 扩展团队请求审核但未遵循这些指南，审核过程可能会更长。

这些指南不会取代 Docker 的服务条款，也不能保证获得批准：
- 查看[设计指南](../design/design-guidelines.md)
- 确保[UI 样式](../design/_index.md)符合 Docker Desktop 指南
- 确保您的扩展支持浅色和深色模式
- 考虑扩展的新用户和现有用户的需求
- 与潜在用户测试您的扩展
- 测试您的扩展是否存在崩溃、错误和性能问题
- 在各种平台（Mac、Windows、Linux）上测试您的扩展
- 阅读[服务条款](https://www.docker.com/legal/extensions_marketplace_developer_agreement/)

#### 验证流程

提交的扩展会经过自动化验证流程。如果所有验证检查都成功通过，扩展将在几小时内发布到市场并供所有用户访问。
这是让开发者获得所需工具并在您不断完善/打磨扩展时获得反馈的最快方式。

> [!IMPORTANT]
>
> Docker Desktop 会缓存市场中可用扩展列表 12 小时。如果您在市场中看不到您的扩展，可以重启 Docker Desktop 以强制刷新缓存。
