---
title: 构建证明（Build attestations）
keywords: build, attestations, sbom, provenance, metadata
description: |
  介绍在 Docker Build 中使用 SBOM 与溯源（provenance）证明：
  它们是什么、为何需要以及如何工作
aliases:
  - /build/attestations/
---

{{< youtube-embed qOzcycbTs4o >}}

构建证明用于描述镜像是如何构建的，以及其包含的内容。证明由 BuildKit 在
构建时生成，并作为元数据附加到最终镜像上。

构建证明的目的，是让你能够检查镜像的来源、由谁以何种方式创建、以及镜像
包含哪些内容。借助这些信息，你可以更好地评估镜像对应用供应链安全的影响，
并基于预先设定的策略规则，使用策略引擎对镜像进行合规性校验。

目前主要有两类构建证明：

- 软件物料清单（Software Bill of Materials，SBOM）：列出镜像所包含的软件构件，
  或构建该镜像时使用到的软件构件。
- 溯源证明（provenance）：记录镜像是如何被构建的。

## 构建证明的目的

当下开源与第三方组件的使用比以往更加普遍。开发者之所以共享与复用代码，
是因为这能显著提升效率，帮助团队更快地构建出更好的产品。

但如果在未充分审查的情况下引入外部代码，会带来严重的安全风险。即便你已经
对引入的软件进行了审查，仍可能会出现新的零日漏洞，迫使团队需要及时进行
修复与处置。

构建证明让你更容易了解镜像的内容与来源。你可以基于证明来分析是否采用某个
镜像，或判断当前使用的镜像是否暴露在已知漏洞之下。

## 创建构建证明

使用 `docker buildx build` 构建镜像时，可以通过 `--provenance` 与
`--sbom` 选项为生成的镜像附加证明记录。你可以选择仅添加 SBOM、仅添加
溯源证明，或两者同时添加。

```console
$ docker buildx build --sbom=true --provenance=true .
```

> [!NOTE]
>
> 默认镜像存储不支持构建证明。如果你使用默认镜像存储并通过默认的
> `docker` 驱动构建镜像，或者使用其他驱动但通过 `--load` 加载到本地，
> 这些证明将不会被保留。
>
> 若需确保证明被保留，你可以：
>
> - 使用 `docker-container` 驱动并添加 `--push`，将镜像直接推送到仓库。
> - 启用 [containerd 镜像存储](/manuals/desktop/features/containerd.md)。

