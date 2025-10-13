---
title: 镜像证明存储（Image attestation storage）
---

BuildKit 支持为构建产物创建并附加“证明”（attestations）。这些证明可以从构建
过程中提供有价值的信息，包括但不限于：
[SBOM](https://en.wikipedia.org/wiki/Software_supply_chain)、
[SLSA 溯源](https://slsa.dev/provenance)、构建日志等。

本文档描述当前用于存储证明的自定义格式，该格式旨在与当今主流的镜像仓库实现
保持兼容。未来，我们可能会支持以更多格式导出证明。

证明以清单（manifest）对象的形式存储在镜像索引（image index）中，其风格与
OCI 工件（artifact）类似。

## 属性

### 证明清单（Attestation Manifest）

证明清单附加在镜像的根索引对象下，以独立的
[OCI 镜像清单](https://github.com/opencontainers/image-spec/blob/main/manifest.md)
存在。每个证明清单可包含多个[证明 blob](#证明-blob)，清单中的所有证明均
对应单个平台的镜像清单。标准 OCI 与 Docker 清单的所有属性在此同样适用。

清单中的 `config` 描述符会指向一个有效的
[镜像配置](https://github.com/opencontainers/image-spec/blob/main/config.md)，
但该配置不包含与证明相关的细节，仅为兼容性而保留，可忽略。

`layers` 中的每一层都包含一个单独的[证明 blob](#证明-blob) 的描述符。
每层的 `mediaType` 会根据其内容设置，目前支持：

- `application/vnd.in-toto+json`

  表示 in-toto 证明 blob（当前唯一支持的选项）

任何未知的 `mediaType` 均应被忽略。

为便于遍历证明，可在每个层的描述符上设置以下注解：

- `in-toto.io/predicate-type`

  当封装的证明是 in-toto 证明（当前唯一支持的选项）时会设置该注解；其值
  与证明内部的 `predicateType` 属性相同。

  当该注解存在时，可据此快速定位所需的特定证明，避免拉取其他无关内容。

### 证明 blob（Attestation Blob）

每一层的内容取决于其 `mediaType`：

- `application/vnd.in-toto+json`

  层内容包含完整的
  [in-toto 证明声明（Statement）](https://github.com/in-toto/attestation/blob/main/spec/README.md#statement)：

  ```json
  {
    "_type": "https://in-toto.io/Statement/v0.1",
    "subject": [
      {
        "name": "<NAME>",
        "digest": {"<ALGORITHM>": "<HEX_VALUE>"}
      },
      ...
    ],
    "predicateType": "<URI>",
    "predicate": { ... }
  }
  ```

  证明的 subject 应设置为与
  [证明清单描述符](#证明清单描述符-attestation-manifest-descriptor)
  所指向的目标清单的摘要一致，或为其内部对象的摘要。

### 证明清单描述符（Attestation Manifest Descriptor）

证明清单附加在根[镜像索引](https://github.com/opencontainers/image-spec/blob/main/image-index.md)
的 `manifests` 数组中，位于所有可运行镜像清单之后。标准 OCI 与 Docker 清单
描述符的所有属性在此同样适用。

为防止容器运行时误拉取或运行证明清单所描述的镜像，该证明清单的 `platform`
属性将被设置为 `unknown/unknown`，如下所示：

```json
"platform": {
  "architecture": "unknown",
  "os": "unknown"
}
```

为便于遍历索引，会在该清单描述符上设置如下注解：

- `vnd.docker.reference.type`

  用于标识工件类型，值为 `attestation-manifest`。若为其他值，则应忽略该清单。

- `vnd.docker.reference.digest`

  指向该证明清单所关联的目标对象在镜像索引中的摘要值。

  当该注解存在时，可据此为选定的镜像清单找到匹配的证明清单。

## 示例

_示例：展示一个附加在 `linux/amd64` 镜像上的 SBOM 证明_

#### 镜像索引（`sha256:94acc2ca70c40f3f6291681f37ce9c767e3d251ce01c7e4e9b98ccf148c26260`）：

该镜像索引定义了两个描述符：一个 AMD64 镜像 `sha256:23678f31..`，以及与其对应
的一个证明清单 `sha256:02cb9aa7..`。

```json
{
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "schemaVersion": 2,
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:23678f31b3b3586c4fb318aecfe64a96a1f0916ba8faf9b2be2abee63fa9e827",
      "size": 1234,
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:02cb9aa7600e73fcf41ee9f0f19cc03122b2d8be43d41ce4b21335118f5dd943",
      "size": 1234,
      "annotations": {
        "vnd.docker.reference.digest": "sha256:23678f31b3b3586c4fb318aecfe64a96a1f0916ba8faf9b2be2abee63fa9e827",
        "vnd.docker.reference.type": "attestation-manifest"
      },
      "platform": {
         "architecture": "unknown",
         "os": "unknown"
      }
    }
  ]
}
```

#### 证明清单（`sha256:02cb9aa7600e73fcf41ee9f0f19cc03122b2d8be43d41ce4b21335118f5dd943`）：

该证明清单包含一个 in-toto 证明，其 `predicateType` 为
"https://spdx.dev/Document"，表明它定义了该镜像的 SBOM。

```json
{
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "digest": "sha256:a781560066f20ec9c28f2115a95a886e5e71c7c7aa9d8fd680678498b82f3ea3",
    "size": 123
  },
  "layers": [
    {
      "mediaType": "application/vnd.in-toto+json",
      "digest": "sha256:133ae3f9bcc385295b66c2d83b28c25a9f294ce20954d5cf922dda860429734a",
      "size": 1234,
      "annotations": {
        "in-toto.io/predicate-type": "https://spdx.dev/Document"
      }
    }
  ]
}
```

#### 镜像配置（`sha256:a781560066f20ec9c28f2115a95a886e5e71c7c7aa9d8fd680678498b82f3ea3`）：

```json
{
  "architecture": "unknown",
  "os": "unknown",
  "config": {},
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:133ae3f9bcc385295b66c2d83b28c25a9f294ce20954d5cf922dda860429734a"
    ]
  }
}
```

#### 层内容（`sha256:1ea07d5e55eb47ad0e6bbfa2ec180fb580974411e623814e519064c88f022f5c`）：

该层为证明主体，包含以 SPDX 格式列出的、构建过程中使用的软件包等 SBOM 数据。

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://spdx.dev/Document",
  "subject": [
    {
      "name": "_",
      "digest": {
        "sha256": "23678f31b3b3586c4fb318aecfe64a96a1f0916ba8faf9b2be2abee63fa9e827"
      }
    }
  ],
  "predicate": {
    "SPDXID": "SPDXRef-DOCUMENT",
    "spdxVersion": "SPDX-2.2",
    ...
```


