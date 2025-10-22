---
title: 无缝集成
description: 了解 Docker 加固镜像如何在不改变现有研发与部署流程的前提下，为你注入更高等级的安全能力。
description_short: 看 DHI 如何与 CI/CD、漏洞扫描、容器仓库等现有工具链无缝协同。
keywords: 容器 CI CD, 漏洞扫描, SLSA 构建三级, 签名 SBOM, OCI 兼容仓库
---

Docker 加固镜像（DHI）专为“零改造”而生：无需打乱现有流程，就能让安全能力一键落地。

## 在 Docker Hub 上即点即看

组织[开通服务](https://www.docker.com/products/hardened-images/#getstarted)后，开发与安全团队可直接在 Docker Hub 内：

- 浏览全量镜像及语言/框架变体
- 查看支持的发行版
- 对比 dev 与 runtime 版本差异

每个仓库都附带标签清单、基础镜像配置及专属文档，助你秒选最适合的变体。

## CI/CD 即插即用

DHI 与普通基础镜像用法完全一致，任何基于 Dockerfile 的流水线都能直接替换。GitHub Actions、GitLab CI/CD、Jenkins、CircleCI 等主流平台无需额外配置即可接入。

## 专为 DevSecOps 生态打造

DHI 已提前对接扫描工具、镜像仓库、CI/CD 及策略引擎。Docker 与多家生态伙伴深度合作，确保镜像开箱即用，扫描结果、元数据验证与合规洞察可直接回流你的流水线。

所有 DHI 均内置：

- 已签名的软件物料清单（SBOM）
- CVE 漏洞数据
- VEX 漏洞可利用性交换文档
- SLSA 构建三级来源证明

结构化且已签名的元数据，可被策略引擎或合规仪表盘直接消费，审计零负担。

## 想推到哪里就推到哪里

DHI 先同步至你在 Docker Hub 的组织命名空间；随后可一键推送至任意 OCI 兼容仓库：

- Amazon ECR
- Google Artifact Registry
- GitHub Container Registry
- Azure Container Registry
- Harbor
- JFrog Artifactory
- 或其他私有/公有 OCI 仓库

多仓库分发，策略与构建系统零中断。

## 一句话总结

DHI 与你现有工具同频共振：

- 标准 Docker 命令与流水线无需改造
- 主流扫描器与仓库全部适配
- 安全元数据直接对接合规体系

升级安全，不打扰工程效率。
