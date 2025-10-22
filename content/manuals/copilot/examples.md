---
title: Docker 代理示例 prompt
linkTitle: 示例 prompt
description: |
  探索与 Docker 代理交互的示例 prompt，学习如何自动化任务，如项目容器化或打开拉取请求。
weight: 30
---

{{< summary-bar feature_name="Docker GitHub Copilot" >}}

## 使用场景

以下是一些您可以向 Docker 代理提出的问题类型示例：

### 提出一般 Docker 问题

您可以提出关于 Docker 的一般性问题。例如：

- `@docker 什么是 Dockerfile？`
- `@docker 如何构建 Docker 镜像？`
- `@docker 如何运行 Docker 容器？`
- `@docker 'docker buildx imagetools inspect' 命令的作用是什么？`

### 获取项目容器化帮助

您可以请求代理帮助您容器化现有项目：

- `@docker 能否帮助为这个项目创建 compose 文件？`
- `@docker 能否为这个项目创建 Dockerfile？`

#### 打开拉取请求

Docker 代理将分析您的项目，生成必要的文件，并在适用情况下，提出使用必要的 Docker 资源创建拉取请求。

只有当代理生成新的 Docker 资源时，才会自动针对您的仓库打开拉取请求。

### 分析项目漏洞

代理可以帮助您使用 [Docker Scout](/manuals/scout/_index.md) 提高安全防护能力：

- `@docker 能否帮助我找到项目中的漏洞？`
- `@docker 我的项目是否包含任何不安全的依赖项？`

代理将使用 Docker Scout 分析您项目的依赖项，并报告您是否容易受到任何[已知 CVE](/manuals/scout/deep-dive/advisory-db-sources.md)的影响。

![Copilot 漏洞报告](images/copilot-vuln-report.png?w=500px&border=1)

## 限制

- 代理目前无法访问仓库中的特定文件，例如编辑器中当前打开的文件，或者您在聊天消息中传递文件引用的情况。

## 反馈

如遇到问题或有反馈，请访问 [GitHub 反馈仓库](https://github.com/docker/copilot-issues)。