> [!NOTE]
>
> 默认启用溯源证明，模式为 `mode=min`。你可以通过 `--provenance=false`
> 禁用溯源证明，或设置环境变量
> [`BUILDX_NO_DEFAULT_ATTESTATIONS`](/manuals/build/building/variables.md#buildx_no_default_attestations)。
>
> 使用 `--provenance=true` 时，默认以 `mode=min` 形式附加溯源证明。
> 详见[溯源证明](./slsa-provenance.md)。

在构建过程中，BuildKit 会生成这些证明。证明记录会以 in-toto JSON 格式
封装，并作为清单附加到最终镜像的镜像索引中。

## 存储

BuildKit 生成的证明符合
[in-toto 格式](https://github.com/in-toto/attestation)，该格式由
[in-toto 框架](https://in-toto.io/)定义，并获得 Linux 基金会支持。

证明以清单的形式附加在镜像索引上；其数据记录以 JSON blob 的形式保存。

由于证明作为清单附加，你无需拉取整个镜像，就可以直接在仓库中查看某个镜像
的证明信息。

所有 BuildKit 导出器均支持证明。但 `local` 与 `tar` 导出器不会将证明写入
镜像清单，因为它们输出的是目录或 tar 包而非镜像。相应地，这些导出器会在
导出结果的根目录下写入一个或多个 JSON 文件，用来保存证明。

## 示例

下面的示例展示了经过截断的、以 in-toto JSON 表示的 SBOM 证明。

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://spdx.dev/Document",
  "subject": [
    {
      "name": "pkg:docker/<registry>/<image>@<tag/digest>?platform=<platform>",
      "digest": {
        "sha256": "e8275b2b76280af67e26f068e5d585eb905f8dfd2f1918b3229db98133cb4862"
      }
    }
  ],
  "predicate": {
    "SPDXID": "SPDXRef-DOCUMENT",
    "creationInfo": {
      "created": "2022-12-15T11:47:54.546747383Z",
      "creators": ["Organization: Anchore, Inc", "Tool: syft-v0.60.3"],
      "licenseListVersion": "3.18"
    },
    "dataLicense": "CC0-1.0",
    "documentNamespace": "https://anchore.com/syft/dir/run/src/core-da0f600b-7f0a-4de0-8432-f83703e6bc4f",
    "name": "/run/src/core",
    // 镜像所包含的文件列表，例如：
    "files": [
      {
        "SPDXID": "SPDXRef-1ac501c94e2f9f81",
        "comment": "layerID: sha256:9b18e9b68314027565b90ff6189d65942c0f7986da80df008b8431276885218e",
        "fileName": "/bin/busybox",
        "licenseConcluded": "NOASSERTION"
      }
    ],
    // 镜像中识别到的包列表：
    "packages": [
      {
        "name": "busybox",
        "originator": "Person: Sören Tempel <soeren+alpine@soeren-tempel.net>",
        "sourceInfo": "acquired package info from APK DB: lib/apk/db/installed",
        "versionInfo": "1.35.0-r17",
        "SPDXID": "SPDXRef-980737451f148c56",
        "description": "Size optimized toolbox of many common UNIX utilities",
        "downloadLocation": "https://busybox.net/",
        "licenseConcluded": "GPL-2.0-only",
        "licenseDeclared": "GPL-2.0-only"
        // ...
      }
    ],
    // 文件与包之间的关系
    "relationships": [
      {
        "relatedSpdxElement": "SPDXRef-1ac501c94e2f9f81",
        "relationshipType": "CONTAINS",
        "spdxElementId": "SPDXRef-980737451f148c56"
      },
      ...
    ],
    "spdxVersion": "SPDX-2.2"
  }
}
```

关于证明存储方式的更多细节，请参阅
[镜像证明存储（BuildKit）](attestation-storage.md)。

## 证明清单格式

证明以清单的形式存储，并由镜像索引引用。每个“证明清单”都对应一个“镜像
清单”（即镜像的某个平台变体）。证明清单仅包含一个层（layer），该层
即为证明的“值”。

下面的示例展示了证明清单的结构：

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 167,
    "digest": "sha256:916d7437a36dd0e258e64d9c5a373ca5c9618eeb1555e79bd82066e593f9afae"
  },
  "layers": [
    {
      "mediaType": "application/vnd.in-toto+json",
      "size": 1833349,
      "digest": "sha256:3138024b98ed5aa8e3008285a458cd25a987202f2500ce1a9d07d8e1420f5491",
      "annotations": {
        "in-toto.io/predicate-type": "https://spdx.dev/Document"
      }
    }
  ]
}
```

### 将证明作为 OCI 工件

你可以通过 `image` 与 `registry` 导出器的
[`oci-artifact` 选项](/manuals/build/exporters/image-registry.md#synopsis)
来配置证明清单的格式。若设置为 `true`，证明清单的结构会发生如下变化：

- 在证明清单中新增 `artifactType` 字段，值为 `application/vnd.docker.attestation.manifest.v1+json`。
- `config` 字段将变为一个 [empty descriptor]，而不再是“占位”配置。
- 新增 `subject` 字段，指向该证明所对应的镜像清单。

[empty descriptor]: https://github.com/opencontainers/image-spec/blob/main/manifest.md#guidance-for-an-empty-descriptor

下面的示例展示了采用 OCI artifact 格式的证明：

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.docker.attestation.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "size": 2,
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "application/vnd.in-toto+json",
      "size": 2208,
      "digest": "sha256:6d2f2c714a6bee3cf9e4d3cb9a966b629efea2dd8556ed81f19bd597b3325286",
      "annotations": {
        "in-toto.io/predicate-type": "https://slsa.dev/provenance/v0.2"
      }
    }
  ],
  "subject": {
    "mediaType": "application/vnd.oci.image.manifest.v1+json",
    "size": 1054,
    "digest": "sha256:bc2046336420a2852ecf915786c20f73c4c1b50d7803aae1fd30c971a7d1cead",
    "platform": {
      "architecture": "amd64",
      "os": "linux"
    }
  }
}
```

## 下一步

进一步了解可用的证明类型及其用法：

- [溯源证明（Provenance）](slsa-provenance.md)
- [软件物料清单（SBOM）](sbom.md)
