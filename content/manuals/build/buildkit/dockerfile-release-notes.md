---
title: Dockerfile 发布说明
description: Dockerfile 前端的发布说明
keywords: build, dockerfile, frontend, release notes
tags: [Release notes]
toc_max: 2
aliases:
  - /build/dockerfile/release-notes/
---

本页汇总了 [Dockerfile 参考](/reference/dockerfile.md) 中的新特性、改进、已知问题与缺陷修复信息。

用法请参见 [Dockerfile 前端语法](frontend.md)。

## 1.19.0

{{< release-date date="2025-09-30" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.19.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.19.0
```

* `COPY` 与 `ADD` 指令的 `--exclude` 标志现已正式可用。此前该标志仅在 `labs` 渠道中提供。 [moby/buildkit#6232](https://github.com/moby/buildkit/pull/6232)
* 修复在 `COPY` 添加 `--exclude` 标志时，可能因断开的符号链接导致构建失败的问题。 [moby/buildkit#6220](https://github.com/moby/buildkit/pull/6220)
* 修复 `EXPOSE` 指令未正确格式化其创建的历史记录条目的问题。 [moby/buildkit#6218](https://github.com/moby/buildkit/pull/6218)

## 1.18.0

{{< release-date date="2025-09-03" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.18.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.18.0
```

* 为远程构建上下文增加对 Git URL 的支持，且 `ADD` 指令现允许使用带查询参数的全新语法（`?key=value`）以更精细地控制 Git 克隆过程。本次发布支持：`ref`、`tag`、`branch`、`checksum`（别名 `commit`）、`subdir`、`keep-git-dir` 与 `submodules`。 [moby/buildkit#6172](https://github.com/moby/buildkit/pull/6172) [moby/buildkit#6173](https://github.com/moby/buildkit/pull/6173)
* 新增检查规则 `ExposeProtoCasing` 与 `ExposeInvalidFormat`，以改进 `EXPOSE` 指令的使用。 [moby/buildkit#6135](https://github.com/moby/buildkit/pull/6135)
* 修复在使用命名上下文时，未能从基础镜像正确继承创建时间的问题。 [moby/buildkit#6096](https://github.com/moby/buildkit/pull/6096)

## 1.17.0

{{< release-date date="2025-06-17" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.17.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.17.0
```

* 新增 `ADD --unpack=bool`，用于控制是否解包来自 URL 路径的归档。默认行为仍为根据源路径自动判断是否解包，与之前版本保持一致。 [moby/buildkit#5991](https://github.com/moby/buildkit/pull/5991)
* 新增在解包归档时支持 `ADD --chown`，与复制常规文件时的行为一致。 [moby/buildkit#5987](https://github.com/moby/buildkit/pull/5987)

## 1.16.0

{{< release-date date="2025-05-22" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.16.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.16.0
```

* 为 Git URL 增加 `ADD --checksum` 支持。 [moby/buildkit#5975](https://github.com/moby/buildkit/pull/5975)
* 允许在 heredoc 中使用空白字符。 [moby/buildkit#5817](https://github.com/moby/buildkit/pull/5817)
* `WORKDIR` 现支持 `SOURCE_DATE_EPOCH`。 [moby/buildkit#5960](https://github.com/moby/buildkit/pull/5960)
* 对于 Windows 容器，保留基础镜像设置的默认 PATH。 [moby/buildkit#5895](https://github.com/moby/buildkit/pull/5895)

## 1.15.1

{{< release-date date="2025-03-30" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.15.1) 查看。

```dockerfile
# syntax=docker/dockerfile:1.15.1
```

* 修复当使用 `--attest type=sbom` 时出现的 `no scan targets for linux/arm64/v8` 问题。 [moby/buildkit#5941](https://github.com/moby/buildkit/pull/5941)

## 1.15.0

{{< release-date date="2025-04-15" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.15.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.15.0
```

- 当目标无效时，构建错误信息现在会给出可能的正确名称建议。 [moby/buildkit#5851](https://github.com/moby/buildkit/pull/5851)
- 修复 Windows 目标的 SBOM 证明错误。 [moby/buildkit#5837](https://github.com/moby/buildkit/pull/5837)
- 修复处理 outline 请求时，递归 `ARG` 可能导致无限循环的问题。 [moby/buildkit#5823](https://github.com/moby/buildkit/pull/5823)
- 修复从 JSON 解析语法指令时，如果包含非字符串类型会失败的问题。 [moby/buildkit#5815](https://github.com/moby/buildkit/pull/5815)
- 修复镜像配置中的平台字段未规范化（1.12 回归）。 [moby/buildkit#5776](https://github.com/moby/buildkit/pull/5776)
- 修复在 WCOW 上目标目录不存在时复制到目录的问题。 [moby/buildkit#5249](https://github.com/moby/buildkit/pull/5249)

## 1.14.1

{{< release-date date="2025-03-05" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.14.1) 查看。

```dockerfile
# syntax=docker/dockerfile:1.14.1
```

- 规范化镜像配置中的平台字段。 [moby/buildkit#5776](https://github.com/moby/buildkit/pull/5776)

## 1.14.0

{{< release-date date="2025-02-19" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.14.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.14.0
```

- `COPY --chmod` 现允许使用非八进制值。该特性此前位于 labs 渠道，现已合入主发布。 [moby/buildkit#5734](https://github.com/moby/buildkit/pull/5734)
- 修复当基础镜像设置了 OSVersion 属性时的处理问题。 [moby/buildkit#5714](https://github.com/moby/buildkit/pull/5714)
- 修复命名上下文元数据在当前构建配置不可达时仍被解析，导致构建错误的问题。 [moby/buildkit#5688](https://github.com/moby/buildkit/pull/5688)

## 1.14.0（labs）

{{< release-date date="2025-02-19" >}}

{{% include "dockerfile-labs-channel.md" %}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.14.0-labs) 查看。

```dockerfile
# syntax=docker.io/docker/dockerfile-upstream:1.14.0-labs
```

- 新增 `RUN --device=name,[required]` 标志，允许构建请求在步骤中使用 CDI 设备。需要 BuildKit v0.20.0+。 [moby/buildkit#4056](https://github.com/moby/buildkit/pull/4056), [moby/buildkit#5738](https://github.com/moby/buildkit/pull/5738)

## 1.13.0

{{< release-date date="2025-01-20" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.13.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.13.0
```

- 新增内置构建参数 `TARGETOSVERSION`、`BUILDOSVERSION`（用于 Windows 构建）；且 `TARGETPLATFORM` 的值现在也包含 `OSVersion`。 [moby/buildkit#5614](https://github.com/moby/buildkit/pull/5614)
- 允许带 BOM（字节序标记）的文件将语法转发到外部前端。 [moby/buildkit#5645](https://github.com/moby/buildkit/pull/5645)
- Windows 容器的默认 PATH 更新以包含 `powershell.exe` 目录。 [moby/buildkit#5446](https://github.com/moby/buildkit/pull/5446)
- 修复 Dockerfile 指令解析，避免无效语法被接受。 [moby/buildkit#5646](https://github.com/moby/buildkit/pull/5646)
- 修复 `ONBUILD` 命令在继承阶段可能运行两次的问题。 [moby/buildkit#5593](https://github.com/moby/buildkit/pull/5593)
- 修复可能遗漏子阶段命名上下文替换的问题。 [moby/buildkit#5596](https://github.com/moby/buildkit/pull/5596)

## 1.13.0（labs）

{{< release-date date="2025-01-20" >}}

{{% include "dockerfile-labs-channel.md" %}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.13.0-labs) 查看。

```dockerfile
# syntax=docker.io/docker/dockerfile-upstream:1.13.0-labs
```

- 修复 `COPY --chmod` 对非八进制值的支持。 [moby/buildkit#5626](https://github.com/moby/buildkit/pull/5626)

## 1.12.0

{{< release-date date="2024-11-27" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.12.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.12.0
```

- 修复当存在多个 `ARG` 指令时，镜像配置 History 行描述不正确的问题。 [moby/buildkit#5508]

[moby/buildkit#5508]: https://github.com/moby/buildkit/pull/5508

## 1.11.1

{{< release-date date="2024-11-08" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.11.1) 查看。

```dockerfile
# syntax=docker/dockerfile:1.11.1
```

- 修复在同一 Dockerfile 内继承阶段使用 `ONBUILD` 指令时的回归问题。 [moby/buildkit#5490]

[moby/buildkit#5490]: https://github.com/moby/buildkit/pull/5490

## 1.11.0

{{< release-date date="2024-10-30" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.11.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.11.0
```

- [`ONBUILD` 指令](/reference/dockerfile.md#onbuild) 现在支持引用其他阶段或镜像的命令（如 `COPY --from` 或 `RUN mount=from=...`）。 [moby/buildkit#5357]
- 改进 [`SecretsUsedInArgOrEnv`](/reference/build-checks/secrets-used-in-arg-or-env.md) 构建检查，减少误报。 [moby/buildkit#5208]
- 新增 [`InvalidDefinitionDescription`](/reference/build-checks/invalid-definition-description.md) 构建检查，建议为构建参数与阶段描述的注释进行格式化。该检查为[实验特性](/manuals/build/checks.md#experimental-checks)。 [moby/buildkit#5208], [moby/buildkit#5414]
- 多项 `ONBUILD` 指令在进度与错误处理上的修复。 [moby/buildkit#5397]
- 改进缺失标志位时的错误报告。 [moby/buildkit#5369]
- 增强将机密值作为环境变量挂载时的进度输出。 [moby/buildkit#5336]
- 新增内置构建参数 `TARGETSTAGE`，用于暴露当前构建（最终）目标阶段的名称。 [moby/buildkit#5431]

## 1.11.0（labs）

{{% include "dockerfile-labs-channel.md" %}}

- `COPY --chmod` 现在支持非八进制值。 [moby/buildkit#5380]

[moby/buildkit#5357]: https://github.com/moby/buildkit/pull/5357
[moby/buildkit#5208]: https://github.com/moby/buildkit/pull/5208
[moby/buildkit#5414]: https://github.com/moby/buildkit/pull/5414
[moby/buildkit#5397]: https://github.com/moby/buildkit/pull/5397
[moby/buildkit#5369]: https://github.com/moby/buildkit/pull/5369
[moby/buildkit#5336]: https://github.com/moby/buildkit/pull/5336
[moby/buildkit#5431]: https://github.com/moby/buildkit/pull/5431
[moby/buildkit#5380]: https://github.com/moby/buildkit/pull/5380

## 1.10.0

{{< release-date date="2024-09-10" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.10.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.10.0
```

- [构建机密](/manuals/build/building/secrets.md#target) 现可通过 `env=VARIABLE` 选项以环境变量形式挂载。 [moby/buildkit#5215]
- [`# check` 指令](/reference/dockerfile.md#check) 现支持新的实验性属性，用于启用如 `CopyIgnoredFile` 等实验性校验规则。 [moby/buildkit#5213]
- 改进对不支持的变量替换修饰符的校验。 [moby/buildkit#5146]
- `ADD` 与 `COPY` 指令现支持对 `--chmod` 选项值进行构建参数变量插值。 [moby/buildkit#5151]
- 改进对 `COPY` 与 `ADD` 指令 `--chmod` 选项的校验。 [moby/buildkit#5148]
- 修复挂载属性中 size 与 destination 的自动补全缺失问题。 [moby/buildkit#5245]
- 现在会为 Dockerfile 前端发布镜像设置 OCI 注解。 [moby/buildkit#5197]

[moby/buildkit#5215]: https://github.com/moby/buildkit/pull/5215
[moby/buildkit#5213]: https://github.com/moby/buildkit/pull/5213
[moby/buildkit#5146]: https://github.com/moby/buildkit/pull/5146
[moby/buildkit#5151]: https://github.com/moby/buildkit/pull/5151
[moby/buildkit#5148]: https://github.com/moby/buildkit/pull/5148
[moby/buildkit#5245]: https://github.com/moby/buildkit/pull/5245
[moby/buildkit#5197]: https://github.com/moby/buildkit/pull/5197

## 1.9.0

{{< release-date date="2024-07-11" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.9.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.9.0
```

- 新增校验规则：
  - `SecretsUsedInArgOrEnv`
  - `InvalidDefaultArgInFrom`
  - `RedundantTargetPlatform`
  - `CopyIgnoredFile`（实验性）
  - `FromPlatformFlagConstDisallowed`
- 对处理大型 Dockerfile 的性能进行了多项优化。 [moby/buildkit#5067](https://github.com/moby/buildkit/pull/5067/), [moby/buildkit#5029](https://github.com/moby/buildkit/pull/5029/)
- 修复当 Dockerfile 未定义任何阶段时可能触发的 panic。 [moby/buildkit#5150](https://github.com/moby/buildkit/pull/5150/)
- 修复不正确的 JSON 解析问题，避免部分错误的 JSON 值未报错即通过。 [moby/buildkit#5107](https://github.com/moby/buildkit/pull/5107/)
- 修复回归问题：当 `COPY --link` 的目标路径为 `.` 时可能失败。 [moby/buildkit#5080](https://github.com/moby/buildkit/pull/5080/)
- 修复当与 Git URL 配合使用时，对 `ADD --checksum` 的校验。 [moby/buildkit#5085](https://github.com/moby/buildkit/pull/5085/)

## 1.8.1

{{< release-date date="2024-06-18" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.8.1) 查看。

```dockerfile
# syntax=docker/dockerfile:1.8.1
```

### 缺陷修复与增强

- 修复变量展开中对空字符串的处理。 [moby/buildkit#5052](https://github.com/moby/buildkit/pull/5052/)
- 改进构建警告的输出格式。 [moby/buildkit#5037](https://github.com/moby/buildkit/pull/5037/), [moby/buildkit#5045](https://github.com/moby/buildkit/pull/5045/), [moby/buildkit#5046](https://github.com/moby/buildkit/pull/5046/)
- 修复多阶段构建中 `UndeclaredVariable` 警告可能输出无效内容的问题。 [moby/buildkit#5048](https://github.com/moby/buildkit/pull/5048/)

## 1.8.0

{{< release-date date="2024-06-11" >}}

此版本的完整发布说明可在
[GitHub](https://github.com/moby/buildkit/releases/tag/dockerfile%2F1.8.0) 查看。

```dockerfile
# syntax=docker/dockerfile:1.8.0
```

- 新增大量校验规则，用于确保 Dockerfile 符合最佳实践。这些规则会在构建时进行校验；新的 `check` 前端方法也可用于仅触发校验而不完成整个构建。
- 新增 `#check` 指令与构建参数 `BUILDKIT_DOCKERFILE_CHECK`，用于控制构建校验行为。 [moby/buildkit#4962](https://github.com/moby/buildkit/pull/4962/)
- 现会校验：当使用与期望平台不匹配的单平台基础镜像时给出提示。 [moby/buildkit#4924](https://github.com/moby/buildkit/pull/4924/)
- 正确处理在全局作用域中展开 `ARG` 定义时的错误。 [moby/buildkit#4856](https://github.com/moby/buildkit/pull/4856/)
- 仅在用户未覆盖时才展开 `ARG` 的默认值。此前会先展开后再忽略，可能导致意外的展开错误。 [moby/buildkit#4856](https://github.com/moby/buildkit/pull/4856/)
- 提升对包含大量阶段的大型 Dockerfile 的解析性能。 [moby/buildkit#4970](https://github.com/moby/buildkit/pull/4970/)
- 修复部分 Windows 路径处理一致性问题。 [moby/buildkit#4825](https://github.com/moby/buildkit/pull/4825/)

## 1.7.0

{{< release-date date="2024-03-06" >}}

### 稳定版

```dockerfile
# syntax=docker/dockerfile:1.7
```

- 变量展开现支持字符串替换与截断。
  [moby/buildkit#4427](https://github.com/moby/buildkit/pull/4427),
  [moby/buildkit#4287](https://github.com/moby/buildkit/pull/4287)
- 对本地来源的命名上下文，现在仅传输 Dockerfile 实际使用的文件，而非整个源目录。
  [moby/buildkit#4161](https://github.com/moby/buildkit/pull/4161)
- Dockerfile 现在会更好地校验阶段顺序，并在顺序不正确时反馈带堆栈的友好错误。
  [moby/buildkit#4568](https://github.com/moby/buildkit/pull/4568),
  [moby/buildkit#4567](https://github.com/moby/buildkit/pull/4567)
- 历史记录中的提交消息现在包含 `COPY` 与 `ADD` 所使用的标志。 [moby/buildkit#4597](https://github.com/moby/buildkit/pull/4597)
- 改进从 Git 与 HTTP 源执行 `ADD` 命令时的进度输出。 [moby/buildkit#4408](https://github.com/moby/buildkit/pull/4408)

### Labs

```dockerfile
# syntax=docker/dockerfile:1.7-labs
```

- 新增 `--parents` 标志到 `COPY`，可在复制文件时保留父目录结构。
  [moby/buildkit#4598](https://github.com/moby/buildkit/pull/4598),
  [moby/buildkit#3001](https://github.com/moby/buildkit/pull/3001),
  [moby/buildkit#4720](https://github.com/moby/buildkit/pull/4720),
  [moby/buildkit#4728](https://github.com/moby/buildkit/pull/4728),
  [文档](/reference/dockerfile.md#copy---parents)
- 新增 `--exclude` 标志，可在 `COPY` 与 `ADD` 命令中过滤被复制的文件。
  [moby/buildkit#4561](https://github.com/moby/buildkit/pull/4561),
  [文档](/reference/dockerfile.md#copy---exclude)

## 1.6.0

{{< release-date date="2023-06-13" >}}

### 新增

- 为 [`HEALTHCHECK` 指令](/reference/dockerfile.md#healthcheck) 增加 `--start-interval` 标志。

以下特性已从 labs 渠道毕业至稳定版：

- `ADD` 指令现在可以[直接从 Git URL 导入文件](/reference/dockerfile.md#adding-a-git-repository-add-git-ref-dir)
- `ADD` 指令现在支持 [`--checksum` 标志](/reference/dockerfile.md#verifying-a-remote-file-checksum-add---checksumchecksum-http-src-dest)
  以校验远程 URL 内容

### 缺陷修复与增强

- 变量替换现在支持更多不带 `:` 的 POSIX 兼容变体。
  [moby/buildkit#3611](https://github.com/moby/buildkit/pull/3611)
- 导出的 Windows 镜像现在包含基础镜像中的 OSVersion 与 OSFeatures 值。
  [moby/buildkit#3619](https://github.com/moby/buildkit/pull/3619)
- 将 Heredocs 的默认权限调整为 0644。
  [moby/buildkit#3992](https://github.com/moby/buildkit/pull/3992)

## 1.5.2

{{< release-date date="2023-02-14" >}}

### 缺陷修复与增强

- 修复当 Git 引用缺少分支名但包含子目录（subdir）时的构建问题。
- 386 平台镜像现已包含在发行包中。

## 1.5.1

{{< release-date date="2023-01-18" >}}

### 缺陷修复与增强

- 修复在多平台构建中，当出现警告条件时可能触发的 panic。

## 1.5.0（labs）

{{< release-date date="2023-01-10" >}}

{{% include "dockerfile-labs-channel.md" %}}

### 新增

- `ADD` 命令现支持 [`--checksum` 标志](/reference/dockerfile.md#verifying-a-remote-file-checksum-add---checksumchecksum-http-src-dest)，
  以校验远程 URL 内容。

## 1.5.0

{{< release-date date="2023-01-10" >}}

### 新增

- `ADD` 命令现在可以[直接从 Git URL 导入文件](/reference/dockerfile.md#adding-a-git-repository-add-git-ref-dir)

### 缺陷修复与增强

- 命名上下文现在支持 `oci-layout://` 协议，可从本地 OCI 布局结构中包含镜像。
- Dockerfile 现在支持二次请求以列出所有构建目标，或打印某个构建目标可接受参数的概要。
- Dockerfile 的 `#syntax` 指令（用于重定向到外部前端镜像）现在也允许通过 `//` 注释或 JSON 设置；文件也可以包含 shebang 头部。
- 命名上下文现在可以初始化为空的 scratch 镜像。
- 命名上下文现在可以使用 SSH Git URL 初始化。
- 修复从 Schema1 镜像导入时的 `ONBUILD` 处理。

## 1.4.3

{{< release-date date="2022-08-23" >}}

### 缺陷修复与增强

- 修复当从 `docker-image://` 命名上下文构建镜像时，创建时间戳未重置的问题。
- 修复当从 `docker-image://` 命名上下文加载时，`FROM` 的 `--platform` 标志传递问题。

## 1.4.2

{{< release-date date="2022-05-06" >}}

### 缺陷修复与增强

- 修复当通过构建上下文传入镜像时，加载某些环境变量的问题。

## 1.4.1

{{< release-date date="2022-04-08" >}}

### 缺陷修复与增强

- 修复当输入为不同平台构建时，跨编译场景的命名上下文解析。

## 1.4.0

{{< release-date date="2022-03-09" >}}

### 新增

- [`COPY --link` 与 `ADD --link`](/reference/dockerfile.md#copy---link)
  允许以更高的缓存效率复制文件，并在无需重建的情况下重构（rebase）镜像。`--link` 会将文件复制到独立层，随后通过新的 LLB MergeOp 将相互独立的层链接在一起。
- [Heredocs](/reference/dockerfile.md#here-documents) 从 labs 渠道毕业至稳定版。该特性允许编写多行的内联脚本与文件。
- 可向构建传入额外的[命名构建上下文](/reference/cli/docker/buildx/build.md#build-context)，以添加或覆盖构建中的阶段或镜像。上下文来源可以是本地、镜像、Git 或 HTTP URL。
- [`BUILDKIT_SANDBOX_HOSTNAME` 构建参数](/reference/dockerfile.md#buildkit-built-in-build-args) 可用于为 `RUN` 步骤设置默认主机名。

### 缺陷修复与增强

- 当使用跨编译阶段时，进度输出中现在会显示步骤的目标平台。
- 修复某些情况下 Heredocs 错误去除内容引号的问题。

## 1.3.1

{{< release-date date="2021-10-04" >}}

### 缺陷修复与增强

- 修复解析不带值的 `required` 挂载键。

## 1.3.0（labs）

{{< release-date date="2021-07-16" >}}

{{% include "dockerfile-labs-channel.md" %}}

### 新增

- 现已支持在 `RUN` 与 `COPY` 命令中使用[Heredoc 语法](/reference/dockerfile.md#here-documents)，
  以编写多行内联脚本与文件。

## 1.3.0

{{< release-date date="2021-07-16" >}}

### 新增

- `RUN` 命令允许使用 [`--network` 标志](/reference/dockerfile.md#run---network)，以请求特定的网络条件。`--network=host` 需要允许 `network.host` 权限。此前该特性仅在 labs 渠道提供。

### 缺陷修复与增强

- 现在，带远程 URL 的 `ADD` 命令可以正确处理 `--chmod` 标志。
- [`RUN --mount` 标志](/reference/dockerfile.md#run---mount) 的值现支持变量展开（`from` 字段除外）。
- 允许使用 [`BUILDKIT_MULTI_PLATFORM` 构建参数](/reference/dockerfile.md#buildkit-built-in-build-args) 强制始终创建多平台镜像，即使仅包含单一平台。

## 1.2.1（labs）

{{< release-date date="2020-12-12" >}}

{{% include "dockerfile-labs-channel.md" %}}

### 缺陷修复与增强

- `RUN` 命令允许使用 [`--network` 标志](/reference/dockerfile.md#run---network) 请求特定网络条件。`--network=host` 需要允许 `network.host` 权限。

## 1.2.1

{{< release-date date="2020-12-12" >}}

### 缺陷修复与增强

- 回滚 “确保 ENTRYPOINT 至少包含一个参数”。
- 优化在多平台交叉编译构建中处理 `COPY` 调用。

## 1.2.0（labs）

{{< release-date date="2020-12-03" >}}

{{% include "dockerfile-labs-channel.md" %}}

### 缺陷修复与增强

- 将实验渠道重命名为 labs。

## 1.2.0

{{< release-date date="2020-12-03" >}}

### 新增

- 用于创建 secret、ssh、bind 与 cache 挂载的 [`RUN --mount` 语法](/reference/dockerfile.md#run---mount) 已合入主发布。
- [`ARG` 命令](/reference/dockerfile.md#arg) 现在支持在一行中定义多个构建参数，类似 `ENV`。

### 缺陷修复与增强

- 现在会将元数据加载错误视为致命错误，以避免产生错误的构建结果。
- 允许 Dockerfile 名称使用小写。
- `ADD` 的 `--chown` 标志现在允许参数展开。
- `ENTRYPOINT` 至少需要一个参数，以避免创建坏镜像。

## 1.1.7

{{< release-date date="2020-04-18" >}}

### 缺陷修复与增强

- 将 `FrontendInputs` 转发至网关。

## 1.1.2（labs）

{{< release-date date="2019-07-31" >}}

{{% include "dockerfile-labs-channel.md" %}}

### 缺陷修复与增强

- 允许使用 `RUN --security=sandbox|insecure` 为进程设置安全模式。
- 允许为[缓存挂载](/reference/dockerfile.md#run---mounttypecache)设置 uid/gid。
- 避免请求内部链接路径被拉取到构建上下文。
- 确保缺失的缓存 ID 默认指向目标路径。
- 允许通过 [`BUILDKIT_CACHE_MOUNT_NS` 构建参数](/reference/dockerfile.md#buildkit-built-in-build-args) 为缓存挂载设置命名空间。

## 1.1.2

{{< release-date date="2019-07-31" >}}

### 缺陷修复与增强

- 修复以正确的用户创建工作目录，并且不重置自定义所有权。
- 修复同时作为 `ENV` 使用的空构建参数的处理。
- 检测循环依赖。

## 1.1.0

{{< release-date date="2019-04-27" >}}

### 新增

- `ADD/COPY` 命令现支持基于 `llb.FileOp` 的实现，若内置文件操作可用，则无需 helper 镜像。
- `COPY` 命令的 `--chown` 标志现在支持变量展开。

### 缺陷修复与增强

- 为查找从构建上下文中忽略的文件，Dockerfile 前端会优先查找文件 `<path/to/Dockerfile>.dockerignore`；若未找到，则回退到构建上下文根目录的 `.dockerignore`。这允许在包含多个 Dockerfile 的项目中使用不同的 `.dockerignore` 定义。
