---
description: Docker Scout 如何处理镜像元数据
keywords: |
  scout, scanning, supply chain, security, purl, sbom, provenance, environment,
  materials, config, ports, labels, os, registry, timestamp, digest, layers,
  architecture, license, dependencies, base image
title: Docker Scout 中的数据采集与存储
aliases:
  /scout/data-handling/
---

Docker Scout 的镜像分析通过收集你所分析的容器镜像的元数据来完成。这些元数据会存储在 Docker Scout 平台上。

## 数据传输

本节介绍 Docker Scout 会收集并发送到平台的数据。

### 镜像元数据

Docker Scout 会收集以下镜像元数据：

- 镜像创建时间戳
- 镜像摘要（digest）
- 镜像暴露的端口
- 环境变量名称与取值
- 镜像标签（label）的名称与取值
- 镜像层（layer）的顺序
- 硬件架构
- 操作系统类型与版本
- 仓库（registry）的 URL 与类型

镜像在构建并推送到仓库时，会为其每一层生成摘要。摘要是层内容的 SHA256 值。
Docker Scout 不会创建这些摘要；它们来源于镜像清单（manifest）。

这些摘要会与你的私有镜像以及 Docker 的公共镜像数据库进行比对，以识别哪些镜像共享相同的层。
与当前分析的镜像共享层数最多的镜像会被视为其基础镜像匹配项。

### SBOM 元数据

软件物料清单（SBOM）元数据用于将软件包类型与版本和漏洞数据进行匹配，从而推断镜像是否受影响。
当 Docker Scout 平台从安全通告中接收到新的 CVE 或其他风险（例如泄露的密钥）时，
会将这些信息与 SBOM 进行交叉比对；一旦匹配，Docker Scout 会在展示其数据的界面中显示结果，
例如 Docker Scout 控制台和 Docker Desktop。

Docker Scout 会收集以下 SBOM 元数据：

- 软件包 URL（PURL）
- 软件包作者与描述
- 许可证 ID
- 软件包名称与命名空间
- 软件包的方案与大小
- 软件包类型与版本
- 软件包在镜像内的文件路径
- 直接依赖的类型
- 软件包总数

Docker Scout 中的 PURL 遵循
[purl-spec](https://github.com/package-url/purl-spec) 规范。软件包信息来源于镜像内容，
包括操作系统层面的程序与软件包，以及应用层的软件包（例如 Maven、npm 等）。

### 运行环境元数据

如果你通过
[Sysdig 集成](/manuals/scout/integrations/environment/sysdig.md)
将 Docker Scout 接入运行环境，Docker Scout 会收集以下部署相关的数据点：

- Kubernetes 命名空间
- 工作负载名称
- 工作负载类型（例如 DaemonSet）

### 本地分析

对于在开发者机器上进行的本地镜像分析，Docker Scout 仅会传输 PURL 与层摘要。
这些数据不会在 Docker Scout 平台上持久化存储，只用于执行分析。

### 溯源（Provenance）

对于带有[溯源证明（provenance attestation）](/manuals/build/metadata/attestations/slsa-provenance.md)的镜像，
Docker Scout 在 SBOM 之外还会存储以下数据：

- 材料（Materials）
- 基础镜像
- VCS 信息
- Dockerfile

## 数据存储

为提供 Docker Scout 服务，数据存储在以下位置：

- 位于美国东部的 Amazon Web Services（AWS）
- 位于美国东部的 Google Cloud Platform（GCP）

数据的使用遵循
[docker.com/legal](https://www.docker.com/legal/) 中的流程说明，
以提供 Docker Scout 的核心能力。
