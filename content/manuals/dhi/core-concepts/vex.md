---
title: 漏洞可利用性交换 (VEX)
linktitle: VEX
description: 了解 VEX 如何通过识别 Docker 加固镜像中实际可利用的漏洞，帮助您优先处理真实风险。
keywords: vex 容器安全, 漏洞可利用性, 过滤误报, docker scout vex, cve 优先级排序
---

## 什么是 VEX？

漏洞可利用性交换（Vulnerability Exploitability eXchange, VEX）是由美国网络安全和基础设施安全局（CISA）开发的标准化框架，用于记录软件组件中漏洞的可利用性。

与传统 CVE（通用漏洞披露）数据库不同，VEX 提供基于上下文的评估，明确指出某个漏洞在特定环境中是否可利用。这种方法通过区分可利用的漏洞与特定使用场景无关的漏洞，帮助组织优先安排修复工作。

## VEX 的重要性

VEX 通过以下方式增强传统漏洞管理：

- **减少误报**：通过提供特定于上下文的评估，VEX 有助于过滤掉在特定环境中不构成威胁的漏洞
- **优先修复**：组织可以集中资源处理在其特定环境中可利用的漏洞，提高漏洞管理效率
- **增强合规性**：VEX 报告提供详细信息，有助于满足监管要求和内部安全标准

这种方法在复杂环境中尤其有益，因为传统基于 CVE 的评估可能导致不必要的修复工作。

## Docker 加固镜像如何集成 VEX

为增强漏洞管理能力，Docker 加固镜像（DHI）集成了 VEX 报告，提供已知漏洞的上下文特定评估。

这种集成使您能够：

- **评估可利用性**：确定镜像组件中已知漏洞在特定环境中是否可利用
- **优先处理**：将修复工作重点放在实际构成风险的漏洞上，优化资源分配
- **简化审计**：利用 VEX 报告提供的详细信息，简化合规审计和报告流程

通过将 DHI 的安全特性与 VEX 的上下文洞察相结合，组织可以实现更有效、更高效的漏洞管理方法。

## 使用 VEX 过滤已知不可利用的 CVE

使用 Docker Scout 时，VEX 声明会自动应用，无需手动配置。

如需手动获取支持 VEX 的工具所需的证明文件：

```console
$ docker scout vex get <your-namespace>/dhi-<image>:<tag> --output vex.json
```

> [!NOTE]
>
> `docker scout vex get` 命令需要 [Docker Scout CLI](https://github.com/docker/scout-cli/) 版本 1.18.3 或更高版本。

例如：

```console
$ docker scout vex get docs/dhi-python:3.13 --output vex.json
```

这将创建一个包含指定镜像 VEX 声明的 `vex.json` 文件。然后，您可以将此文件与支持 VEX 的工具一起使用，以过滤掉已知不可利用的 CVE。

例如，使用 Grype 和 Trivy 时，您可以使用 `--vex` 标志在扫描过程中应用 VEX 声明：

```console
$ grype <your-namespace>/dhi-<image>:<tag> --vex vex.json
```