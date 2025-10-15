---
description: 自动化测试
keywords: 自动化, 测试, 存储库
title: 存储库自动化测试
weight: 30
aliases:
- /docker-hub/builds/automated-testing/
---

> [!NOTE]
>
> 使用自动构建需要 Docker Pro、Team 或 Business 订阅。

Docker Hub 可以使用容器自动测试你的源代码存储库中的变更。你可以在任意 Docker Hub 存储库上启用 `Autotest`，针对源代码存储库的每个 pull request 运行测试，从而构建一条持续集成测试服务。

启用 `Autotest` 会为测试目的构建镜像，但不会自动将构建好的镜像推送到 Docker 存储库。若希望将构建产物推送到 Docker Hub 存储库，请启用[自动构建](index.md)。

## 设置自动化测试文件

要设置自动化测试，请创建 `docker-compose.test.yml` 文件，定义一个 `sut` 服务用于列出需要运行的测试。
`docker-compose.test.yml` 文件应与用于构建镜像的 Dockerfile 位于同一目录。

例如：

```yaml
services:
  sut:
    build: .
    command: run_tests.sh
```

上述示例会构建存储库，并在使用构建出的镜像启动的容器中运行 `run_tests.sh` 文件。

你可以在该文件中定义任意数量的关联服务。唯一的要求是必须定义 `sut` 服务，其返回码用于判断测试是否通过：当 `sut` 返回 `0` 时测试通过，否则视为失败。

> [!NOTE]
>
> 只有 `sut` 服务及其在 [`depends_on`](/reference/compose-file/services.md#depends_on) 中声明的所有依赖服务会被启动。如果你有服务需要轮询其他服务的变化，务必将这些轮询服务也加入 [`depends_on`](/reference/compose-file/services.md#depends_on) 列表，以确保它们一并启动。

如有需要，你可以定义多个 `docker-compose.test.yml` 文件。凡是以 `.test.yml` 结尾的文件都会用于测试，测试将按顺序依次运行。你也可以使用[自定义构建 hooks](advanced.md#override-build-test-or-push-commands) 进一步定制测试行为。

> [!NOTE]
>
> 如果启用了自动构建，构建过程也会执行 `test.yml` 文件中定义的测试。

## 在存储库上启用自动测试

若要在源代码存储库上启用测试，首先需要在 Docker Hub 中创建与之关联的构建存储库。`Autotest` 的设置与[自动构建](index.md) 位于同一页面；你可以不启用自动构建而仅使用自动测试。自动构建是按分支或标签启用的，并非必须启用。

无论 `Autotest` 如何配置，只有启用了自动构建的分支才会将镜像推送到 Docker 存储库。

1. 登录 Docker Hub，选择 **My Hub** > **Repositories**。

2. 选择你希望启用 `Autotest` 的存储库。

3. 在存储库视图中选择 **Builds** 选项卡。

4. 选择 **Configure automated builds**。

5. 按[自动构建](index.md) 的说明配置自动构建设置。

    至少需要配置：

    * 源代码存储库
    * 构建位置
    * 至少一条构建规则

6. 选择你的 **Autotest** 选项。

    可选项如下：

    * `Off`：不执行额外的测试构建。仅当其作为自动构建的一部分配置时才运行测试。

    * `Internal pull requests`：对匹配构建规则的分支的任意 pull request 触发测试构建，但仅当该 pull request 来自同一个源代码存储库时。

    * `Internal and external pull requests`：对匹配构建规则的任意 pull request 触发测试构建，包括来自外部源代码存储库的 pull request。

    > [!IMPORTANT]
    >
    > 出于安全考虑，公共存储库上的外部 pull request 的自动测试会受限：不会拉取私有镜像，且在 Docker Hub 中定义的环境变量不可用。自动构建不受影响，照常运行。

7. 选择 **Save** 保存设置，或选择 **Save and build** 保存并立即运行一次初始测试。

## 查看测试结果

在存储库详情页选择 **Timeline**。

在该选项卡中，你可以查看该存储库的构建与测试运行的待处理、进行中、成功与失败记录。

你可以选择任意条目查看每次测试运行的日志。
