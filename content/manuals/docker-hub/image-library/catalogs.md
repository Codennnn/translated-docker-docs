---
description: 探索 Docker Hub 的专题集合，例如生成式 AI 目录。
keywords: Docker Hub, Hub, 目录
title: Docker Hub 目录
linkTitle: 目录
weight: 60
---

Docker Hub 目录是经过信任、开箱即用的容器镜像与资源集合，面向特定开发需求量身定制。它们让你更容易找到高质量、预先验证的内容，从而能够更快、更有把握地构建、部署并管理应用。Docker Hub 中的目录可以：

- 简化内容发现：经过组织与策展的内容，便于按特定领域或技术查找合适的工具与资源。
- 降低复杂度：由 Docker 及其合作伙伴审查的可信资源，保障安全性、可靠性与最佳实践一致性。
- 加速开发：无需大量调研或繁琐配置，即可快速为应用集成先进能力。

以下章节概述了 Docker Hub 中的关键目录。

## MCP 目录

[MCP 目录](https://hub.docker.com/mcp/) 是用于发现、分享与运行 MCP（Model Context Protocol，模型上下文协议）兼容工具的集中、可信注册表。该目录与 Docker Hub 无缝集成，包含：

- 100+ 个经过验证的 MCP 服务器（以 Docker 镜像打包）
- 来自 New Relic、Stripe、Grafana 等合作伙伴的工具
- 具备发布者校验的版本化发布
- 通过 Docker Desktop 与 Docker CLI 简化“拉取即运行”的体验

每个服务器都运行在隔离的容器中，以确保行为一致并尽可能减少配置负担。对于使用 Claude Desktop 或其他 MCP 客户端的开发者，该目录提供了即插即用的工具扩展方式。

要了解 MCP 服务器的更多信息，请参阅 [MCP Catalog and Toolkit](../../ai/mcp-catalog-and-toolkit/_index.md)。

## AI 模型目录

[AI 模型目录](https://hub.docker.com/catalogs/models/) 提供可与 [Docker Model Runner](../../ai/model-runner/_index.md) 搭配使用的精选可信模型。该目录通过提供预打包、开箱即用的模型，旨在降低 AI 开发门槛；你可以使用熟悉的 Docker 工具来拉取、运行并与模型交互。

借助 AI 模型目录与 Docker Model Runner，你可以：

- 从 Docker Hub 或任一符合 OCI 标准的注册表拉取并提供模型服务
- 通过兼容 OpenAI 的 API 与模型交互
- 使用 Docker Desktop 或 CLI 在本地运行与测试模型
- 使用 `docker model` CLI 打包并发布模型

无论你是在构建生成式 AI 应用、将 LLM 集成进工作流，还是试验各类机器学习工具，AI 模型目录都能简化模型管理体验。
