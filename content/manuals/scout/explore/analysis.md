---
title: Docker Scout 镜像分析
description:
  Docker Scout 的镜像分析为你提供镜像组成与其包含漏洞的细粒度视图
keywords: scout, scanning, vulnerabilities, supply chain, security, analysis
aliases:
  - /scout/advanced-image-analysis/
  - /scout/image-analysis/
---

当你为某个仓库启用镜像分析后，
Docker Scout 会自动分析你推送到该仓库的新镜像。

镜像分析会提取软件物料清单（Software Bill of Materials，SBOM）
及其他镜像元数据，并与[安全通告](/manuals/scout/deep-dive/advisory-db-sources.md)中的漏洞数据进行比对评估。

如果你通过 CLI 或 Docker Desktop 以一次性任务方式运行镜像分析，
Docker Scout 不会存储任何镜像相关数据。
但如果你为容器镜像仓库启用 Docker Scout，分析完成后 Docker Scout 会保存镜像的元数据快照。
当有新的漏洞数据发布时，Docker Scout 会基于该快照重新校准分析结果，
这意味着镜像的安全状态会实时更新，而无需在每次披露新 CVE 时重新分析镜像。

Docker Hub 仓库默认支持 Docker Scout 镜像分析。
你也可以集成第三方镜像仓库与服务。更多信息请参阅
[将 Docker Scout 集成到其他系统](/manuals/scout/integrations/_index.md)。

## 在仓库中启用 Docker Scout

Docker Personal 默认包含 1 个启用 Scout 的仓库。若需要更多仓库，
可升级你的 Docker 订阅。
各订阅档位包含的启用仓库数量，见
[订阅与功能](../../subscription/details.md)。

