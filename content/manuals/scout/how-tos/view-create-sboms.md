---
title: 使用 Docker Scout 查看与生成 SBOM
description: 使用 Docker Scout 为你的项目提取（或生成）SBOM。
keywords: scout, supply chain, sbom, software bill of material, spdx, cli, attestations, file
aliases:
- /engine/sbom/
- /scout/sbom/
---

【说明】镜像分析会基于镜像的 SBOM 来识别其中包含的软件包及其版本。
若镜像已附带 SBOM 证明（attestation），Docker Scout 会优先使用（推荐）。
如果镜像没有 SBOM 证明，Docker Scout 会通过索引镜像内容来生成一份。

## 通过 CLI 查看

要查看 Docker Scout 生成的 SBOM 内容，可使用 `docker scout sbom` 命令：

```console
$ docker scout sbom [IMAGE]
```

默认情况下，该命令会以 JSON 格式将 SBOM 输出到标准输出。
注意：`docker scout sbom` 默认生成的 JSON 并非 SPDX-JSON。
如需输出 SPDX 格式，请使用 `--format spdx` 参数：

```console
$ docker scout sbom --format spdx [IMAGE]
```

若想获得更易读的列表格式，请使用 `--format list` 参数：

```console
$ docker scout sbom --format list alpine

           Name             Version    Type
───────────────────────────────────────────────
  alpine-baselayout       3.4.3-r1     apk
  alpine-baselayout-data  3.4.3-r1     apk
  alpine-keys             2.4-r1       apk
  apk-tools               2.14.0-r2    apk
  busybox                 1.36.1-r2    apk
  busybox-binsh           1.36.1-r2    apk
  ca-certificates         20230506-r0  apk
  ca-certificates-bundle  20230506-r0  apk
  libc-dev                0.7.2-r5     apk
  libc-utils              0.7.2-r5     apk
  libcrypto3              3.1.2-r0     apk
  libssl3                 3.1.2-r0     apk
  musl                    1.2.4-r1     apk
  musl-utils              1.2.4-r1     apk
  openssl                 3.1.2-r0     apk
  pax-utils               1.3.7-r1     apk
  scanelf                 1.3.7-r1     apk
  ssl_client              1.36.1-r2    apk
  zlib                    1.2.13-r1    apk
```

关于 `docker scout sbom` 命令的更多信息，参见[CLI 参考](/reference/cli/docker/scout/sbom.md)。

## 作为构建证明附加 {#attest}

你可以在构建阶段生成 SBOM，并以[证明（attestation）](/manuals/build/metadata/attestations/_index.md)的形式附加到镜像上。BuildKit 提供的默认 SBOM 生成器与 Docker Scout 使用的不同。
你可以通过 `docker build` 命令的 `--attest` 参数，配置 BuildKit 使用 Docker Scout 的 SBOM 生成器。
Docker Scout 的 SBOM 索引器能产出更丰富的结果，并与 Docker Scout 的镜像分析有更好的兼容性。

```console
$ docker build --tag <org>/<image> \
  --attest type=sbom,generator=docker/scout-sbom-indexer:latest \
  --push .
```

要构建带有 SBOM 证明的镜像，你必须启用 [containerd 镜像存储](/manuals/desktop/features/containerd.md)功能，或使用 `docker-container` 构建器并配合 `--push` 参数将镜像（包含证明）直接推送到仓库。经典镜像存储不支持清单列表或镜像索引，而这些是向镜像添加证明所必需的。

## 导出为文件

将镜像的 SBOM 导出为 SPDX JSON 文件的方式，取决于该镜像是否已推送到仓库，或只是本地镜像。

### 远程镜像

要提取镜像的 SBOM 并保存到文件，可使用 `docker buildx imagetools inspect` 命令。该命令仅适用于仓库中的镜像。

```console
$ docker buildx imagetools inspect <image> --format "{{ json .SBOM }}" > sbom.spdx.json
```

### 本地镜像

如需为本地镜像导出 SPDX 文件，请使用 `local` 导出器进行构建，并启用 `scout-sbom-indexer` SBOM 生成器插件。

如下命令会将 SBOM 保存到 `build/sbom.spdx.json`：

```console
$ docker build --attest type=sbom,generator=docker/scout-sbom-indexer:latest \
  --output build .
```
