---
title: STIG
description: 了解 Docker 加固镜像如何提供经过 STIG 强化的容器镜像，并附带可验证的安全扫描证明，满足政府和企业的合规要求。
keywords: docker stig, stig 强化镜像, stig 指南, openscap docker, 安全容器镜像
---

## 什么是 STIG？

[安全技术实施指南 (Security Technical Implementation Guides, STIGs)](https://public.cyber.mil/stigs/) 是由美国国防信息系统局（DISA）发布的配置标准。这些指南定义了在美国国防部（DoD）环境中使用的操作系统、应用程序、数据库和其他技术的安全要求。

STIG 有助于确保系统以安全且一致的方式进行配置，从而减少漏洞。它们通常基于更广泛的要求，如国防部的通用操作系统安全要求指南（GPOS SRG）。

## STIG 指南的重要性

遵循 STIG 指南对于与美国政府系统合作或提供支持的组织至关重要。这体现了与 DoD 安全标准的一致性，并有助于：

- **加速 DoD 系统的运行授权 (ATO) 流程**
- **降低配置错误和可利用弱点的风险**
- **通过标准化基线简化审计和报告**

即使在联邦环境之外，注重安全的组织也将 STIG 作为强化系统配置的基准。

STIG 源自更广泛的 NIST 指南，特别是 [NIST 特别出版物 800-53](https://csrc.nist.gov/publications/sp800)，该出版物定义了联邦系统的安全和隐私控制目录。追求 800-53 或相关框架（如 FedRAMP）合规性的组织可以使用 STIG 作为实施指南，帮助满足适用的控制要求。

## Docker 加固镜像如何帮助应用 STIG 指南

Docker 加固镜像 (DHI) 包含 STIG 变体，这些变体针对自定义的基于 STIG 的配置文件进行扫描，并包含签名的 STIG 扫描证明。这些证明可以支持审计和合规性报告。

Docker 基于 GPOS SRG 和 DoD 容器强化过程指南为镜像创建自定义的基于 STIG 的配置文件。由于 DISA 尚未发布专门针对容器的 STIG，这些配置文件有助于以一致、可审查的方式将类似 STIG 的指南应用于容器环境，并旨在减少容器镜像中常见的误报。

## 识别包含 STIG 扫描结果的镜像

包含 STIG 扫描结果的 Docker 加固镜像在 Docker 加固镜像目录中标记为 **STIG**。

要查找包含 STIG 镜像变体的 DHI 仓库，请[浏览镜像](../how-to/explore.md)并：

- 在目录页面上使用 **STIG** 过滤器
- 在单个镜像列表中查找 **STIG** 标签

要在仓库内查找 STIG 镜像变体，请转到仓库中的 **Tags** 标签页，并在 **Compliance** 列中查找标记为 **STIG** 的镜像。

## 查看和验证 STIG 扫描结果

Docker 为每个经过 STIG 强化的镜像提供签名的 [STIG 扫描证明](../core-concepts/attestations.md)。这些证明包括：

- 扫描结果摘要，包括通过、失败和不适用检查的数量
- 使用的 STIG 配置文件的名称和版本
- 完整的 HTML 和 XCCDF (XML) 格式输出

### 查看 STIG 扫描证明

您可以使用 Docker Scout CLI 检索和检查 STIG 扫描证明：

```console
$ docker scout attest get \
  --predicate-type https://docker.com/dhi/stig/v0.1 \
  --verify \
  --predicate \
  <your-namespace>/dhi-<image>:<tag>
```

### 提取 HTML 报告

要提取并查看人类可读的 HTML 报告：

```console
$ docker scout attest get <your-namespace>/dhi-<image>:<tag> \
  --predicate-type https://docker.com/dhi/stig/v0.1 \
  --verify \
  --predicate \
  | jq -r '.[0].output[] | select(.format == "html").content | @base64d' > stig_report.html
```

### 提取 XCCDF 报告

要提取 XML (XCCDF) 报告以便与其他工具集成：

```console
$ docker scout attest get <your-namespace>/dhi-<image>:<tag> \
  --predicate-type https://docker.com/dhi/stig/v0.1 \
  --verify \
  --predicate \
  | jq -r '.[0].output[] | select(.format == "xccdf").content | @base64d' > stig_report.xml
```

### 查看 STIG 扫描摘要

要仅查看扫描摘要而不包含完整报告：

```console
$ docker scout attest get <your-namespace>/dhi-<image>:<tag> \
  --predicate-type https://docker.com/dhi/stig/v0.1 \
  --verify \
  --predicate \
  | jq -r '.[0] | del(.output)'
```


