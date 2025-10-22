---
title: 证明
description: 查看每个 Docker 加固镜像包含的完整签名证明集，如 SBOM、VEX、构建来源和扫描结果。
keywords: 容器镜像证明, 签名 sbom, 构建来源, slsa 合规性, vex 文档
---

Docker 加固镜像（DHIs）包含全面的、经过签名的安全证明，用于验证镜像的构建过程、内容和安全状态。这些证明是安全软件供应链实践的核心部分，帮助用户验证镜像是否可信且符合策略。

## 什么是证明？

证明是一种经过签名的声明，提供关于镜像的可验证信息，例如它是如何构建的、内部包含什么以及通过了哪些安全检查。证明通常使用 Sigstore 工具（如 Cosign）进行签名，使其具有防篡改特性并可进行加密验证。

证明遵循标准化格式（如 [in-toto](https://in-toto.io/)、[CycloneDX](https://cyclonedx.org/) 和 [SLSA](https://slsa.dev/)），并作为符合 OCI 标准的元数据附加到镜像上。证明可以在镜像构建过程中自动生成，也可以手动添加以记录额外的测试、扫描结果或自定义来源。

## 为什么证明很重要？

证明通过以下方式提供对软件供应链的关键可见性：

- 记录镜像中包含的内容（例如，SBOM）
- 验证其构建方式（例如，构建来源）
- 捕获其通过或失败的安全扫描（例如，CVE 报告、秘密扫描、测试结果）
- 帮助组织执行合规性和安全策略
- 支持运行时信任决策和 CI/CD 策略门控

它们对于满足 SLSA 等行业标准至关重要，并通过使构建和安全数据透明且可验证，帮助团队降低供应链攻击的风险。

## Docker 加固镜像如何使用证明

所有 DHI 都使用 [SLSA 构建级别 3](https://slsa.dev/spec/latest/levels) 实践构建，每个镜像变体都发布有一整套签名证明。这些证明允许用户：

- 验证镜像是在安全环境中从可信源构建的
- 查看多种格式的 SBOM 以了解组件级详细信息
- 审查扫描结果以检查漏洞或嵌入的秘密
- 确认每个镜像的构建和部署历史

证明会自动发布并与您的 Docker Hub 组织中的每个镜像 DHI 关联。可以使用 [Docker Scout](../how-to/verify.md) 或 [Cosign](https://docs.sigstore.dev/cosign/overview) 等工具进行检查，并可被 CI/CD 工具或安全平台消费。

## 可用的证明

虽然每个 DHI 变体都包含一组证明，但证明可能因镜像变体而异。例如，一些镜像可能包含 STIG 扫描证明。下表是可能包含在 DHI 中的所有证明的全面列表。要查看特定镜像变体可用的证明，您可以在 Docker Hub 中 [查看镜像变体详细信息](../how-to/explore.md#view-image-variant-details)。

| 证明类型                   | 描述                                                                                                                                                                                                                             | 谓词类型 URI                                         |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------|
| CycloneDX SBOM             | [CycloneDX](https://cyclonedx.org/) 格式的软件物料清单，列出组件、库和版本。                                                                                                                                                     | `https://cyclonedx.org/bom/v1.5`                   |
| STIG 扫描                   | STIG 扫描的结果，输出为 HTML 和 XCCDF 格式。                                                                                                                                                                                          | `https://docker.com/dhi/stig/v0.1`                 |
| CVE（In-Toto 格式）         | 影响镜像组件的已知漏洞（CVE）列表，基于包和发行版扫描。                                                                                                                                                                          | `https://in-toto.io/attestation/vulns/v0.1`        |
| VEX                        | [漏洞可利用性交换 (VEX)](https://openvex.dev/) 文档，标识不适用于镜像的漏洞并解释原因（例如，不可达或不存在）。                                                                                                                     | `https://openvex.dev/ns/v0.2.0`                    |
| Scout 健康评分              | 来自 Docker Scout 的签名证明，总结镜像的整体安全和质量状态。                                                                                                                                                                       | `https://scout.docker.com/health/v0.1`             |
| Scout 来源                  | 由 Docker Scout 生成的来源元数据，包括源 Git 提交、构建参数和环境详细信息。                                                                                                                                                           | `https://scout.docker.com/provenance/v0.1`         |
| Scout SBOM                 | 由 Docker Scout 生成并签名的 SBOM，包括额外的 Docker 特定元数据。                                                                                                                                                                     | `https://scout.docker.com/sbom/v0.1`               |
| 秘密扫描                   | 对意外包含的秘密（如凭据、令牌或私钥）的扫描结果。                                                                                                                                                                               | `https://scout.docker.com/secrets/v0.1`            |
| 测试                       | 对镜像运行的自动化测试记录，如功能检查或验证脚本。                                                                                                                                                                               | `https://scout.docker.com/tests/v0.1`              |
| 病毒扫描                   | 对镜像层执行的防病毒扫描结果。                                                                                                                                                                                                   | `https://scout.docker.com/virus/v0.1`              |
| CVE（Scout 格式）           | 由 Docker Scout 生成的漏洞报告，列出已知 CVE 和严重性数据。                                                                                                                                                                          | `https://scout.docker.com/vulnerabilities/v0.1`    |
| SLSA 来源                   | 标准的 [SLSA](https://slsa.dev/) 来源声明，描述镜像的构建方式，包括构建工具、参数和源代码。                                                                                                                                         | `https://slsa.dev/provenance/v0.2`                 |
| SLSA 验证摘要               | 指示镜像符合 SLSA 要求的摘要证明。                                                                                                                                                                                                 | `https://slsa.dev/verification_summary/v1`         |
| SPDX SBOM                  | [SPDX](https://spdx.dev/) 格式的 SBOM，在开源生态系统中广泛采用。                                                                                                                                                                 | `https://spdx.dev/Document`                        |
| FIPS 合规性                 | 验证镜像使用 FIPS 140 验证的加密模块的证明。                                                                                                                                                                                       | `https://docker.com/dhi/fips/v0.1`                 |

## 查看和验证证明

要查看和验证镜像的证明，请参阅 [验证 Docker 加固镜像](../how-to/verify.md)。

## 添加您自己的证明
  
除了 Docker 加固镜像提供的全面证明外，您在构建派生镜像时可以添加自己的签名证明。如果您在 DHI 之上构建新应用程序并希望保持软件供应链的透明度、可追溯性和信任度，这尤其有用。

通过附加 SBOM、构建来源或自定义元数据等证明，您可以满足合规性要求、通过安全审计，并支持 Docker Scout 等策略评估工具。

然后，这些证明可以使用 Cosign 或 Docker Scout 等工具在下游进行验证。

要了解如何在构建过程中附加自定义证明，请参阅 [构建证明](/manuals/build/metadata/attestations.md)。
