---
title: 溯源证明（Provenance attestations）
keywords: build, attestations, provenance, slsa, git, metadata
description: >
  溯源构建证明用于描述镜像是如何、在何处被构建的。
aliases:
  - /build/attestations/slsa-provenance/
---

溯源证明包含关于构建过程的事实信息，例如：

- Build timestamps
- Build parameters and environment
- Version control metadata
- Source code details
- Materials (files, scripts) consumed during the build

溯源证明遵循
[SLSA 溯源模式（0.2 版）](https://slsa.dev/provenance/v0.2#schema)。

关于 BuildKit 如何填充这些溯源属性的更多信息，请参见
[SLSA 定义](slsa-definitions.md)。

## 创建溯源证明

要创建溯源证明，请在 `docker buildx build` 命令中传入
`--attest type=provenance` 选项：

```console
$ docker buildx build --tag <namespace>/<image>:<version> \
    --attest type=provenance,mode=[min,max] .
```

你也可以使用简写选项 `--provenance=true` 来替代 `--attest type=provenance`。
若需通过简写指定 `mode` 参数，可使用：`--provenance=mode=max`。

若要在 GitHub Actions 中添加溯源证明，请参见
[通过 GitHub Actions 添加证明](/manuals/build/ci/github-actions/attestations.md)。

## 模式（Mode）

你可以使用 `mode` 参数控制溯源证明包含的详细程度。可选值为 `mode=min`
（默认）与 `mode=max`。

### Min

在 `min` 模式下，溯源证明仅包含最小化的信息，例如：

- Build timestamps
- The frontend used
- Build materials
- Source repository and revision
- Build platform
- Reproducibility

构建参数的取值、秘钥标识、以及丰富的层（layer）元数据不会包含在 `min`
模式中。`min` 级别的溯源对所有构建都是安全的，因为它不会泄露任何构建环境
中的敏感信息。

下面的 JSON 示例展示了以 `min` 模式创建的溯源证明所包含的信息：

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://slsa.dev/provenance/v0.2",
  "subject": [
    {
      "name": "pkg:docker/<registry>/<image>@<tag/digest>?platform=<platform>",
      "digest": {
        "sha256": "e8275b2b76280af67e26f068e5d585eb905f8dfd2f1918b3229db98133cb4862"
      }
    }
  ],
  "predicate": {
    "builder": { "id": "" },
    "buildType": "https://mobyproject.org/buildkit@v1",
    "materials": [
      {
        "uri": "pkg:docker/docker/dockerfile@1",
        "digest": {
          "sha256": "9ba7531bd80fb0a858632727cf7a112fbfd19b17e94c4e84ced81e24ef1a0dbc"
        }
      },
      {
        "uri": "pkg:docker/golang@1.19.4-alpine?platform=linux%2Farm64",
        "digest": {
          "sha256": "a9b24b67dc83b3383d22a14941c2b2b2ca6a103d805cac6820fd1355943beaf1"
        }
      }
    ],
    "invocation": {
      "configSource": { "entryPoint": "Dockerfile" },
      "parameters": {
        "frontend": "gateway.v0",
        "args": {
          "cmdline": "docker/dockerfile:1",
          "source": "docker/dockerfile:1",
          "target": "binaries"
        },
        "locals": [{ "name": "context" }, { "name": "dockerfile" }]
      },
      "environment": { "platform": "linux/arm64" }
    },
    "metadata": {
      "buildInvocationID": "c4a87v0sxhliuewig10gnsb6v",
      "buildStartedOn": "2022-12-16T08:26:28.651359794Z",
      "buildFinishedOn": "2022-12-16T08:26:29.625483253Z",
      "reproducible": false,
      "completeness": {
        "parameters": true,
        "environment": true,
        "materials": false
      },
      "https://mobyproject.org/buildkit@v1#metadata": {
        "vcs": {
          "revision": "a9ba846486420e07d30db1107411ac3697ecab68",
          "source": "git@github.com:<org>/<repo>.git"
        }
      }
    }
  }
}
```

### Max

`max` 模式包含 `min` 模式中的所有信息，此外还包括：

- The LLB definition of the build. These show the exact steps taken to produce
  the image.
- Information about the Dockerfile, including a full base64-encoded version of
  the file.
- Source maps describing the relationship between build steps and image layers.

在可能的情况下，建议优先使用 `mode=max`，因为它提供了更多可用于分析的细节。

> [!WARNING]
>
> 注意：`mode=max` 会暴露
> [构建参数](/reference/cli/docker/buildx/build.md#build-arg) 的取值。
>
> 如果你误用构建参数来传递凭据、认证令牌或其他机密信息，应尽快改为使用
> [secret mount](/reference/cli/docker/buildx/build.md#secret)。
> secret mount 不会泄露到构建之外，且不会被包含在溯源证明中。

## 查看溯源信息

若要查看通过 `image` 导出器导出的溯源信息，可使用
[`imagetools inspect`](/reference/cli/docker/buildx/imagetools/inspect.md)。

配合 `--format` 选项可指定输出模板。所有与溯源相关的数据均可通过
`.Provenance` 属性访问。例如，获取 SLSA 格式的原始溯源内容：

```console
$ docker buildx imagetools inspect <namespace>/<image>:<version> \
    --format "{{ json .Provenance.SLSA }}"
{
  "buildType": "https://mobyproject.org/buildkit@v1",
  ...
}
```

你也可以使用 Go 模板的完整能力来构造更复杂的表达式。例如，对于
`mode=max` 生成的溯源信息，可以提取用于构建镜像的 Dockerfile 的完整源码：

```console
$ docker buildx imagetools inspect <namespace>/<image>:<version> \
    --format '{{ range (index .Provenance.SLSA.metadata "https://mobyproject.org/buildkit@v1#metadata").source.infos }}{{ if eq .filename "Dockerfile" }}{{ .data }}{{ end }}{{ end }}' | base64 -d
