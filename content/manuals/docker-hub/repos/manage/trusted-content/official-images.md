---
title: Docker 官方镜像
description: |
  本文介绍 Docker 官方镜像的创建流程，
  以及你如何参与贡献或提供反馈。
keywords: docker official images, doi, 贡献, 上游, 开源
aliases:
- /trusted-content/official-images/contributing/
- /docker-hub/official_repos/
- /docker-hub/official_images/
---

> [!NOTE]
>
> Docker 正在为 Docker 官方镜像（DOI）逐步淘汰 Docker Content Trust（DCT）。
> 你应当开始规划迁移至其他镜像签名与验证方案（如 [Sigstore](https://www.sigstore.dev/) 或
> [Notation](https://github.com/notaryproject/notation#readme)）。Docker 将很快发布迁移指南以协助迁移。
> DCT 的完整弃用时间线正在最终敲定，稍后将公布。
>
> 详情请参阅：https://www.docker.com/blog/retiring-docker-content-trust/。

Docker 公司赞助了一支专门的团队，负责审核与发布 Docker 官方镜像中的全部内容。
该团队与上游软件维护者、安全专家以及更广泛的 Docker 社区协同合作。

虽然更推荐由上游软件作者维护其 Docker 官方镜像，但这并非硬性要求。
为 Docker 官方镜像创建与维护镜像是一个协作过程，
该过程在 [GitHub 上以公开方式进行](https://github.com/docker-library/official-images)，
鼓励所有人参与。任何人都可以提供反馈、贡献代码、提出流程改进建议，甚至提议新增官方镜像。

## 创建 Docker 官方镜像

从宏观上看，一个官方镜像始于一组 GitHub Pull Request 形式的提案。
以下 GitHub 仓库详细说明了提案要求：

- [Docker 官方镜像 GitHub 仓库](https://github.com/docker-library/official-images#readme)
- [Docker 官方镜像文档](https://github.com/docker-library/docs#readme)

Docker 官方镜像团队在社区贡献者的帮助下，会正式审阅每个提案并向作者给出反馈。
这一初审流程可能较为耗时，通常需要多轮往返沟通后才能被接受。

在评审过程中也会包含一些主观性考量，核心问题在于：「这个镜像是否具有普遍用途？」
例如，[Python](https://hub.docker.com/_/python/) Docker 官方镜像对广大的 Python 开发者社区是
“普遍有用”的；相反，一个上周刚写成的冷门文字冒险游戏镜像就不是。

一旦新提案被接受，作者需要对镜像与文档保持最新，并响应用户反馈。
Docker 负责在 Docker Hub 上构建并发布这些镜像。对 Docker 官方镜像的更新同样通过 Pull Request 流程进行，
尽管更新的评审流程会更为精简。Docker 官方镜像团队最终作为所有变更的把关者，帮助确保一致性、质量与安全性。

## 为 Docker 官方镜像提交反馈

所有 Docker 官方镜像的文档都包含 **用户反馈** 部分，涉及该存储库的具体反馈方式。
多数情况下，存放官方镜像 Dockerfile 的 GitHub 仓库也会启用活跃的 issue 跟踪。

关于 Docker 官方镜像的一般性反馈与支持问题，请前往 [Docker 社区 Slack](https://dockr.ly/comm-slack) 的 `#general` 频道。

如果你是 Docker 官方镜像的维护者或贡献者，且需要帮助或建议，
请使用 [Libera.Chat IRC](https://libera.chat) 上的 `#docker-library` 频道。
