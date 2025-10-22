---
title: 验证 Docker 加固镜像
linktitle: 验证镜像
description: 使用 Docker Scout 或 cosign 验证 Docker 加固镜像的签名证明，包括 SBOM、来源和漏洞数据。
weight: 40
keywords: 验证容器镜像, docker scout attest, cosign verify, sbom 验证, 签名容器证明
---

{{< summary-bar feature_name="Docker Hardened Images" >}}

Docker 加固镜像（DHI）包含签名证明，用于验证镜像的构建过程、内容和安全状况。这些证明适用于每个镜像变体，可使用 [cosign](https://docs.sigstore.dev/) 或 Docker Scout CLI 进行验证。

Docker 用于 DHI 镜像的公钥发布在：

- https://registry.scout.docker.com/keyring/dhi/latest.pub
- https://github.com/docker-hardened-images/keyring

## 使用 Docker Scout 验证证明

您可以使用 [Docker Scout](/scout/) CLI 列出和检索 Docker 加固镜像的证明，包括镜像到您组织命名空间的镜像。

> [!NOTE]
>
> 在运行 `docker scout attest` 命令之前，请确保您本地拉取的镜像与远程镜像保持同步。您可以通过运行 `docker pull` 来实现。如果不这样做，您可能会看到 `No attestation found`。

### 为什么选择 Docker Scout 而不是直接使用 cosign？

虽然您可以手动使用 cosign 验证证明，但 Docker Scout CLI 在处理 Docker 加固镜像时提供了几个关键优势：

- **专为 DHI 设计**：Docker Scout 理解 DHI 证明的结构和镜像命名约定，因此您无需手动构建完整的镜像摘要或 URI。

- **自动平台解析**：使用 Scout 时，您可以指定平台（例如 `--platform linux/amd64`），它会自动验证正确的镜像变体。Cosign 需要您自己查找摘要。

- **人类可读的摘要**：Scout 返回证明内容的摘要（例如包计数、来源步骤），而 cosign 只返回原始签名验证输出。

- **一步验证**：`docker scout attest get` 中的 `--verify` 标志验证证明并显示等效的 cosign 命令，让您更容易理解幕后的情况。

- **与 Docker Hub 和 DHI 信任模型集成**：Docker Scout 与 Docker 的证明基础设施和公钥环紧密集成，确保兼容性并为 Docker 生态系统内的用户简化验证。

简而言之，Docker Scout 简化了验证过程并减少了人为错误的可能性，同时仍然为您提供完全可见性，并在需要时可以选择回退到 cosign。

### 列出可用证明

要列出镜像 DHI 的证明：

```console
$ docker scout attest list <your-org-namespace>/dhi-<image>:<tag> --platform <platform>
```

此命令显示所有可用的证明，包括 SBOM、来源、漏洞报告等。

### 检索特定证明

要检索特定证明，请使用 `--predicate-type` 标志和完整的谓词类型 URI：

```console
$ docker scout attest get \
  --predicate-type https://cyclonedx.org/bom/v1.5 \
  <your-org-namespace>/dhi-<image>:<tag> --platform <platform>
```

例如：

```console
$ docker scout attest get \
  --predicate-type https://cyclonedx.org/bom/v1.5 \
  docs/dhi-python:3.13 --platform linux/amd64
```

仅检索谓词主体：

```console
$ docker scout attest get \
  --predicate-type https://cyclonedx.org/bom/v1.5 \
  --predicate \
  <your-org-namespace>/dhi-<image>:<tag> --platform <platform>
```

例如：

```console
$ docker scout attest get \
  --predicate-type https://cyclonedx.org/bom/v1.5 \
  --predicate \
  docs/dhi-python:3.13 --platform linux/amd64
```

### 使用 Docker Scout 验证证明

要使用 Docker Scout 验证证明，您可以使用 `--verify` 标志：

```console
$ docker scout attest get <image-name>:<tag> \
   --predicate-type https://scout.docker.com/sbom/v0.1 --verify
```

例如，要验证 `dhi/node:20.19-debian12-fips-20250701182639` 镜像的 SBOM 证明：

```console
$ docker scout attest get docs/dhi-node:20.19-debian12-fips-20250701182639 \
   --predicate-type https://scout.docker.com/sbom/v0.1 --verify
```

#### 处理缺失的透明度日志条目

使用 `--verify` 时，您有时可能会看到如下错误：

```text
ERROR no matching signatures: signature not found in transparency log
```

发生这种情况是因为 Docker 加固镜像并不总是将证明记录在公共的 [Rekor](https://docs.sigstore.dev/logging/overview/) 透明度日志中。在证明包含私有用户信息的情况下（例如，镜像引用中的组织命名空间），将其写入 Rekor 会公开这些信息。

即使缺少 Rekor 条目，证明仍然使用 Docker 的公钥签名，并且可以通过跳过 Rekor 透明度日志检查来进行离线验证。

要跳过透明度日志检查并使用 Docker 的密钥进行验证，请使用 `--skip-tlog` 标志：

```console
$ docker scout attest get \
  --predicate-type https://cyclonedx.org/bom/v1.6 \
  <your-org-namespace>/dhi-<image>:<tag> --platform <platform> \
  --verify --skip-tlog
```

> [!NOTE]
>
> `--skip-tlog` 标志仅在 Docker Scout CLI 1.18.2 及更高版本中可用。

这等效于使用带有 `--insecure-ignore-tlog=true` 标志的 `cosign`，该标志使用 Docker 发布的公钥验证签名，但忽略透明度日志检查。

### 显示等效的 cosign 命令

使用 `--verify` 标志时，它还会打印相应的 [cosign](https://docs.sigstore.dev/) 命令来验证镜像签名：

```console
$ docker scout attest get \
  --predicate-type https://cyclonedx.org/bom/v1.5 \
  --verify \
  <your-org-namespace>/dhi-<image>:<tag> --platform <platform>
```

例如：

```console
$ docker scout attest get \
  --predicate-type https://cyclonedx.org/bom/v1.5 \
  --verify \
  docs/dhi-python:3.13 --platform linux/amd64
```

如果验证成功，Docker Scout 会打印完整的 `cosign verify` 命令。

示例输出：

```console
    v SBOM obtained from attestation, 101 packages found
    v Provenance obtained from attestation
    v cosign verify registry.scout.docker.com/docker/dhi-python@sha256:b5418da893ada6272add2268573a3d5f595b5c486fb7ec58370a93217a9785ae \
        --key https://registry.scout.docker.com/keyring/dhi/latest.pub --experimental-oci11
    ...
```

> [!IMPORTANT]
>
> 使用 cosign 时，您必须首先向 Docker Hub 注册表和 Docker Scout 注册表进行身份验证。
>
> 例如：
>
> ```console
> $ docker login
> $ docker login registry.scout.docker.com
> $ cosign verify \
>     registry.scout.docker.com/docker/dhi-python@sha256:b5418da893ada6272add2268573a3d5f595b5c486fb7ec58370a93217a9785ae \
>     --key https://registry.scout.docker.com/keyring/dhi/latest.pub --experimental-oci11
> ```

## 可用的 DHI 证明

请参阅[可用证明](../core-concepts/attestations.md#available-attestations)获取每个 DHI 可用证明的列表。

## 在 Docker Hub 上浏览证明

您还可以在[浏览镜像变体](./explore.md#view-image-variant-details)时直观地浏览证明。**证明**部分列出了每个可用证明及其：

- 类型（例如 SBOM、VEX）
- 谓词类型 URI
- 用于 `cosign` 的摘要引用

这些证明作为 Docker 加固镜像构建过程的一部分自动生成和签名。