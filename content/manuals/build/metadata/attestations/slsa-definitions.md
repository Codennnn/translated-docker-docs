---
title: SLSA 定义（SLSA definitions）
---

BuildKit 支持为其运行的构建创建[ SLSA 溯源](./slsa-provenance.md)。

BuildKit 生成的溯源格式遵循 SLSA Provenance 规范（同时支持
[v0.2](https://slsa.dev/spec/v0.2/provenance) 与
[v1](https://slsa.dev/spec/v1.1/provenance)）。

本文说明 BuildKit 如何填充各字段，以及在你以 `mode=min` 与 `mode=max`
生成证明时，各字段是否会被包含。

## SLSA v1

### `buildDefinition.buildType`

* 参考：https://slsa.dev/spec/v1.1/provenance#buildType
* `mode=min` 与 `mode=max` 均包含。

`buildDefinition.buildType` 被设置为
`https://github.com/moby/buildkit/blob/master/docs/attestations/slsa-definitions.md`，
可用于确定溯源内容的结构。

```json
    "buildDefinition": {
      "buildType": "https://github.com/moby/buildkit/blob/master/docs/attestations/slsa-definitions.md",
      ...
    }
```

### `buildDefinition.externalParameters.configSource`

* 参考：https://slsa.dev/spec/v1.1/provenance#externalParameters
* `mode=min` 与 `mode=max` 均包含。

描述初始化构建所使用的配置。

```json
    "buildDefinition": {
      "externalParameters": {
        "configSource": {
          "uri": "https://github.com/moby/buildkit.git#refs/tags/v0.11.0",
          "digest": {
            "sha1": "4b220de5058abfd01ff619c9d2ff6b09a049bea0"
          },
          "path": "Dockerfile"
        },
        ...
      },
    }
```

对于来自远程上下文（如 Git 或 HTTP URL）的构建，该对象在 `uri` 与 `digest`
字段中给出上下文 URL 及其不可变摘要。对于使用本地前端（如 Dockerfile）的
构建，`path` 字段表示初始化构建的前端文件路径（对应 `filename` 前端选项）。

### `buildDefinition.externalParameters.request`

* 参考：https://slsa.dev/spec/v1.1/provenance#externalParameters
* `mode=min` 下部分包含。

描述传入构建的输入。

```json
    "buildDefinition": {
      "externalParameters": {
        "request": {
          "frontend": "gateway.v0",
          "args": {
            "build-arg:BUILDKIT_CONTEXT_KEEP_GIT_DIR": "1",
            "label:FOO": "bar",
            "source": "docker/dockerfile-upstream:master",
            "target": "release"
          },
          "secrets": [
            {
              "id": "GIT_AUTH_HEADER",
              "optional": true
            },
            ...
          ],
          "ssh": [],
          "locals": []
        },
        ...
      },
    }
```

以下字段在 `mode=min` 与 `mode=max` 中均会包含：

- `locals`：列出构建中使用的本地来源，包括构建上下文与前端文件。
- `frontend`：构建所用的 BuildKit 前端类型。目前可能为 `dockerfile.v0` 或
  `gateway.v0`。
- `args`：传入 BuildKit 前端的构建参数。

  `args` 对象中的键与 BuildKit 接收到的选项一致。例如，`build-arg` 与
  `label` 前缀分别用于构建参数与标签，`target` 指定构建的目标阶段；若使用
  Gateway 前端，`source` 指定其来源镜像。

以下字段仅在 `mode=max` 中包含：

- `secrets`：构建过程中使用的机密（不会包含机密的实际值）。
- `ssh`：构建过程中使用的 SSH 转发。

### `buildDefinition.internalParameters.buildConfig`

* 参考：https://slsa.dev/spec/v1.1/provenance#internalParameters
* 仅在 `mode=max` 中包含。

定义构建期间实际执行的构建步骤。

BuildKit 内部使用 LLB 定义来执行构建步骤。构建步骤的 LLB 定义存放在
`buildDefinition.internalParameters.buildConfig.llbDefinition` 中。

每个 LLB 步骤是
[LLB ProtoBuf API](https://github.com/moby/buildkit/blob/v0.10.0/solver/pb/ops.proto)
的 JSON 表示。LLB 图中每个顶点的依赖可在该步骤的 `inputs` 字段中找到。

```json
    "buildDefinition": {
      "internalParameters": {
        "buildConfig": {
          "llbDefinition": [
            {
              "id": "step0",
              "op": {
                "Op": {
                  "exec": {
                    "meta": {
                      "args": [
                        "/bin/sh",
                        "-c",
                        "go build ."
                      ],
                      "env": [
                        "PATH=/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                        "GOPATH=/go",
                        "GOFLAGS=-mod=vendor",
                      ],
                      "cwd": "/src",
                    },
                    "mounts": [...]
                  }
                },
                "platform": {...},
              },
              "inputs": [
                "step8:0",
                "step2:0",
              ]
            },
            ...
          ]
        },
      }
    }
```

### `buildDefinition.internalParameters.builderPlatform`

* 参考：https://slsa.dev/spec/v1.1/provenance#internalParameters
* `mode=min` 与 `mode=max` 均包含。

```json
    "buildDefinition": {
      "internalParameters": {
        "builderPlatform": "linux/amd64"
        ...
      },
    }
```

BuildKit 会记录构建机的 `builderPlatform`。注意，这不一定等同于构建结果的
平台；构建结果的平台可从 `in-toto` 的 subject 字段确定。

### `buildDefinition.resolvedDependencies`

* 参考：https://slsa.dev/spec/v1.1/provenance#resolvedDependencies
* `mode=min` 与 `mode=max` 均包含。

定义构建所涉及的所有外部工件。取值取决于工件类型：

- 包含镜像源码的 Git 仓库 URL
- 远程 tar 包的 HTTP URL，或通过 Dockerfile 的 `ADD` 指令包含的 URL
- 构建期间使用的任意 Docker 镜像

Docker 镜像的 URL 采用
[Package URL](https://github.com/package-url/purl-spec) 格式。

所有构建材料都会包含不可变的校验和。若从可变标签构建，你可以通过摘要信息来
判断该工件相较于构建时是否已被更新。

```json
    "buildDefinition": {
      "resolvedDependencies": [
        {
          "uri": "pkg:docker/alpine@3.17?platform=linux%2Famd64",
          "digest": {
            "sha256": "8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4"
          }
        },
        {
          "uri": "https://github.com/moby/buildkit.git#refs/tags/v0.11.0",
          "digest": {
            "sha1": "4b220de5058abfd01ff619c9d2ff6b09a049bea0"
          }
        },
        ...
      ],
      ...
    }
```

### `runDetails.builder.id`

* 参考：https://slsa.dev/spec/v1.1/provenance#builder.id
* `mode=min` 与 `mode=max` 均包含。

若可用，该字段设置为构建的 URL。

```json
    "runDetails": {
      "builder": {
        "id": "https://github.com/docker/buildx/actions/runs/3709599520"
        ...
      },
      ...
    }
```

> [!NOTE]
> 也可通过证明参数 `builder-id` 进行设置。

### `runDetails.metadata.invocationID`

* 参考：https://slsa.dev/spec/v1.1/provenance#invocationId
* `mode=min` 与 `mode=max` 均包含。

构建调用的唯一标识符。对单次请求构建多平台镜像时，该值会被该镜像的所有平台
版本共享。

```json
    "runDetails": {
      "metadata": {
        "invocationID": "rpv7a389uzil5lqmrgwhijwjz",
        ...
      },
      ...
    }
```

### `runDetails.metadata.startedOn`

* 参考：https://slsa.dev/spec/v1.1/provenance#startedOn
* `mode=min` 与 `mode=max` 均包含。

构建开始的时间戳。

```json
    "runDetails": {
      "metadata": {
        "startedOn": "2021-11-17T15:00:00Z",
        ...
      },
      ...
    }
```

### `runDetails.metadata.finishedOn`

* 参考：https://slsa.dev/spec/v1.1/provenance#finishedOn
* `mode=min` 与 `mode=max` 均包含。

构建结束的时间戳。

```json
    "runDetails": {
      "metadata": {
        "finishedOn": "2021-11-17T15:01:00Z",
        ...
      },
    }
```

### `runDetails.metadata.buildkit_metadata`

* 参考：https://slsa.dev/spec/v1.1/provenance#extension-fields
* `mode=min` 下部分包含。

该扩展字段定义 BuildKit 特有、非 SLSA 规范的一些附加元数据。

```json
    "runDetails": {
      "metadata": {
        "buildkit_metadata": {
          "source": {...},
          "layers": {...},
          "vcs": {...},
        },
        ...
      },
    }
```

#### `source`

仅在 `mode=max` 中包含。

定义 LLB 构建步骤（定义于
`buildDefinition.internalParameters.buildConfig.llbDefinition`）到其原始源码
（如 Dockerfile 命令）的映射。`source.locations` 包含每个 LLB 步骤中运行的
Dockerfile 命令的行范围；`source.infos` 数组包含源码本身。若前端在生成 LLB
定义时提供了映射，则会包含该映射。

#### `layers`

仅在 `mode=max` 中包含。

定义 LLB 构建步骤的挂载与等效层的 OCI 描述符之间的映射（对应
`buildDefinition.internalParameters.buildConfig.llbDefinition`）。当存在层数据
时（通常针对镜像证明，或构建步骤在构建过程中拉取了镜像数据），该映射可用。

#### `vcs`

`mode=min` 与 `mode=max` 均包含。

定义构建所用版本控制系统的可选元数据。若构建使用来自 Git 仓库的远程上下文，
BuildKit 会自动提取详细信息并显示在
`buildDefinition.externalParameters.configSource` 中。若构建使用本地目录作
为来源，即便该目录包含 Git 仓库，VCS 信息也会丢失。在此情况下，构建客户端
可以发送额外的 `vcs:source` 与 `vcs:revision` 构建选项，BuildKit 会将其作
为额外元数据添加到溯源证明中。注意，与
`buildDefinition.externalParameters.configSource` 不同，BuildKit 不会校验
`vcs` 的取值，因此它们不应被信任，仅可作为元数据提示。

### `runDetails.metadata.buildkit_hermetic`

* 参考：https://slsa.dev/spec/v1.1/provenance#extension-fields
* `mode=min` 与 `mode=max` 均包含。

如果构建是密闭（hermetic）的且未访问网络，则该扩展字段为 true。在 Dockerfile
中，若未使用 `RUN` 命令，或通过 `--network=none` 禁用了网络，则该构建为
hermetic。

```json
    "runDetails": {
      "metadata": {
        "buildkit_hermetic": true,
        ...
      },
    }
```

### `runDetails.metadata.buildkit_completeness`

* 参考：https://slsa.dev/spec/v1.1/provenance#extension-fields
* `mode=min` 与 `mode=max` 均包含。

该扩展字段定义溯源信息是否完整。与 SLSA v0.2 中的 `metadata.completeness`
含义相似。

当所有构建参数都包含在 `buildDefinition.externalParameters.request` 中时，
`buildkit_completeness.request` 为 true。在 `min` 模式下，构建参数不会包含
在溯源信息中，因此 request 不完整。对未使用前端的直接 LLB 构建，request
也不完整。

当 `buildDefinition.resolvedDependencies` 包含了构建的所有依赖时，
`buildkit_completeness.resolvedDependencies` 为 true。若从本地目录的
未追踪来源构建，则依赖不完整；若从远程 Git 仓库构建，BuildKit 可追踪所有
依赖，此字段为 true。

```json
    "runDetails": {
      "metadata": {
        "buildkit_completeness": {
          "request": true,
          "resolvedDependencies": true
        },
        ...
      },
    }
```

### `runDetails.metadata.buildkit_reproducible`

* 参考：https://slsa.dev/spec/v1.1/provenance#extension-fields
* `mode=min` 与 `mode=max` 均包含。

定义构建结果是否应当达到字节级可复现。与 SLSA v0.2 中的 `metadata.reproducible`
含义相似。可通过证明参数 `reproducible=true` 设置该值。

```json
    "runDetails": {
      "metadata": {
        "buildkit_reproducible": false,
        ...
      },
    }
```

## SLSA v0.2

### `builder.id`

* 参考：https://slsa.dev/spec/v0.2/provenance#builder.id
* `mode=min` 与 `mode=max` 均包含。

若可用，该字段设置为构建的 URL。

```json
    "builder": {
      "id": "https://github.com/docker/buildx/actions/runs/3709599520"
    },
```

> [!NOTE]
> 也可通过证明参数 `builder-id` 进行设置。

### `buildType`

* 参考：https://slsa.dev/spec/v0.2/provenance#buildType
* `mode=min` 与 `mode=max` 均包含。

`buildType` 被设置为 `https://mobyproject.org/buildkit@v1`，可用于确定溯源
内容的结构。

```json
    "buildType": "https://mobyproject.org/buildkit@v1",
```

### `invocation.configSource`

* 参考：https://slsa.dev/spec/v0.2/provenance#invocation.configSource
* `mode=min` 与 `mode=max` 均包含。

描述初始化构建所使用的配置。

```json
    "invocation": {
      "configSource": {
        "uri": "https://github.com/moby/buildkit.git#refs/tags/v0.11.0",
        "digest": {
          "sha1": "4b220de5058abfd01ff619c9d2ff6b09a049bea0"
        },
        "entryPoint": "Dockerfile"
      },
      ...
    },
```

对于来自远程上下文（如 Git 或 HTTP URL）的构建，该对象在 `uri` 与 `digest`
字段中给出上下文 URL 及其不可变摘要。对于使用本地前端（如 Dockerfile）的
构建，`entryPoint` 字段表示初始化构建的前端文件路径（对应 `filename`
前端选项）。

### `invocation.parameters`

* 参考：https://slsa.dev/spec/v0.2/provenance#invocation.parameters
* `mode=min` 下部分包含。

描述传入构建的输入。

```json
    "invocation": {
      "parameters": {
        "frontend": "gateway.v0",
        "args": {
          "build-arg:BUILDKIT_CONTEXT_KEEP_GIT_DIR": "1",
          "label:FOO": "bar",
          "source": "docker/dockerfile-upstream:master",
          "target": "release"
        },
        "secrets": [
          {
            "id": "GIT_AUTH_HEADER",
            "optional": true
          },
          ...
        ],
        "ssh": [],
        "locals": []
      },
      ...
    },
```

以下字段在 `mode=min` 与 `mode=max` 中均会包含：

- `locals`：列出构建中使用的本地来源，包括构建上下文与前端文件。
- `frontend`：构建所用的 BuildKit 前端类型。目前可能为 `dockerfile.v0` 或
  `gateway.v0`。
- `args`：传入 BuildKit 前端的构建参数。

  `args` 对象中的键与 BuildKit 接收到的选项一致。例如，`build-arg` 与
  `label` 前缀分别用于构建参数与标签，`target` 指定构建的目标阶段；若使用
  Gateway 前端，`source` 指定其来源镜像。

以下字段仅在 `mode=max` 中包含：

- `secrets`：构建过程中使用的机密（不会包含机密的实际值）。
- `ssh`：构建过程中使用的 SSH 转发。

### `invocation.environment`

* 参考：https://slsa.dev/spec/v0.2/provenance#invocation.environment
* `mode=min` 与 `mode=max` 均包含。

```json
    "invocation": {
      "environment": {
        "platform": "linux/amd64"
      },
      ...
    },
```

BuildKit 当前仅设置当前构建机的 `platform`。注意，这不一定等同于构建结果的
平台；构建结果的平台可从 `in-toto` 的 subject 字段确定。

### `materials`

* 参考：https://slsa.dev/spec/v0.2/provenance#materials
* `mode=min` 与 `mode=max` 均包含。

定义构建所涉及的所有外部工件。取值取决于工件类型：

- 包含镜像源码的 Git 仓库 URL
- 远程 tar 包的 HTTP URL，或通过 Dockerfile 的 `ADD` 指令包含的 URL
- 构建期间使用的任意 Docker 镜像

Docker 镜像的 URL 采用
[Package URL](https://github.com/package-url/purl-spec) 格式。

所有构建材料都会包含不可变的校验和。若从可变标签构建，你可以通过摘要信息来
判断该工件相较于构建时是否已被更新。

```json
    "materials": [
      {
        "uri": "pkg:docker/alpine@3.17?platform=linux%2Famd64",
        "digest": {
          "sha256": "8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4"
        }
      },
      {
        "uri": "https://github.com/moby/buildkit.git#refs/tags/v0.11.0",
        "digest": {
          "sha1": "4b220de5058abfd01ff619c9d2ff6b09a049bea0"
        }
      },
      ...
    ],
```

### `buildConfig`

* 参考：https://slsa.dev/spec/v0.2/provenance#buildConfig
* 仅在 `mode=max` 中包含。

定义构建期间执行的构建步骤。

BuildKit 内部使用 LLB 定义来执行构建步骤。构建步骤的 LLB 定义位于
`buildConfig.llbDefinition` 字段中。

每个 LLB 步骤是
[LLB ProtoBuf API](https://github.com/moby/buildkit/blob/v0.10.0/solver/pb/ops.proto)
的 JSON 表示。LLB 图中每个顶点的依赖可在该步骤的 `inputs` 字段中找到。

```json
  "buildConfig": {
    "llbDefinition": [
      {
        "id": "step0",
        "op": {
          "Op": {
            "exec": {
              "meta": {
                "args": [
                  "/bin/sh",
                  "-c",
                  "go build ."
                ],
                "env": [
                  "PATH=/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                  "GOPATH=/go",
                  "GOFLAGS=-mod=vendor",
                ],
                "cwd": "/src",
              },
              "mounts": [...]
            }
          },
          "platform": {...},
        },
        "inputs": [
          "step8:0",
          "step2:0",
        ]
      },
      ...
    ]
  },
```

### `metadata.buildInvocationId`

* 参考：https://slsa.dev/spec/v0.2/provenance#buildInvocationId
* `mode=min` 与 `mode=max` 均包含。

构建调用的唯一标识符。对单次请求构建多平台镜像时，该值会被该镜像的所有平台
版本共享。

```json
    "metadata": {
      "buildInvocationID": "rpv7a389uzil5lqmrgwhijwjz",
      ...
    },
```

### `metadata.buildStartedOn`

* 参考：https://slsa.dev/spec/v0.2/provenance#buildStartedOn
* `mode=min` 与 `mode=max` 均包含。

构建开始的时间戳。

```json
    "metadata": {
      "buildStartedOn": "2021-11-17T15:00:00Z",
      ...
    },
```

### `metadata.buildFinishedOn`

* 参考：https://slsa.dev/spec/v0.2/provenance#buildFinishedOn
* `mode=min` 与 `mode=max` 均包含。

构建结束的时间戳。

```json
    "metadata": {
      "buildFinishedOn": "2021-11-17T15:01:00Z",
      ...
    },
```

### `metadata.completeness`

* 参考：https://slsa.dev/spec/v0.2/provenance#metadata.completeness
* `mode=min` 与 `mode=max` 均包含。

定义溯源信息是否完整。

当所有构建参数都包含在 `parameters` 字段中时，`completeness.parameters` 为
true。在 `min` 模式下，构建参数不会包含在溯源信息中，因此该字段为 false。
未使用前端的直接 LLB 构建时，parameters 也不完整。

对于 BuildKit 构建，`completeness.environment` 总为 true。

当 `materials` 包含构建的所有依赖时，`completeness.materials` 为 true。
若从本地目录的未追踪来源构建，则 materials 不完整；若从远程 Git 仓库
构建，BuildKit 可追踪所有依赖，此字段为 true。

```json
    "metadata": {
      "completeness": {
        "parameters": true,
        "environment": true,
        "materials": true
      },
      ...
    },
```

### `metadata.reproducible`

* 参考：https://slsa.dev/spec/v0.2/provenance#metadata.reproducible
* `mode=min` 与 `mode=max` 均包含。

定义构建结果是否应当达到字节级可复现。可通过证明参数
`reproducible=true` 设置该值。

```json
    "metadata": {
      "reproducible": false,
      ...
    },
```

### `metadata.https://mobyproject.org/buildkit@v1#hermetic`

* `mode=min` 与 `mode=max` 均包含。

如果构建是密闭（hermetic）的且未访问网络，则该扩展字段为 true。在 Dockerfile
中，若未使用 `RUN` 命令，或通过 `--network=none` 禁用了网络，则该构建为
hermetic。

```json
    "metadata": {
      "https://mobyproject.org/buildkit@v1#hermetic": true,
      ...
    },
```

### `metadata.https://mobyproject.org/buildkit@v1#metadata`

* `mode=min` 下部分包含。

该扩展字段定义 BuildKit 特有、非 SLSA 规范的一些附加元数据。

```json
    "metadata": {
      "https://mobyproject.org/buildkit@v1#metadata": {
        "source": {...},
        "layers": {...},
        "vcs": {...},
      },
      ...
    },
```

#### `source`

仅在 `mode=max` 中包含。

定义 LLB 构建步骤（定义于 `buildConfig.llbDefinition`）到其原始源码（如
Dockerfile 命令）的映射。`source.locations` 包含每个 LLB 步骤中运行的
Dockerfile 命令的行范围；`source.infos` 数组包含源码本身。若前端在生成 LLB
定义时提供了映射，则会包含该映射。

#### `layers`

仅在 `mode=max` 中包含。

定义 LLB 构建步骤的挂载与等效层的 OCI 描述符之间的映射（对应
`buildConfig.llbDefinition`）。当存在层数据时（通常针对镜像证明，或构建步
骤在构建过程中拉取了镜像数据），该映射可用。

#### `vcs`

`mode=min` 与 `mode=max` 均包含。

定义构建所用版本控制系统的可选元数据。若构建使用来自 Git 仓库的远程上下文，
BuildKit 会自动提取详细信息并显示在 `invocation.configSource` 中。若构建使
用本地目录作为来源，即便该目录包含 Git 仓库，VCS 信息也会丢失。在此情况下，
构建客户端可以发送额外的 `vcs:source` 与 `vcs:revision` 构建选项，BuildKit
会将其作为额外元数据添加到溯源证明中。注意，与 `invocation.configSource`
不同，BuildKit 不会校验 `vcs` 的取值，因此它们不应被信任，仅可作为元数据
提示。


