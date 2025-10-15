---
title: 将 Docker Scout 用于不同制品类型
description: |
  部分 Docker Scout 命令支持在镜像引用前添加前缀，
  用于指定你要分析的镜像或文件的位置或类型。
keywords: scout, vulnerabilities, analyze, analysis, cli, packages, sbom, cve, security, local, source, code, supply chain
aliases:
  - /scout/image-prefix/
---

部分 Docker Scout CLI 命令支持使用前缀，以指定你希望分析的制品位置或类型。

默认情况下，使用 `docker scout cves` 分析镜像时，会以 Docker 引擎的本地镜像存储为目标。
若本地已存在该镜像，下面的命令将始终使用本地镜像：

```console
$ docker scout cves <image>
```

如果本地不存在该镜像，Docker 会在分析前先拉取该镜像。
再次分析同一镜像时，默认仍会使用本地版本，即使镜像在仓库中的标签已更新。

在镜像引用前添加 `registry://` 前缀，可强制 Docker Scout 分析镜像仓库中的版本：

```console
$ docker scout cves registry://<image>
```

## 支持的前缀

支持的前缀包括：

| 前缀                  | 说明                                                                 |
| --------------------- | -------------------------------------------------------------------- |
| `image://`（默认）     | 优先使用本地镜像；若本地不存在则回退到仓库查询                          |
| `local://`            | 仅使用本地镜像存储中的镜像（不访问仓库）                               |
| `registry://`         | 仅使用镜像仓库中的镜像（忽略本地镜像）                                 |
| `oci-dir://`          | 使用 OCI 布局目录                                                     |
| `archive://`          | 使用 tar 包归档文件（例如通过 `docker save` 生成）                    |
| `fs://`               | 使用本地目录或文件                                                    |

以下命令均可使用这些前缀：

- `docker scout compare`
- `docker scout cves`
- `docker scout quickview`
- `docker scout recommendations`
- `docker scout sbom`

## 示例

本节通过若干示例展示如何使用前缀为 `docker scout` 命令指定制品。

### 分析本地项目

`fs://` 前缀允许你直接分析本地源代码，而无需先构建为容器镜像。
以下 `docker scout quickview` 命令会对当前工作目录下的源代码给出一目了然的漏洞概览：

```console
$ docker scout quickview fs://.
```

若要查看本地源代码中发现的漏洞详情，你可以使用 `docker scout cves --details fs://.` 命令，
并结合其他标志来收敛结果范围，聚焦你感兴趣的包与漏洞。

```console
$ docker scout cves --details --only-severity high fs://.
    ✓ File system read
    ✓ Indexed 323 packages
    ✗ Detected 1 vulnerable package with 1 vulnerability

​## Overview

                    │        Analyzed path
────────────────────┼──────────────────────────────
  Path              │  /Users/david/demo/scoutfs
    vulnerabilities │    0C     1H     0M     0L

​## Packages and Vulnerabilities

   0C     1H     0M     0L  fastify 3.29.0
pkg:npm/fastify@3.29.0

    ✗ HIGH CVE-2022-39288 [OWASP Top Ten 2017 Category A9 - Using Components with Known Vulnerabilities]
      https://scout.docker.com/v/CVE-2022-39288

      fastify is a fast and low overhead web framework, for Node.js. Affected versions of
      fastify are subject to a denial of service via malicious use of the Content-Type
      header. An attacker can send an invalid Content-Type header that can cause the
      application to crash. This issue has been addressed in commit  fbb07e8d  and will be
      included in release version 4.8.1. Users are advised to upgrade. Users unable to
      upgrade may manually filter out http content with malicious Content-Type headers.

      Affected range : <4.8.1
      Fixed version  : 4.8.1
      CVSS Score     : 7.5
      CVSS Vector    : CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H

1 vulnerability found in 1 package
  LOW       0
  MEDIUM    0
  HIGH      1
  CRITICAL  0
```

### 将本地项目与镜像对比

使用 `docker scout compare`，你可以将本地文件系统上的源代码分析结果与某个容器镜像的分析结果进行对比。
下面的示例将本地源代码（`fs://.`）与一个仓库镜像 `registry://docker/scout-cli:latest` 进行对比。
此处比较的基线与目标均使用了前缀。

```console
$ docker scout compare fs://. --to registry://docker/scout-cli:latest --ignore-unchanged
WARN 'docker scout compare' is experimental and its behaviour might change in the future
    ✓ File system read
    ✓ Indexed 268 packages
    ✓ SBOM of image already cached, 234 packages indexed


  ## Overview

                           │              Analyzed File System              │              Comparison Image
  ─────────────────────────┼────────────────────────────────────────────────┼─────────────────────────────────────────────
    Path / Image reference │  /Users/david/src/docker/scout-cli-plugin      │  docker/scout-cli:latest
                           │                                                │  bb0b01303584
      platform             │                                                │ linux/arm64
      provenance           │ https://github.com/dvdksn/scout-cli-plugin.git │ https://github.com/docker/scout-cli-plugin
                           │  6ea3f7369dbdfec101ac7c0fa9d78ef05ffa6315      │  67cb4ef78bd69545af0e223ba5fb577b27094505
      vulnerabilities      │    0C     0H     1M     1L                     │    0C     0H     1M     1L
                           │                                                │
      size                 │ 7.4 MB (-14 MB)                                │ 21 MB
      packages             │ 268 (+34)                                      │ 234
                           │                                                │


  ## Packages and Vulnerabilities


    +   55 packages added
    -   21 packages removed
       213 packages unchanged
```

为简洁起见，上述示例输出已省略部分内容。

### 查看镜像 tar 包的 SBOM

以下示例展示如何使用 `archive://` 前缀，获取通过 `docker save` 生成的镜像 tar 包的 SBOM。
此处镜像为 `docker/scout-cli:latest`，并将 SBOM 以 SPDX 格式导出到文件 `sbom.spdx.json`。

```console
$ docker pull docker/scout-cli:latest
latest: Pulling from docker/scout-cli
257973a141f5: Download complete 
1f2083724dd1: Download complete 
5c8125a73507: Download complete 
Digest: sha256:13318bb059b0f8b0b87b35ac7050782462b5d0ac3f96f9f23d165d8ed68d0894
$ docker save docker/scout-cli:latest -o scout-cli.tar
$ docker scout sbom --format spdx -o sbom.spdx.json archive://scout-cli.tar
```

## 进一步了解

有关命令与可用标志的更多信息，请参阅 CLI 参考文档：

- [`docker scout quickview`](/reference/cli/docker/scout/quickview.md)
- [`docker scout cves`](/reference/cli/docker/scout/cves.md)
- [`docker scout compare`](/reference/cli/docker/scout/compare.md)
