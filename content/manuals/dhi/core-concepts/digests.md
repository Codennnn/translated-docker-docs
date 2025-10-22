---
title: 镜像摘要（digest）
description: 了解 Docker 加固镜像如何通过不可变摘要、签名元数据和多平台清单，保障软件供应链每一阶段的安全与一致性。
keywords: docker 镜像摘要, 按摘要拉取镜像, 不可变容器镜像, 安全镜像引用, 多平台清单
---

## 什么是镜像摘要？

镜像摘要（digest）是 Docker 镜像内容的唯一加密标识（SHA-256 哈希）。与可被重复指向或随意改动的标签不同，摘要一旦生成就不可变，保证每次拉取的都是**完全相同**的镜像，从而在不同环境与部署之间实现绝对一致。

以 `nginx:latest` 为例，其摘要形如：

```text
sha256:94a00394bc5a8ef503fb59db0a7d0ae9e1110866e8aee8ba40cd864cea69ea1a
```

只要镜像内容有任何改动，摘要就会变化，因此它能精确定位到某一特定版本。

## 为什么要用摘要？

- **不可变**：摘要绑定的是镜像“指纹”，内容不变，指纹不变；用摘要拉取，永远拿到同一份镜像。
- **防篡改**：哪怕只改一个字节，摘要也会截然不同，可阻断供应链投毒。
- **一致性**：开发、测试、生产各环境都引用同一摘要，彻底消除“我这边明明没问题”的差异。

## Docker 加固镜像与摘要

通过摘要引用 DHI，可确保应用始终运行在**唯一且安全**的镜像版本上，为合规与审计提供强一致性的基准。

## 查看镜像摘要

### 使用 Docker CLI

```console
$ docker buildx imagetools inspect <镜像名>:<标签>
```

### 使用 Docker Hub 界面

1. 登录 [Docker Hub](https://hub.docker.com/)。
2. 进入组织命名空间，打开对应的 DHI 仓库。
3. 切到 **Tags** 标签页，即可看到每条标签旁的 **Digest** 字段（SHA-256 值）。

## 按摘要拉取镜像

```console
$ docker pull <镜像名>@sha256:<摘要>
```

示例：拉取 `docs/dhi-python:3.13` 的固定版本

```console
$ docker pull docs/dhi-python@sha256:94a00394bc5a8ef503fb59db0a7d0ae9e1110866e8aee8ba40cd864cea69ea1a
```

## 多平台镜像与清单

DHI 均以**多平台镜像**形式发布：同一标签（如 `docs/dhi-python:3.13`）背后对应一张**清单列表（manifest list）**，里面再指向不同架构（`linux/amd64`、`linux/arm64` 等）的具体镜像摘要。

执行：

```console
$ docker buildx imagetools inspect docs/dhi-python:3.13
```

可看到类似输出：

```text
Name:      docs/dhi-python:3.13
MediaType: application/vnd.docker.distribution.manifest.list.v2+json
Digest:    sha256:6e05...d231   ← 清单列表摘要

Manifests:
  Name:        docs/dhi-python:3.13@sha256:94a0...ea1a
  Platform:    linux/amd64
  ...

  Name:        docs/dhi-python:3.13@sha256:7f1d...bc43
  Platform:    linux/arm64
  ...
```

- **清单列表摘要** 代表整个多平台镜像。  
- **每条平台记录** 都有自己的摘要，指向对应架构的实体镜像。

### 为什么这很重要

- **可复现**：跨架构构建或运行时，标签会自动解析到当前平台对应的摘要，保证行为一致。  
- **可验证**：也能单独拉取某平台摘要，验证镜像内容与清单列表无关，进一步降低风险。  
- **策略强制**：在使用 Docker Scout 做基于摘要的策略校验时，每个平台变体都会按自身摘要独立评估。