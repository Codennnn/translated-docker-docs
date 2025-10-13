---
title: 本地与 tar 导出器
keywords: build, buildx, buildkit, exporter, local, tar
description: >
  本地与 tar 导出器会将构建结果保存到本地文件系统
aliases:
  - /build/building/exporters/local-tar/
---

`local` 与 `tar` 导出器会将构建结果的根文件系统输出到本地目录。
它们适合用于生成不是容器镜像的构建产物。

- `local` 会导出文件与目录。
- `tar` 导出相同的内容，但会将导出物封装为一个 tar 包。

## 用法概览

使用 `local` 与 `tar` 导出器构建：

```console
$ docker buildx build --output type=local[,parameters] .
$ docker buildx build --output type=tar[,parameters] .
```

下表描述了可用参数：

| 参数              | 类型    | 默认值  | 说明                                                                                                                                                                                                                                   |
|------------------|---------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dest`           | String  |         | 要复制文件到的目标路径。                                                                                                                                                                                                               |
| `platform-split` | Boolean | `true`  | 当在多平台构建中使用本地导出器时，默认会在目标目录下为每个目标平台创建一个同名子目录。将其设为 `false` 可将所有平台的文件合并到同一目录。                                                                                               |

## 延伸阅读

关于 `local` 与 `tar` 导出器的更多信息，参见
[BuildKit README](https://github.com/moby/buildkit/blob/master/README.md#local-directory)。
