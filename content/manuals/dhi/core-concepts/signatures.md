---
title: 代码签名
description: 了解 Docker 加固镜像如何使用 Cosign 进行加密签名，以验证真实性、完整性和安全来源。
keywords: 容器镜像签名, cosign docker 镜像, 验证镜像签名, 已签名容器镜像, sigstore cosign
---

## 什么是代码签名？

代码签名是向软件制品（如 Docker 镜像）应用加密签名的过程，用于验证其完整性和真实性。通过对镜像进行签名，您可以确保镜像自签名以来未被篡改，并且源自可信来源。

在 Docker 加固镜像 (DHIs) 的上下文中，代码签名通过 [Cosign](https://docs.sigstore.dev/) 实现，这是由 Sigstore 项目开发的工具。Cosign 支持容器镜像的安全和可验证签名，增强了软件供应链中的信任和安全性。

## 为什么代码签名很重要？

代码签名在现代软件开发和网络安全中扮演着关键角色：

- **真实性**：验证镜像由可信来源创建
- **完整性**：确保镜像自签名以来未被篡改
- **合规性**：帮助满足法规和组织安全要求

## Docker 加固镜像的代码签名

每个 DHI 都使用 Cosign 进行加密签名，确保镜像未被篡改且源自可信来源。

## 为什么要对自己的镜像进行签名？

Docker 加固镜像已由 Docker 签名以证明其来源和完整性，但如果您构建的应用程序镜像扩展或使用 DHIs 作为基础镜像，您也应该对自己的镜像进行签名。

通过对自己的镜像进行签名，您可以：

- **证明镜像由您的团队或流水线构建**
- **确保构建在推送后未被篡改**
- **支持软件供应链框架（如 SLSA）**
- **在部署工作流中启用镜像验证**

这在频繁构建和推送镜像的 CI/CD 环境中尤其重要，或者在任何需要可审计镜像来源的场景中。

## 如何查看和使用代码签名

### 查看签名

您可以使用 Docker Scout 或 Cosign 来验证 Docker 加固镜像是否已签名且可信。

要列出附加到镜像的所有证明（包括签名元数据），请使用以下命令：

```console
$ docker scout attest list <image-name>:<tag> --platform <platform>
```

要验证特定的已签名证明（例如 SBOM、VEX、来源）：

```console
$ docker scout attest get \
  --predicate-type <predicate-uri> \
  --verify \
  <image-name>:<tag> --platform <platform>
```

例如：

```console
$ docker scout attest get \
  --predicate-type https://openvex.dev/ns/v0.2.0 \
  --verify \
  docs/dhi-python:3.13 --platform linux/amd64
```

如果验证有效，Docker Scout 将确认签名并显示签名有效载荷，以及用于验证镜像的等效 Cosign 命令。

### 对镜像进行签名

要对 Docker 镜像进行签名，请使用 [Cosign](https://docs.sigstore.dev/)。将 `<image-name>:<tag>` 替换为镜像名称和标签。

```console
$ cosign sign <image-name>:<tag>
```

此命令将提示您通过 OIDC 提供商（如 GitHub、Google 或 Microsoft）进行身份验证。成功验证后，Cosign 将生成短期证书并对镜像进行签名。签名将存储在透明日志中，并与注册表中的镜像关联。