FROM ubuntu:24.04
RUN apt-get update
...
```

## 溯源证明示例

<!-- TODO: add a link to the definitions page, imported from moby/buildkit -->

下面的示例展示了 `mode=max` 模式下溯源证明的 JSON 表示：

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://slsa.dev/provenance/v0.2",
  "subject": [
    {
      "name": "pkg:docker/<registry>/<image>@<tag/digest>?platform=<platform>",
      "digest": {
        "sha256": "e8275b2b76280af67e26f068e5d585eb905f8dfd2f1918b3229db98133cb4862"
      }
    }
  ],
  "predicate": {
    "builder": { "id": "" },
    "buildType": "https://mobyproject.org/buildkit@v1",
    "materials": [
      {
        "uri": "pkg:docker/docker/dockerfile@1",
        "digest": {
          "sha256": "9ba7531bd80fb0a858632727cf7a112fbfd19b17e94c4e84ced81e24ef1a0dbc"
        }
      },
      {
        "uri": "pkg:docker/golang@1.19.4-alpine?platform=linux%2Farm64",
        "digest": {
          "sha256": "a9b24b67dc83b3383d22a14941c2b2b2ca6a103d805cac6820fd1355943beaf1"
        }
      }
    ],
    "buildConfig": {
      "llbDefinition": [
        {
          "id": "step4",
          "op": {
            "Op": {
              "exec": {
                "meta": {
                  "args": ["/bin/sh", "-c", "go mod download -x"],
                  "env": [
                    "PATH=/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                    "GOLANG_VERSION=1.19.4",
                    "GOPATH=/go",
                    "CGO_ENABLED=0"
                  ],
                  "cwd": "/src"
                },
                "mounts": [
                  { "input": 0, "dest": "/", "output": 0 },
                  {
                    "input": -1,
                    "dest": "/go/pkg/mod",
                    "output": -1,
                    "mountType": 3,
                    "cacheOpt": { "ID": "//go/pkg/mod" }
                  },
                  {
                    "input": 1,
                    "selector": "/go.mod",
                    "dest": "/src/go.mod",
                    "output": -1,
                    "readonly": true
                  },
                  {
                    "input": 1,
                    "selector": "/go.sum",
                    "dest": "/src/go.sum",
                    "output": -1,
                    "readonly": true
                  }
                ]
              }
            },
            "platform": { "Architecture": "arm64", "OS": "linux" },
            "constraints": {}
          },
          "inputs": ["step3:0", "step1:0"]
        }
      ]
    },
    "metadata": {
      "buildInvocationID": "edf52vxjyf9b6o5qd7vgx0gru",
      "buildStartedOn": "2022-12-15T15:38:13.391980297Z",
      "buildFinishedOn": "2022-12-15T15:38:14.274565297Z",
      "reproducible": false,
      "completeness": {
        "parameters": true,
        "environment": true,
        "materials": false
      },
      "https://mobyproject.org/buildkit@v1#metadata": {
        "vcs": {
          "revision": "a9ba846486420e07d30db1107411ac3697ecab68-dirty",
          "source": "git@github.com:<org>/<repo>.git"
        },
        "source": {
          "locations": {
            "step4": {
              "locations": [
                {
                  "ranges": [
                    { "start": { "line": 5 }, "end": { "line": 5 } },
                    { "start": { "line": 6 }, "end": { "line": 6 } },
                    { "start": { "line": 7 }, "end": { "line": 7 } },
                    { "start": { "line": 8 }, "end": { "line": 8 } }
                  ]
                }
              ]
            }
          },
          "infos": [
            {
              "filename": "Dockerfile",
              "data": "RlJPTSBhbHBpbmU6bGF0ZXN0Cg==",
              "llbDefinition": [
                {
                  "id": "step0",
                  "op": {
                    "Op": {
                      "source": {
                        "identifier": "local://dockerfile",
                        "attrs": {
                          "local.differ": "none",
                          "local.followpaths": "[\"Dockerfile\",\"Dockerfile.dockerignore\",\"dockerfile\"]",
                          "local.session": "s4j58ngehdal1b5hn7msiqaqe",
                          "local.sharedkeyhint": "dockerfile"
                        }
                      }
                    },
                    "constraints": {}
                  }
                },
                { "id": "step1", "op": { "Op": null }, "inputs": ["step0:0"] }
              ]
            }
          ]
        },
        "layers": {
          "step2:0": [
            [
              {
                "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
                "digest": "sha256:261da4162673b93e5c0e7700a3718d40bcc086dbf24b1ec9b54bca0b82300626",
                "size": 3259190
              },
              {
                "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
                "digest": "sha256:bc729abf26b5aade3c4426d388b5ea6907fe357dec915ac323bb2fa592d6288f",
                "size": 286218
              },
              {
                "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
                "digest": "sha256:7f1d6579712341e8062db43195deb2d84f63b0f2d1ed7c3d2074891085ea1b56",
                "size": 116878653
              },
              {
                "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
                "digest": "sha256:652874aefa1343799c619d092ab9280b25f96d97939d5d796437e7288f5599c9",
                "size": 156
              }
            ]
          ]
        }
      }
    }
  }
}
```
