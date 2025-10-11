---
description: 了解标签（labels），这是一种用于管理 Docker 对象元数据的机制。
keywords: labels, metadata, docker, annotations
title: Docker 对象标签
aliases:
  - /engine/userguide/labels-custom-metadata/
  - /config/labels-custom-metadata/
---

标签是一种将元数据应用到 Docker 对象上的机制，适用于以下对象：

- 镜像（Images）
- 容器（Containers）
- 本地守护进程（Local daemons）
- 卷（Volumes）
- 网络（Networks）
- Swarm 节点（Swarm nodes）
- Swarm 服务（Swarm services）

你可以使用标签来组织镜像、记录许可信息、标注容器/卷/网络之间的关系，或以任何适合你业务与应用的方式来管理对象。

## 标签键与值

标签是一个以字符串形式存储的键值对。你可以为同一个对象指定多个标签，但在同一对象内每个键必须唯一。如果同一个键被多次赋值，则最新写入的值会覆盖之前的值。

### 键名格式建议

标签键是键值对的左侧。键名是字母数字字符串，可包含点（`.`）、下划线（`_`）、斜杠（`/`）和连字符（`-`）。多数 Docker 用户会使用由其他组织创建的镜像，遵循以下准则有助于避免在不同对象之间意外重复标签，尤其当你计划将标签用于自动化时：

- 第三方工具的作者应当在每个标签键前加上自己拥有域名的反向 DNS 前缀，例如 `com.example.some-label`。

- 未经域名所有者许可，请勿在标签键中使用该域名。

- `com.docker.*`、`io.docker.*` 和 `org.dockerproject.*` 命名空间由 Docker 保留用于内部用途。

- 标签键应以小写字母开头和结尾，且仅包含小写字母数字、点（`.`）和连字符（`-`）。不允许连续的点或连字符。

- 点（`.`）用于分隔命名空间“字段”。无命名空间的标签键保留给 CLI 使用，便于用户在交互式场景中以更短的、便于输入的字符串为 Docker 对象打标签。

上述准则当前并非强制执行，特定用例可能还需遵循其他约束。

### 值的建议

标签的值可以是任何可序列化为字符串的数据类型，包括（但不限于）JSON、XML、CSV 或 YAML。唯一的要求是：先将该值序列化为字符串，序列化方式取决于其数据结构类型。例如，要把 JSON 序列化为字符串，可以使用 JavaScript 的 `JSON.stringify()` 方法。

由于 Docker 不会反序列化标签值，因此在按标签值进行查询或过滤时，不能将 JSON 或 XML 文档当作嵌套结构来处理，除非你在第三方工具中自行实现相关能力。

## 在对象上管理标签

每种支持标签的对象类型，都提供了添加、管理以及结合该对象场景使用标签的机制。下面的链接是了解如何在部署中使用标签的良好起点：

镜像、容器、本地守护进程、卷与网络上的标签在对象整个生命周期内是静态的。如需修改这些标签，你必须重新创建对象。Swarm 节点与服务上的标签则可以动态更新。

 - 镜像与容器

  - [为镜像添加标签](/reference/dockerfile.md#label)
  - [在运行时覆盖容器的标签](/reference/cli/docker/container/run.md#label)
  - [查看镜像或容器上的标签](/reference/cli/docker/inspect.md)
  - [按标签筛选镜像](/reference/cli/docker/image/ls.md#filter)
  - [按标签筛选容器](/reference/cli/docker/container/ls.md#filter)

 - 本地 Docker 守护进程

  - [在运行时为 Docker 守护进程添加标签](/reference/cli/dockerd.md)
  - [查看 Docker 守护进程的标签](/reference/cli/docker/system/info.md)

 - 卷

  - [为卷添加标签](/reference/cli/docker/volume/create.md)
  - [查看卷的标签](/reference/cli/docker/volume/inspect.md)
  - [按标签筛选卷](/reference/cli/docker/volume/ls.md#filter)

 - 网络

  - [为网络添加标签](/reference/cli/docker/network/create.md)
  - [查看网络的标签](/reference/cli/docker/network/inspect.md)
  - [按标签筛选网络](/reference/cli/docker/network/ls.md#filter)

 - Swarm 节点

  - [添加或更新 Swarm 节点的标签](/reference/cli/docker/node/update.md#label-add)
  - [查看 Swarm 节点的标签](/reference/cli/docker/node/inspect.md)
  - [按标签筛选 Swarm 节点](/reference/cli/docker/node/ls.md#filter)

 - Swarm 服务
  - [创建 Swarm 服务时添加标签](/reference/cli/docker/service/create.md#label)
  - [更新 Swarm 服务的标签](/reference/cli/docker/service/update.md)
  - [查看 Swarm 服务的标签](/reference/cli/docker/service/inspect.md)
  - [按标签筛选 Swarm 服务](/reference/cli/docker/service/ls.md#filter)
