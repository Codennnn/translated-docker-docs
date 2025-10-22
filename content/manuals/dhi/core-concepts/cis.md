---
title: CIS 基准
description: 了解 Docker 加固镜像如何符合 CIS Docker 基准，帮助组织加固容器镜像以实现安全部署。
keywords: docker cis 基准, cis docker 合规性, cis docker 镜像, docker 加固镜像, 安全容器镜像
---

## 什么是 CIS Docker 基准？

[CIS Docker 基准](https://www.cisecurity.org/benchmark/docker)是全球公认的 CIS 基准的一部分，由[互联网安全中心 (CIS)](https://www.cisecurity.org/)开发。它定义了 Docker 容器生态系统所有方面的推荐安全配置，包括容器主机、Docker 守护进程、容器镜像和容器运行时。

## 为什么 CIS 基准合规性很重要

遵循 CIS Docker 基准有助于组织：

- 通过广泛认可的加固指导降低安全风险。
- 满足引用 CIS 控制的法规或合同要求。
- 在团队间标准化镜像和 Dockerfile 实践。
- 通过基于公共标准的配置决策展示审计准备情况。

## Docker 加固镜像如何符合 CIS 基准

Docker 加固镜像（DHIs）在设计时考虑了安全性，并经过验证符合最新 CIS Docker 基准（v1.8.0）中适用于容器镜像和 Dockerfile 配置范围的相关控制。

符合 CIS 标准的 DHI 符合第 4 节中的所有控制，唯一例外是要求 Docker 内容信任（DCT）的控制，该控制已被 [Docker 正式淘汰](https://www.docker.com/blog/retiring-docker-content-trust/)。相反，DHI 使用 Cosign 进行[签名](/manuals/dhi/core-concepts/signatures.md)，提供了更高级别的真实性和完整性。通过从符合 CIS 标准的 DHI 开始，团队可以更快速、更自信地采用基准中的镜像级最佳实践。

> [!NOTE]
>
> CIS Docker 基准还包括主机、守护进程和运行时的控制。符合 CIS 标准的 DHI 仅解决镜像和 Dockerfile 范围（第 4 节）。整体合规性仍取决于您如何配置和操作更广泛的环境。

## 识别符合 CIS 标准的镜像

符合 CIS 标准的镜像在 Docker 加固镜像目录中标记为 **CIS**。要查找它们，请[探索镜像](../how-to/explore.md)并在单个列表中查找 **CIS** 标记。

## 获取基准

直接从 CIS 下载最新的 CIS Docker 基准：
https://www.cisecurity.org/benchmark/docker
