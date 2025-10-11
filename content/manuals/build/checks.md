---
title: 校验你的构建配置
linkTitle: 构建校验
params:
  sidebar:
    badge:
      color: green
      text: New
weight: 30
description: 了解如何使用构建校验来验证你的构建配置。
keywords: build, buildx, buildkit, checks, validate, configuration, lint
---

{{< summary-bar feature_name="Build checks" >}}

构建校验是 Dockerfile 1.8 引入的特性。它允许你在执行构建之前验证构建配置并进行一系列检查。
可以把它理解为 Dockerfile 与构建选项的“高级 Lint”，或者构建的“预演（dry-run）模式”。

所有可用校验及其说明，参见：[构建校验参考](/reference/build-checks/)。

## 构建校验的工作机制

通常情况下，构建会按 Dockerfile 与构建选项执行具体步骤。
启用构建校验后，Docker 不直接执行构建步骤，而是先对 Dockerfile 与构建选项进行检查，并报告发现的问题。

构建校验的用途包括：

- 在构建前验证 Dockerfile 与构建选项
- 确保你的 Dockerfile 与构建选项符合最新最佳实践
- 发现 Dockerfile 与构建选项中的潜在问题或反模式

> [!TIP]
>
> 想在 VS Code 中改进对 Dockerfile 的 Lint、代码导航与漏洞扫描体验，
> 可参考 [Docker VS Code 扩展](https://marketplace.visualstudio.com/items?itemName=docker.docker)。

## 在构建中启用校验

构建校验适用于：

- Buildx 0.15.0 及更高版本
- [docker/build-push-action](https://github.com/docker/build-push-action) 6.6.0 及更高版本
- [docker/bake-action](https://github.com/docker/bake-action) 5.6.0 及更高版本

默认情况下，执行构建会同时运行校验，并在构建输出中显示任何违规信息。
例如，以下命令会同时构建镜像并运行校验：

```console
$ docker build .
[+] Building 3.5s (11/11) FINISHED
...

1 warning found (use --debug to expand):
  - Lint Rule 'JSONArgsRecommended': JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 7)

```

该示例中，构建成功，但报告了一个
[JSONArgsRecommended](/reference/build-checks/json-args-recommended/) 警告，
原因是 `CMD` 指令应使用 JSON 数组语法。

在 GitHub Actions 中，校验结果会显示在 Pull Request 的差异视图里。

```yaml
name: Build and push Docker images
on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build and push
        uses: docker/build-push-action@v6.6.0
```

![GitHub Actions build check annotations](./images/gha-check-annotations.png)

### 更详细的输出

常规 `docker build` 的校验警告会给出简要信息：规则名称、提示信息，以及问题在 Dockerfile 中的大致行号。
如需查看更多细节，可使用 `--debug` 标志。例如：

```console
$ docker --debug build .
[+] Building 3.5s (11/11) FINISHED
...

 1 warning found:
 - JSONArgsRecommended: JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 4)
JSON arguments recommended for ENTRYPOINT/CMD to prevent unintended behavior related to OS signals
More info: https://docs.docker.com/go/dockerfile/rule/json-args-recommended/
Dockerfile:4
--------------------
   2 |
   3 |     FROM alpine
   4 | >>> CMD echo "Hello, world!"
   5 |
--------------------

```

使用 `--debug` 后，输出会包含该校验的文档链接，以及命中问题的 Dockerfile 片段。

## 仅运行校验而不执行构建

如果只想运行校验、不执行实际构建，可以在常规 `docker build` 命令中增加 `--check` 标志：

```console
$ docker build --check .
```

该命令不会执行构建步骤，只会运行校验并报告发现的问题。例如：

```text {title="Output with --check"}
[+] Building 1.5s (5/5) FINISHED
=> [internal] connecting to local controller
=> [internal] load build definition from Dockerfile
=> => transferring dockerfile: 253B
=> [internal] load metadata for docker.io/library/node:22
=> [auth] library/node:pull token for registry-1.docker.io
=> [internal] load .dockerignore
=> => transferring context: 50B
JSONArgsRecommended - https://docs.docker.com/go/dockerfile/rule/json-args-recommended/
JSON arguments recommended for ENTRYPOINT/CMD to prevent unintended behavior related to OS signals
Dockerfile:7
--------------------
5 |
6 |     COPY index.js .
7 | >>> CMD node index.js
8 |
--------------------
```

如上所示，`--check` 模式会显示该校验的[详细信息](#更详细的输出)。

与常规构建不同的是，使用 `--check` 时，如果存在违规，命令会以非零状态码退出。

## 违规时让构建失败

默认情况下，构建中的违规会以警告形式报告，退出码为 0。
你可以在 Dockerfile 中使用 `check=error=true` 指令，配置在报告违规时让构建失败。
这样会在执行实际构建前、构建校验结束后直接报错退出。

```dockerfile {title=Dockerfile,linenos=true,hl_lines=2}
# syntax=docker/dockerfile:1
# check=error=true

FROM alpine
CMD echo "Hello, world!"
```

如果没有 `# check=error=true` 指令，构建会以 0 退出码完成；加上该指令后，校验违规会导致非零退出码：

```console
$ docker build .
[+] Building 1.5s (5/5) FINISHED
...

 1 warning found (use --debug to expand):
 - JSONArgsRecommended: JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 5)
Dockerfile:1
--------------------
   1 | >>> # syntax=docker/dockerfile:1
   2 |     # check=error=true
   3 |
--------------------
ERROR: lint violation found for rules: JSONArgsRecommended
$ echo $?
1
```

你也可以通过构建参数 `BUILDKIT_DOCKERFILE_CHECK` 在命令行设置该错误指令：

```console
$ docker build --check --build-arg "BUILDKIT_DOCKERFILE_CHECK=error=true" .
```

## 跳过校验

默认构建时会运行所有校验。如需跳过特定校验，可在 Dockerfile 中使用 `check=skip` 指令。
`skip` 参数接收一个以逗号分隔的校验 ID 列表。例如：

```dockerfile {title=Dockerfile}
# syntax=docker/dockerfile:1
# check=skip=JSONArgsRecommended,StageNameCasing

FROM alpine AS BASE_STAGE
CMD echo "Hello, world!"
```

构建该 Dockerfile 将不会出现校验违规。

也可以通过传入构建参数 `BUILDKIT_DOCKERFILE_CHECK`，以逗号分隔的方式指定要跳过的校验 ID：

```console
$ docker build --check --build-arg "BUILDKIT_DOCKERFILE_CHECK=skip=JSONArgsRecommended,StageNameCasing" .
```

如需跳过所有校验，使用 `skip=all`：

```dockerfile {title=Dockerfile}
# syntax=docker/dockerfile:1
# check=skip=all
```

## 组合使用 error 与 skip 参数

若既要跳过部分校验、又要在违规时让构建失败，可同时设置 `skip` 与 `error` 参数（用分号 `;` 分隔），
可在 Dockerfile 的 `check` 指令或构建参数中配置。例如：

```dockerfile {title=Dockerfile}
# syntax=docker/dockerfile:1
# check=skip=JSONArgsRecommended,StageNameCasing;error=true
```

```console {title="Build argument"}
$ docker build --check --build-arg "BUILDKIT_DOCKERFILE_CHECK=skip=JSONArgsRecommended,StageNameCasing;error=true" .
```

## 实验性校验

在某些校验进入稳定阶段前，可能以实验性校验的形式提供；默认它们是禁用的。
可在[构建校验参考](/reference/build-checks/)查看实验性校验列表。

要启用所有实验性校验，将构建参数 `BUILDKIT_DOCKERFILE_CHECK` 设置为 `experimental=all`：

```console
$ docker build --check --build-arg "BUILDKIT_DOCKERFILE_CHECK=experimental=all" .
```

你也可以在 Dockerfile 中使用 `check` 指令启用实验性校验：

```dockerfile {title=Dockerfile}
# syntax=docker/dockerfile:1
# check=experimental=all
```

如需选择性启用某些实验性校验，可在 Dockerfile 的 `check` 指令或构建参数中，
以逗号分隔形式传入待启用的校验 ID。例如：

```dockerfile {title=Dockerfile}
# syntax=docker/dockerfile:1
# check=experimental=JSONArgsRecommended,StageNameCasing
```

注意：`experimental` 指令优先级高于 `skip`，即使设置了 `skip`，实验性校验仍会运行。
例如：即便设置了 `skip=all`，只要启用了实验性校验，它们仍会执行：

```dockerfile {title=Dockerfile}
# syntax=docker/dockerfile:1
# check=skip=all;experimental=all
```

## 延伸阅读

想进一步了解构建校验，参见：

- [构建校验参考](/reference/build-checks/)
- [使用 GitHub Actions 校验构建配置](/manuals/build/ci/github-actions/checks.md)
