---
title: 关于
description: 了解 Docker 硬化镜像，包括其用途、构建和测试方式，以及安全方面的共同责任模型。
weight: 5
params:
  grid_about:
    - title: 什么是硬化镜像？为什么要使用它们？
      description: 了解硬化镜像的定义，Docker 硬化镜像的构建方式，它们与典型基础镜像和应用镜像的区别，以及使用它们的原因。
      icon: info
      link: /dhi/about/what/
    - title: 镜像测试
      description: 了解 Docker 硬化镜像如何自动进行标准合规性、功能性和安全性测试。
      icon: science
      link: /dhi/about/test/
    - title: 责任概述
      description: 了解在将 Docker 硬化镜像作为安全软件供应链一部分使用时，Docker 和用户各自承担的责任。
      icon: group
      link: /dhi/about/responsibility/
    - title: 镜像类型
      description: 了解 Docker 硬化镜像目录中提供的不同镜像类型、发行版和变体。
      icon: view_module
      link: /dhi/about/available/
---

Docker 硬化镜像（Docker Hardened Images，简称 DHIs）专为现代软件供应链中的安全、合规性和可靠性而构建。本章节将介绍这些镜像与标准基础镜像和应用镜像的区别，它们的构建和测试方式，以及 Docker 和用户在保护容器化工作负载安全方面如何分担责任。

## 了解 Docker 硬化镜像

{{< grid
  items="grid_about"
>}}