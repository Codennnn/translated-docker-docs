---
title: 使用提供方服务（Provider Services）
description: 了解如何在 Docker Compose 中使用提供方服务，将外部平台能力集成到你的应用中
keywords: compose, docker compose, provider, services, platform capabilities, integration, model runner, ai
weight: 112
params:
  sidebar:
    badge:
      color: green
      text: New
---

{{< summary-bar feature_name="Compose provider services" >}}

Docker Compose 支持“提供方服务（provider services）”，它允许你集成那些由第三方组件而非 Compose 自身管理生命周期的服务。  
借助该特性，你可以声明并使用特定平台的服务，而无需手动搭建或直接管理其生命周期。

## 什么是提供方服务？

提供方服务是 Compose 中的一类特殊服务，它代表的是平台能力而非容器。
它允许你为应用声明对特定平台功能的依赖。

当你在 Compose 文件中定义了一个提供方服务后，Compose 会与对应平台协作来供应与配置所请求的能力，并将其提供给你的应用服务使用。

## 如何使用提供方服务

要在 Compose 文件中使用提供方服务，你需要：

1. 定义一个包含 `provider` 属性的服务
2. 指定要使用的提供方 `type`
3. 配置提供方特定的 `options`
4. 在应用服务中声明对该提供方服务的依赖

示例：

```yaml
services:
  database:
    provider:
      type: awesomecloud
      options:
        type: mysql
        foo: bar  
  app:
    image: myapp 
    depends_on:
       - database
```

请注意 `database` 服务中的 `provider` 专用属性。
该属性表明该服务由某个“提供方”管理，并允许你为该类型提供方设置特定选项。

`app` 服务中的 `depends_on` 表示它依赖于 `database` 服务。
这意味着 `database` 会先于 `app` 启动，从而使提供方注入的信息能够传递给 `app`。

## 工作原理

在执行 `docker compose up` 的过程中，Compose 会识别出依赖提供方的服务并与提供方协作去供应所请求的能力。随后，提供方会向 Compose 模型填充访问这些已供应资源所需的信息。

这些信息会传递给声明了依赖关系的服务，通常是通过环境变量的形式。变量的命名约定为：

```env
<<PROVIDER_SERVICE_NAME>>_<<VARIABLE_NAME>>
```

例如，如果你的提供方服务名为 `database`，你的应用服务可能会收到如下环境变量：

- `DATABASE_URL`：访问已供应资源所需的 URL
- `DATABASE_TOKEN`：认证令牌
- 其他与提供方相关的变量

随后，你的应用即可使用这些环境变量与已供应资源进行交互。

## 提供方类型

提供方服务的 `type` 字段可以引用以下两者之一：

1. Docker CLI 插件（例如 `docker-model`）
2. 位于用户 PATH 中的可执行文件

当 Compose 遇到提供方服务时，会查找具有该名称的插件或可执行文件，以处理所请求能力的供应。

例如，当你指定 `type: model` 时，Compose 会在 PATH 中查找名为 `docker-model` 的 CLI 插件，或名为 `model` 的可执行文件。

```yaml
services:
  ai-runner:
    provider:
      type: model  # 查找 docker-model 插件或名为 model 的可执行文件
      options:
        model: ai/example-model
```

该插件或可执行文件负责：

1. 解析提供方服务中给定的 options
2. 供应所请求的能力
3. 返回如何访问已供应资源的信息

这些信息随后会以环境变量的形式传递给依赖该提供方的服务。

> [!TIP]
>
> 如果你在 Compose 中使用 AI 模型，请优先考虑使用 [`models` 顶层元素](/manuals/ai/compose/models-and-compose.md)。

## 使用提供方服务的好处

在 Compose 应用中使用提供方服务具有以下优点：

1. 配置更简化：无需手动配置和管理平台能力
2. 声明式方式：可在同一处声明应用的全部依赖
3. 一致的工作流：使用同样的 Compose 命令管理整个应用，包括平台能力

## 自定义提供方

如果你希望自定义提供方以扩展 Compose 的能力，可以实现一个 Compose 插件来注册提供方类型。

关于如何创建与实现自定义提供方的详细信息，请参阅 [Compose 扩展文档](https://github.com/docker/compose/blob/main/docs/extension.md)。  
该文档介绍了扩展机制，你可以通过它向 Compose 添加新的提供方类型。

## 参考

- [Docker Model Runner 文档](/manuals/ai/model-runner.md)
- [Compose 扩展文档](https://github.com/docker/compose/blob/main/docs/extension.md)