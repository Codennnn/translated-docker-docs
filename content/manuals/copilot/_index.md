---
title: Docker for GitHub Copilot
params:
  sidebar:
    group: Products
    badge:
      color: violet
      text: EA
weight: 50
description: |
  了解如何使用 Docker for GitHub Copilot 扩展简化 Docker 相关任务。该集成帮助您在各种开发环境中通过 GitHub Copilot Chat 生成 Docker 资源、分析漏洞并自动化容器化。
keywords: Docker, GitHub Copilot, 扩展, Visual Studio Code, 聊天, 人工智能, 容器化
---

{{< summary-bar feature_name="Docker GitHub Copilot" >}}

[Docker for GitHub Copilot](https://github.com/marketplace/docker-for-github-copilot)
扩展将 Docker 的功能与 GitHub Copilot 集成，为应用容器化、生成 Docker 资源和分析项目漏洞提供帮助。此扩展帮助您在任何支持 GitHub Copilot Chat 的环境中简化 Docker 相关任务。

## 主要功能

Docker for GitHub Copilot 扩展的主要功能包括：

- 在任何支持 GitHub Copilot Chat 的环境中（如 GitHub.com 和 Visual Studio Code）询问有关容器化的问题并获得响应。
- 自动为项目生成 Dockerfile、Docker Compose 文件和 `.dockerignore` 文件。
- 直接从聊天界面使用生成的 Docker 资源打开拉取请求。
- 从 [Docker Scout](/manuals/scout/_index.md) 获取项目漏洞摘要，并通过命令行界面接收后续操作建议。

## 数据隐私

Docker 代理仅基于 Docker 的文档和工具进行训练，以协助容器化及相关任务。它无法访问您所提问题上下文之外的项目数据。

使用 Docker for GitHub Copilot 扩展时，如果获得用户授权，GitHub Copilot 可能会在其请求中包含对当前打开文件的引用。Docker 代理可以读取该文件以提供上下文相关的响应。

如果代理被要求检查漏洞或生成 Docker 相关资源，它将把引用的仓库克隆到内存存储中以执行必要的操作。

源代码或项目元数据永远不会被持久存储。问题和答案会保留用于分析和故障排除。Docker 代理处理的数据绝不会与第三方共享。

## 支持的语言

Docker for GitHub Copilot 扩展支持以下编程语言，用于从头开始容器化项目的任务：

- Go
- Java
- JavaScript
- Python
- Rust
- TypeScript
