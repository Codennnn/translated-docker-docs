---
title: Docker 扩展 UI 样式概述
linkTitle: 设计和 UI 样式
description: Docker 扩展设计
keywords: Docker, extensions, design
aliases:
 - /desktop/extensions-sdk/design/design-overview/
 - /desktop/extensions-sdk/design/overview/
 - /desktop/extensions-sdk/design/
weight: 60
---

我们的设计系统是一套不断发展的规范，旨在确保 Docker 产品之间的视觉一致性，并满足 [AA 级无障碍标准](https://www.w3.org/WAI/WCAG2AA-Conformance)。我们已向扩展开发者开放了部分系统内容，包括基本样式（颜色、排版）和组件。请参阅：[Docker 扩展样式指南](https://www.figma.com/file/U7pLWfEf6IQKUHLhdateBI/Docker-Design-Guidelines?node-id=1%3A28771)。

我们要求扩展在一定程度上与更广泛的 Docker Desktop UI 保持一致，并保留在未来使这一要求更加严格的权利。

要开始构建您的 UI，请遵循以下步骤。

## 步骤一：选择您的框架

### 推荐：使用我们的主题的 React+MUI

Docker Desktop 的 UI 是使用 React 和 [MUI](https://mui.com/)（具体来说是 Material UI）编写的。这是构建扩展的唯一官方支持框架，也是我们的 `init` 命令为您自动配置的框架。使用它为开发者带来显著优势：

- 您可以使用我们的 [Material UI 主题](https://www.npmjs.com/package/@docker/docker-mui-theme) 自动复制 Docker Desktop 的外观和感觉。
- 未来，我们将发布专门针对这种组合的工具和组件（例如自定义 MUI 组件，或用于与 Docker 交互的 React hooks）。

阅读我们的 [MUI 最佳实践](mui-best-practices.md) 指南，了解在 Docker Desktop 中使用 MUI 的面向未来的方法。

### 不推荐：其他框架

您可能更喜欢使用另一个框架，也许是因为您或您的团队更熟悉它，或者因为您有想要重用的现有资源。这是可能的，但非常不推荐。这意味着：

- 您需要手动复制 Docker Desktop 的外观和感觉。这需要大量工作，如果您不能足够接近地匹配我们的主题，用户会觉得您的扩展不协调，我们可能会在审查过程中要求您进行更改。
- 您将有更高的维护负担。每当 Docker Desktop 的主题发生变化（可能在任何版本中发生），您都需要手动更改扩展以匹配它。
- 如果您的扩展是开源的，故意避免常见约定将使社区更难为其做出贡献。

## 步骤二：遵循以下建议

### 遵循我们的 MUI 最佳实践（如果适用）

请参阅我们的 [MUI 最佳实践](mui-best-practices.md) 文章。

### 仅使用我们调色板中的颜色

除了少数例外，例如显示您的徽标，您应该只使用我们调色板中的颜色。这些可以在我们的 [样式指南文档](https://www.figma.com/file/U7pLWfEf6IQKUHLhdateBI/Docker-Design-Guidelines?node-id=1%3A28771) 中找到，并且很快也将在我们的 MUI 主题和通过 CSS 变量提供。

### 在浅色/深色模式中使用对应的颜色

我们的颜色经过精心选择，使得调色板的每个变体中的对应颜色应具有相同的基本特征。无论在浅色模式还是深色模式中，您都应该使用相同的颜色（例如，在浅色模式中使用 `red-300`，在深色模式中也使用 `red-300`）。

## 下一步

- 查看我们的 [MUI 最佳实践](mui-best-practices.md)。
- 了解如何[发布您的扩展](../extensions/_index.md)。
