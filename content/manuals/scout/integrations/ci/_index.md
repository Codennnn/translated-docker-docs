---
description: 在持续集成流水线中使用 Docker Scout 的配置方法。
keywords: scanning, vulnerabilities, Hub, supply chain, security, ci, continuous integration,
  github actions, gitlab
title: 在持续集成中使用 Docker Scout
linkTitle: 持续集成
aliases:
- /scout/ci/
---

在持续集成（CI）流水线中，您可以在构建镜像的同时，通过 GitHub Action 或 Docker Scout CLI 插件对 Docker 镜像进行分析。

可用的集成：

- [GitHub Actions](gha.md)
- [GitLab](gitlab.md)
- [Microsoft Azure DevOps Pipelines](azure.md)
- [Circle CI](circle-ci.md)
- [Jenkins](jenkins.md)

您也可以在 CI/CD 流水线中加入运行时集成，在部署时将镜像分配到对应环境（例如 `production` 或 `staging`）。更多信息见：[环境监控](../environment/_index.md)。
