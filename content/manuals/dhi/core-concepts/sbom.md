---
title: 软件物料清单 (SBOMs)
linktitle: SBOMs
description: 了解什么是 SBOM、它们的重要性，以及 Docker 加固镜像如何包含已签名的 SBOM 来支持透明度和合规性。
keywords: sbom docker 镜像, 软件物料清单, 已签名 sbom, 容器 sbom 验证, sbom 合规性
---

## 什么是 SBOM？

SBOM 是一个详细的清单，列出了构建软件应用程序时使用的所有组件、库和依赖项。它通过记录每个组件的版本、来源以及与其他组件的关系，为软件供应链提供了透明度。可以将其视为软件的"配方"，详细说明了每个成分以及它们如何组合在一起。

SBOM 中包含的用于描述软件制品的元数据可能包括：

- 制品名称
- 版本
- 许可证类型
- 作者
- 唯一包标识符

## 为什么 SBOM 很重要？

在当今的软件环境中，应用程序通常包含来自各种来源的众多组件，包括开源库、第三方服务和专有代码。这种复杂性可能会掩盖对潜在漏洞的可见性，并使合规性工作复杂化。SBOM 通过提供应用程序内所有组件的详细清单来解决这些挑战。

SBOM 的重要性体现在以下几个关键因素中：

- **增强透明度**：SBOM 提供了构成应用程序的所有组件的全面视图，使组织能够识别和评估与第三方库和依赖项相关的风险。

- **主动漏洞管理**：通过维护最新的 SBOM，组织可以快速识别和解决软件组件中的漏洞，减少潜在攻击的暴露窗口。

- **法规合规性**：许多法规和行业标准现在要求组织对其使用的软件组件保持控制。SBOM 通过提供清晰可访问的记录来促进合规性。

- **改进事件响应**：在发生安全漏洞时，SBOM 使组织能够快速识别受影响的组件并采取适当措施，从而最大限度地减少潜在损害。

## Docker 加固镜像的 SBOM

Docker 加固镜像内置了 SBOM，确保镜像中的每个组件都有文档记录且可验证。这些 SBOM 经过加密签名，提供了镜像内容的防篡改记录。这种集成简化了审计工作，并增强了软件供应链中的信任度。

## 查看 Docker 加固镜像中的 SBOM

要查看 Docker 加固镜像的 SBOM，您可以使用 `docker scout sbom` 命令。将 `<image-name>:<tag>` 替换为镜像名称和标签。

```console
$ docker scout sbom <image-name>:<tag>
```

## 验证 Docker 加固镜像的 SBOM

由于 Docker 加固镜像附带已签名的 SBOM，您可以使用 Docker Scout 来验证附加到镜像的 SBOM 的真实性和完整性。这确保了 SBOM 未被篡改，并且镜像内容是可信的。

要使用 Docker Scout 验证 Docker 加固镜像的 SBOM，请使用以下命令：

```console
$ docker scout attest get <image-name>:<tag> \
   --predicate-type https://scout.docker.com/sbom/v0.1 --verify --platform <platform>
```

例如，要验证 `dhi/node:20.19-debian12-fips-20250701182639` 镜像的 SBOM 证明：

```console
$ docker scout attest get docs/dhi-node:20.19-debian12-fips-20250701182639 \
   --predicate-type https://scout.docker.com/sbom/v0.1 --verify --platform linux/amd64
```

## 相关资源

有关 SBOM 证明和 Docker Build 的更多详细信息，请参阅 [SBOM 证明](/build/metadata/attestations/sbom/)。

要了解更多关于 Docker Scout 和使用 SBOM 的信息，请参阅 [Docker Scout SBOMs](../../scout/how-tos/view-create-sboms.md)。