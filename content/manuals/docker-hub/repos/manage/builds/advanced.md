---
description: 自动构建
keywords: 自动构建, 构建, 镜像
title: 自动构建与自动测试的高级选项
linkTitle: 高级选项
weight: 40
aliases:
- /docker-hub/builds/advanced/
---

> [!NOTE]
>
> 使用自动构建需要 Docker Pro、Team 或 Business 订阅。

以下选项用于自定义你的自动构建与自动测试流程。

## 构建与测试的环境变量

构建过程中会设置若干辅助环境变量，这些变量在自动构建、自动测试以及执行 hooks 时均可用。

> [!NOTE]
>
> 这些环境变量仅对构建与测试流程可见，不会影响服务的运行时环境。

* `SOURCE_BRANCH`：当前正在测试的分支名或标签名。
* `SOURCE_COMMIT`：当前测试提交的 SHA1 哈希。
* `COMMIT_MSG`：当前测试与构建的提交信息。
* `DOCKER_REPO`：正在构建的 Docker 存储库名称。
* `DOCKERFILE_PATH`：当前用于构建的 Dockerfile 路径。
* `DOCKER_TAG`：正在构建的 Docker 存储库标签。
* `IMAGE_NAME`：正在构建的镜像的名称与标签（由 `DOCKER_REPO`:`DOCKER_TAG` 组合而成）。

如果你在自动测试的 `docker-compose.test.yml` 文件中使用上述构建环境变量，请按如下方式在 `sut` 服务的 `environment` 中声明。

```yaml
services:
  sut:
    build: .
    command: run_tests.sh
    environment:
      - SOURCE_BRANCH
```


## 覆盖 build、test 或 push 命令

Docker Hub 允许你在自动构建与测试流程中使用 hooks 覆盖并自定义 `build`、`test` 与 `push` 命令。例如，你可以通过 build hook 设置仅在构建阶段使用的构建参数。也可以配置[自定义构建阶段 hooks](#custom-build-phase-hooks)，在这些命令之间执行额外操作。

> [!IMPORTANT]
>
> 使用这些 hooks 时需谨慎。hook 文件中的内容会替换基础的 `docker` 命令，因此你必须在 hook 中包含等价的 build、test 或 push 命令，否则自动化流程将无法完成。

要覆盖这些阶段，请在源代码仓库中、与 Dockerfile 同级目录创建 `hooks` 文件夹。然后创建 `hooks/build`、`hooks/test` 或 `hooks/push` 文件，并在其中编写构建器可执行的命令（例如 `docker` 与 `bash` 命令，并在文件首行加入合适的 `#!/bin/bash`）。

这些 hooks 运行在一台 [Ubuntu](https://releases.ubuntu.com/) 实例上，其中包含 Perl、Python 等解释器，以及 `git`、`curl` 等常用工具。可参考[Ubuntu 文档](https://ubuntu.com/)了解完整的可用解释器与工具列表。

## 自定义构建阶段 hooks

通过创建 hooks，你可以在构建流程的各个阶段之间运行自定义命令。hooks 让你能够为自动构建与自动测试提供额外指令。

在源代码仓库中（与 Dockerfile 同级目录）创建 `hooks` 文件夹，并将定义 hooks 的文件放入其中。只要在文件开头正确添加 `#!/bin/bash`，hook 文件既可以包含 `docker` 命令，也可以包含 `bash` 命令。构建器会在各步骤的前后执行这些文件中的命令。

可用的 hooks 包括：

* `hooks/pre_push`（仅在执行构建规则或[自动构建](index.md) 时使用）
* `hooks/post_push`（仅在执行构建规则或[自动构建](index.md) 时使用）

### 构建 hook 示例

#### 覆盖“build”阶段以设置变量

你可以在 hook 文件中或通过自动构建界面定义构建环境变量，随后在 hooks 中引用这些变量。

下面的示例展示了一个 build hook，使用 `docker build` 的参数根据在 Docker Hub 构建设置中定义的变量值设置 `CUSTOM`。`$DOCKERFILE_PATH` 是你提供的变量，指向要构建的 Dockerfile；`$IMAGE_NAME` 是正在构建的镜像名称。

```console
$ docker build --build-arg CUSTOM=$VAR -f $DOCKERFILE_PATH -t $IMAGE_NAME .
```

> [!IMPORTANT]
>
> `hooks/build` 文件会覆盖构建器使用的基本 `docker build` 命令，因此你必须在钩子中包含类似的构建命令，否则自动化构建失败。

参阅 [docker build 文档](/reference/cli/docker/buildx/build.md#build-arg) 了解更多构建时变量信息。

#### 推送到多个存储库

默认情况下，构建过程只会将镜像推送到配置了构建设置的那个存储库。如果你需要将同一镜像推送到多个存储库，可以通过 `post_push` hook 增加额外的标签并执行多次推送。

```console
$ docker tag $IMAGE_NAME $DOCKER_REPO:$SOURCE_COMMIT
$ docker push $DOCKER_REPO:$SOURCE_COMMIT
```

## 源仓库或分支的克隆方式

当 Docker Hub 从源代码仓库拉取某个分支时，会执行浅克隆（仅克隆该分支的最新提交）。这样可以最小化传输的数据量，并通过只拉取必要的最小代码来加快构建速度。

因此，如果你需要执行依赖其他分支的自定义操作（例如在 `post_push` hook 中），默认无法直接检出该分支，除非你采取以下方法之一：

* 获取目标分支的浅检出：

    ```console
    $ git fetch origin branch:mytargetbranch --depth 1
    ```

* 也可以对当前仓库“去浅化”，通过在 `fetch` 中使用 `--unshallow` 取回完整的 Git 历史（这可能耗时且数据量较大）：

    ```console
    $ git fetch --unshallow origin
    ```
