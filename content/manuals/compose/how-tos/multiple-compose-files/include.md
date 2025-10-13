---
description: 如何使用 Docker Compose 的 include 顶级元素
keywords: compose, docker, include, compose file
title: 包含（Include）
aliases:
- /compose/multiple-compose-files/include/
---

{{< summary-bar feature_name="Compose include" >}}

{{% include "compose/include.md" %}}

[`include` 顶级元素](/reference/compose-file/include.md) 有助于在配置文件的组织结构中，直接体现负责该代码的工程团队。同时，它还解决了 [`extends`](extends.md) 和 [合并](merge.md) 带来的相对路径问题。

`include` 部分中列出的每个路径都会作为独立的 Compose 应用模型加载，并使用其自身的项目目录来解析相对路径。

当包含的 Compose 应用加载完成后，其所有资源都会被复制到当前的 Compose 应用模型中。

> [!NOTE]
>
> `include` 具备递归性：如果被包含的 Compose 文件中也声明了 `include`，这些文件也会被一并包含。

## 示例

```yaml
include:
  - my-compose-include.yaml  # 已声明 serviceB
services:
  serviceA:
    build: .
    depends_on:
      - serviceB # 可直接使用 serviceB，就像它在当前 Compose 文件中声明的一样
```

`my-compose-include.yaml` 管理 `serviceB`，其中定义了副本数、用于查看数据的 Web UI、隔离的网络、用于持久化的数据卷等细节。依赖 `serviceB` 的应用无需了解这些基础设施细节，只需将该 Compose 文件视为可复用的模块即可。

这意味着负责 `serviceB` 的团队可以重构其数据库组件、引入更多服务，而不会影响到任何依赖方团队。同时，依赖方也无需在每条 Compose 命令中添加额外的参数。

```yaml
include:
  - oci://docker.io/username/my-compose-app:latest # 使用存储为 OCI 制品的 Compose 文件
services:
  serviceA:
    build: .
    depends_on:
      - serviceB 
```
`include` 允许你引用来自远程来源的 Compose 文件，例如 OCI 制品或 Git 仓库。
此处的 `serviceB` 定义于存储在 Docker Hub 上的 Compose 文件中。

## 为被包含的 Compose 文件使用覆盖（override）

如果 `include` 引入的资源与被包含 Compose 文件中的资源发生冲突，Compose 会报错。此规则用于避免与被包含文件作者定义的资源产生意外冲突。不过，在某些情况下你可能需要自定义被包含的模型。可以通过在 include 指令上添加覆盖文件（override）来实现：

```yaml
include:
  - path : 
      - third-party/compose.yaml
      - override.yaml  # 对三方模型的本地覆盖
```

这种方式的主要限制是：你需要为每个 include 维护一份独立的覆盖文件。对于包含多个 include 的复杂项目，这会产生大量 Compose 文件。

另一种方式是使用全局的 `compose.override.yaml` 文件。尽管在使用 `include` 的文件中声明相同资源会导致冲突被拒绝，但全局的 Compose 覆盖文件可以覆盖最终合并得到的模型，如下例所示：

Main `compose.yaml` file:
```yaml
include:
  - team-1/compose.yaml # 声明 service-1
  - team-2/compose.yaml # 声明 service-2
```

Override `compose.override.yaml` file:
```yaml
services:
  service-1:
    # 覆盖被包含的 service-1，开启调试端口
    ports:
      - 2345:2345

  service-2:
    # 覆盖被包含的 service-2，使用包含测试数据的本地数据目录
    volumes:
      - ./data:/data
```

这样组合起来，你既能受益于第三方可复用组件，又能按需调整最终的 Compose 模型。

## 参考信息

[`include` 顶级元素](/reference/compose-file/include.md)