在第三方仓库的仓库上启用镜像分析前，
需要先为你的 Docker 组织完成该仓库与 Docker Scout 的集成。
Docker Hub 默认已完成集成。更多信息参见
[容器仓库集成](/manuals/scout/integrations/_index.md#container-registries)。

> [!NOTE]
>
> 你必须在 Docker 组织中拥有 **Editor** 或 **Owner** 角色，
> 才能在仓库上启用镜像分析。

启用镜像分析：

1. 进入 Docker Scout Dashboard 的 [Repository settings](https://scout.docker.com/settings/repos)。
2. 选择需要启用的仓库。
3. 选择 **Enable image analysis**。

如果你的仓库已经包含镜像，
Docker Scout 会自动拉取并分析最新的镜像。

## 分析仓库中的镜像

要触发仓库中某个镜像的分析，请将该镜像 push 到已与 Docker Scout 集成、且已启用镜像分析的仓库。

> [!NOTE]
>
> Docker Scout 平台上的镜像分析单个镜像文件大小上限为 10 GB，除非该镜像包含 SBOM 证明（attestation）。
> 参见[最大镜像大小](#maximum-image-size)。

1. 使用你的 Docker ID 登录，可通过 `docker login` 命令或 Docker Desktop 中的 **Sign in** 按钮。
2. 构建并 push 你要分析的镜像。

   ```console
   $ docker build --push --tag <org>/<image:tag> --provenance=true --sbom=true .
   ```

   使用 `--provenance=true` 与 `--sbom=true` 构建会将
   [构建证明（build attestations）](/manuals/build/metadata/attestations/_index.md)附加到镜像。
   Docker Scout 会利用这些证明提供更细粒度的分析结果。

   > [!NOTE]
   >
   > 仅当你使用 [containerd 镜像存储](/manuals/desktop/features/containerd.md) 时，
   > 默认的 `docker` 驱动才支持构建证明。

3. 打开 Docker Scout Dashboard 的 [Images 页面](https://scout.docker.com/reports/images)。

   你 push 到仓库的镜像会很快出现在列表中。
   分析结果可能需要等待数分钟才会显示。

## 本地分析镜像

你可以使用 Docker Desktop 或 Docker CLI 的 `docker scout` 命令来分析本地镜像。

### Docker Desktop

> [!NOTE]
>
> Docker Desktop 的后台索引支持大小最高为 10 GB 的镜像。
> 参见[最大镜像大小](#maximum-image-size)。

使用 Docker Desktop GUI 在本地分析镜像：

1. 拉取或构建你想要分析的镜像。
2. 打开 Docker Dashboard 的 **Images** 视图。
3. 在列表中选择一个本地镜像。

   这将打开[镜像详情视图](./image-details-view.md)，展示 Docker Scout 针对该镜像的分析结果，
   包括软件包与发现的漏洞。

### CLI

`docker scout` CLI 命令为你提供在终端使用 Docker Scout 的命令行接口。

- `docker scout quickview`：查看指定镜像的摘要，见[Quickview](#quickview)
- `docker scout cves`：在本地分析指定镜像，见[CVEs](#cves)
- `docker scout compare`：分析并比较两个镜像

默认情况下，结果会输出到标准输出。
你也可以将结果导出为结构化文件，
例如 SARIF（Static Analysis Results Interchange Format，静态分析结果交换格式）。

#### Quickview

`docker scout quickview` 命令会概览指定镜像及其基础镜像中发现的漏洞。

```console
$ docker scout quickview traefik:latest
    ✓ SBOM of image already cached, 311 packages indexed

  Your image  traefik:latest  │    0C     2H     8M     1L
  Base image  alpine:3        │    0C     0H     0M     0L
```

如果你的基础镜像已过期，`quickview` 还会显示“更新基础镜像后对当前镜像漏洞暴露的改善程度”。

```console
$ docker scout quickview postgres:13.1
    ✓ Pulled
    ✓ Image stored for indexing
    ✓ Indexed 187 packages

  Your image  postgres:13.1                 │   17C    32H    35M    33L
  Base image  debian:buster-slim            │    9C    14H     9M    23L
  Refreshed base image  debian:buster-slim  │    0C     1H     6M    29L
                                            │    -9    -13     -3     +6
  Updated base image  debian:stable-slim    │    0C     0H     0M    17L
                                            │    -9    -14     -9     -6
```

#### CVEs

`docker scout cves` 命令为你提供镜像中全部漏洞的完整视图。
该命令支持若干参数，用于更精确地筛选你感兴趣的漏洞，例如按严重等级或软件包类型：

```console
$ docker scout cves --format only-packages --only-vuln-packages \
  --only-severity critical postgres:13.1
    ✓ SBOM of image already cached, 187 packages indexed
    ✗ Detected 10 vulnerable packages with a total of 17 vulnerabilities

     Name            Version         Type        Vulnerabilities
───────────────────────────────────────────────────────────────────────────
  dpkg        1.19.7                 deb      1C     0H     0M     0L
  glibc       2.28-10                deb      4C     0H     0M     0L
  gnutls28    3.6.7-4+deb10u6        deb      2C     0H     0M     0L
  libbsd      0.9.1-2                deb      1C     0H     0M     0L
  libksba     1.3.5-2                deb      2C     0H     0M     0L
  libtasn1-6  4.13-3                 deb      1C     0H     0M     0L
  lz4         1.8.3-1                deb      1C     0H     0M     0L
  openldap    2.4.47+dfsg-3+deb10u5  deb      1C     0H     0M     0L
  openssl     1.1.1d-0+deb10u4       deb      3C     0H     0M     0L
  zlib        1:1.2.11.dfsg-1        deb      1C     0H     0M     0L
```

关于这些命令及其用法的更多信息，请参阅 CLI 参考文档：

- [`docker scout quickview`](/reference/cli/docker/scout/quickview.md)
- [`docker scout cves`](/reference/cli/docker/scout/cves.md)

## 漏洞严重性评估

Docker Scout 基于[通告来源](/manuals/scout/deep-dive/advisory-db-sources.md)中的漏洞数据为漏洞赋予严重等级。
不同类型的软件包在通告中的优先级不同。例如，若漏洞影响的是操作系统软件包，
则更优先采用发行版维护者给出的严重性等级。

如果首选通告来源为某个 CVE 指定了严重等级，但未提供 CVSS 分数，
Docker Scout 会回退显示来自其他来源的 CVSS 分数。
页面会同时展示首选通告来源的严重等级与回退来源的 CVSS 分数。
因此，可能出现“某个漏洞的严重等级为 `LOW`，但 CVSS 分数为 9.8”的情况——
即首选通告标为 `LOW` 且未给出 CVSS 分数，而回退来源给出了 9.8 的 CVSS 分数。

若某漏洞在所有来源中都未被赋予 CVSS 分数，
则会被归类为 **Unspecified**（U，未指定）。

Docker Scout 并未实现自有的漏洞度量体系。
其使用的所有度量均来自已集成的安全通告来源。
不同通告对漏洞分级阈值可能有所不同，
但大多数遵循 CVSS v3.0 规范，其将 CVSS 分数映射到如下严重等级：

| CVSS 分数  | 严重等级          |
| ---------- | ----------------- |
| 0.1 – 3.9  | **Low** (L)       |
| 4.0 – 6.9  | **Medium** (M)    |
| 7.0 – 8.9  | **High** (H)      |
| 9.0 – 10.0 | **Critical** (C)  |

更多信息见 [Vulnerability Metrics (NIST)](https://nvd.nist.gov/vuln-metrics/cvss)。

需要注意的是，由于前述的通告优先与回退机制，
Docker Scout 中显示的严重等级可能与该映射有偏差。

## 最大镜像大小

Docker Scout 平台上的镜像分析，以及 Docker Desktop 后台索引触发的分析，
对镜像文件大小（未压缩）有 10 GB 的限制。
如需分析更大的镜像：

- 在构建时附加 [SBOM attestation](/manuals/build/metadata/attestations/sbom.md)。若镜像包含 SBOM 证明，Docker Scout 将直接使用它而非重新生成，因而不受 10 GB 限制。
- 或者，你可以使用 [CLI](#cli) 在本地分析镜像。使用 CLI 时同样不受 10 GB 限制。若镜像已包含 SBOM 证明，CLI 会利用其加速分析。

