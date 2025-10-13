---
title: 构建镜像
weight: 20
keywords: build images, Dockerfile, layers, tag, push, cache, multi-stage
description: |
  学习如何从 Dockerfile 构建 Docker 镜像。你将理解 Dockerfile 的结构、
  如何构建镜像，以及如何定制构建过程。
summary: |
  构建容器镜像既是技术活也是一门艺术。你既要让镜像足够小、足够聚焦，
  以提升安全性与部署效率；也要在诸如缓存命中率等权衡中做出正确选择。
  本系列将带你深入理解镜像的工作原理、构建方式与最佳实践。
layout: series
params:
  skill: 初学者
  time: 25 分钟
  prereq: 无
---

## 关于本系列

学习如何构建面向生产环境的精简高效镜像，它们能显著降低开销、提升部署效率，
是现代生产环境中不可或缺的基础能力。

## 你将学到什么

- 理解镜像分层
- 编写 Dockerfile
- 构建、打标签并发布镜像
- 使用构建缓存
- 多阶段构建
