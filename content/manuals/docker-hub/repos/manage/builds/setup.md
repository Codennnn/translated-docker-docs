---
description: 设置自动构建
keywords: 自动构建, 构建, 镜像, Docker Hub
title: 设置自动构建
linkTitle: 设置
weight: 10
aliases:
- /docker-hub/builds/automated-build/
- /docker-hub/builds/classic/
- /docker-hub/builds/
---

> [!NOTE]
>
> 使用自动构建需要 Docker Pro、Team 或 Business 订阅。

## 配置自动构建

你可以在 Docker Hub 中配置存储库，使其在你向源码提供方推送新代码时自动构建镜像。若已配置[自动测试](automated-testing.md)，仅当测试通过时才会推送新镜像。

1. 在 [Docker Hub](https://hub.docker.com) 中，前往 **My Hub** > **Repositories**，选择一个存储库查看详情。

2. 选择 **Builds** 选项卡。

3. 选择 GitHub 或 Bitbucket 以连接存放镜像源代码的位置。

   > [!NOTE]
   >
   > 你可能会被重定向到设置页面以[关联](link-source.md) 源码服务。若你正在编辑现有自动构建的设置，请选择 **Configure automated builds**。

4. 选择用于构建 Docker 镜像的 **source repository**。

   > [!NOTE]
   >
   > 你可能需要在源码提供方中指定组织或用户。选择用户后，其源代码存储库会显示在 **Select repository** 下拉列表中。

5. 可选。启用[自动测试](automated-testing.md#enable-automated-tests-on-a-repository)。

6. 查看默认的 **Build Rules**。

    构建规则用于控制 Docker Hub 如何从源代码存储库的内容构建镜像，以及如何在 Docker 存储库中为生成的镜像打标签。

    系统已为你创建了一条默认规则，你可以编辑或删除它。该默认规则会从源代码存储库中的 `master` 或 `main` 分支构建，并创建带有 `latest` 标签的 Docker 镜像。更多信息见[设置构建规则](#set-up-build-rules)。

7. 可选。选择 **plus** 图标以添加并[配置更多构建规则](#set-up-build-rules)。

8. 针对每个分支或标签，启用或禁用 **Autobuild** 开关。

    只有启用 Autobuild 的分支或标签才会被构建、测试，并将结果镜像推送到存储库。对于禁用 Autobuild 的分支，如果在存储库层面启用了测试，则仅用于测试目的构建，构建出的镜像不会被推送到存储库。

9. 针对每个分支或标签，启用或禁用 **Build Caching** 开关。

    [构建缓存](/manuals/build/building/best-practices.md#leverage-build-cache) 在频繁构建大型镜像或存在大量依赖时可以节省时间。若你希望确保所有依赖都在构建时解析，或存在一个本地构建更快的大层，则可关闭构建缓存。

10. 选择 **Save** 保存设置，或选择 **Save and build** 保存并运行一次初始测试。

    > [!NOTE]
    >
    > 系统会自动向你的源代码存储库添加一个 webhook，以在每次 push 时通知 Docker Hub。只有推送到被列为一个或多个标签来源的分支，才会触发构建。

### 设置构建规则

默认情况下，当你设置自动构建时，系统会为你创建一条基础构建规则。该默认规则会监听源代码存储库中 `master` 或 `main` 分支的变更，并将其构建为带有 `latest` 标签的 Docker 镜像。

在 **Build Rules** 区域，输入一个或多个待构建来源。

针对每个来源：

* 选择 **Source type**，指定按标签（tag）或分支（branch）进行构建。这会告知构建系统应在源代码存储库中关注何种对象。

* 输入要构建的 **Source** 分支或标签名称。

  首次配置自动构建时，系统会为你设定一条默认构建规则：从源代码中的 `master` 分支进行构建，并创建带有 `latest` 标签的 Docker 镜像。

  你也可以使用正则表达式选择要构建的源分支或标签。详见[正则表达式](#regexes-and-automated-builds)。

* 输入将应用于从该来源构建出的 Docker 镜像的标签（tag）。

  若使用了正则表达式选择来源，你可以引用其捕获组，并将结果用作标签的一部分。详见[正则表达式](#regexes-and-automated-builds)。

* 将 **Dockerfile location** 指定为相对于源代码存储库根目录的路径。若 Dockerfile 位于存储库根目录，请保持为 `/`。

> [!NOTE]
>
> 当 Docker Hub 从源代码存储库拉取某个分支时，会执行浅克隆（仅克隆指定分支的最新提交）。参见[自动构建与自动测试的高级选项](advanced.md#source-repository-or-branch-clones)了解更多。

### 构建的环境变量

你可以在配置自动构建时为构建过程设置环境变量。在 **Build environment variables** 区域选择 **plus** 图标添加变量，然后输入变量名与值。

当你在 Docker Hub 界面中设置变量值后，就可以在 `hooks` 文件中定义的命令里使用这些变量。变量值仅对具有该 Docker Hub 存储库 `admin` 访问权限的用户可见，因此可用于存放访问令牌等需要保密的信息。

> [!NOTE]
>
> 在构建配置界面设置的变量仅用于构建过程，不应与服务运行时使用的环境变量混淆（例如用于创建服务链接的变量）。

## 自动构建的高级选项

配置自动构建的最小要求是：由源分支或标签与目标 Docker 标签组成的一条构建规则。除此之外，你还可以：

- 更改构建查找 Dockerfile 的位置
- 设置构建应使用的文件路径（构建上下文）
- 配置多个静态标签或分支进行构建
- 使用正则表达式（regex）动态选择要构建的源并创建动态标签

以上所有选项都可在每个存储库的 **Build configuration** 界面中配置。在 [Docker Hub](https://hub.docker.com) 中，选择 **My Hub** > **Repositories**，进入目标存储库，选择 **Builds** 选项卡，然后点击 **Configure Automated builds**。

### 标签与分支构建

你可以配置自动构建，使得对特定分支或标签的 push 会触发构建。

1. 在 **Build Rules** 区域，选择 **plus** 图标以添加更多来源。

2. 选择 **Source type**，指定按标签（tag）或分支（branch）构建。

    > [!NOTE]
    >
    > 该选项用于告知构建系统在代码存储库中查找哪种类型的来源。

3. 输入要构建的 **Source** 分支或标签名称。

    > [!NOTE]
    >
    > 你可以直接输入名称，或使用正则表达式匹配要构建的源分支或标签名称。详见[正则表达式](index.md#regexes-and-automated-builds)。

4. 输入将应用于从该来源构建出的 Docker 镜像的标签。

   > [!NOTE]
   >
   > 若使用了正则表达式选择来源，你可以引用其捕获组，并将结果用作标签的一部分。详见[正则表达式](index.md#regexes-and-automated-builds)。

5. 为每条新建的构建规则重复步骤 2 至 4。

### 设置构建上下文与 Dockerfile 路径

根据你在源代码存储库中的文件组织方式，用于构建镜像的文件可能并不位于存储库根目录。在这种情况下，你可以指定构建搜索文件的路径。

构建上下文（build context）是相对于存储库根目录的、构建所需文件所在路径。请在 **Build context** 字段中输入该路径。若需将构建上下文设置为源代码存储库根目录，请输入 `/`。

> [!NOTE]
>
> 如果你从 **Build context** 字段中删除默认路径 `/` 并留空，构建系统会使用 Dockerfile 的路径作为构建上下文。但为避免混淆，建议明确填写完整路径。

你可以将 **Dockerfile location** 指定为相对于构建上下文的路径。若 Dockerfile 位于构建上下文的根目录，请将其路径保持为 `/`。如果构建上下文字段留空，请从源代码存储库根目录设置 Dockerfile 的路径。

### 正则表达式与自动构建

你可以指定正则表达式（regex），使只有匹配的分支或标签才会被构建。你也可以使用正则匹配的结果来创建应用于构建镜像的 Docker 标签。

你可以使用最多九个正则表达式捕获组（即用括号括起的表达式）选择要构建的来源，并在 **Docker Tag** 字段中通过 `{"\1"}` 到 `{"\9"}` 进行引用。

<!-- Capture groups Not a priority
#### Regex example: build from version number branch and tag with version number

You could also use capture groups to build and label images that come from various
sources. For example, you might have

`/(alice|bob)-v([0-9.]+)/` -->

### 使用 BuildKit 构建镜像

Autobuild 默认使用 BuildKit 构建系统。若希望使用传统的 Docker 构建系统，可添加[环境变量](index.md#environment-variables-for-builds) `DOCKER_BUILDKIT=0`。更多信息参见 [BuildKit](/manuals/build/buildkit/_index.md)。

## 团队的自动构建

当你在自己的用户账户下创建自动构建存储库时，你可以启动、取消、重试构建，并编辑与删除自己的存储库。

若你是组织的所有者，在 Docker Hub 上对团队存储库也具备相同操作。如果你是具有 `write` 权限的团队成员，你可以在团队存储库中启动、取消、重试构建，但无法编辑团队存储库设置或删除团队存储库。若你的用户账户为 `read` 权限，或你所在团队仅有 `read` 权限，你可以查看构建配置（包括测试设置）。

| 操作/权限              | 读取 | 写入 | 管理 | 所有者 |
| --------------------- | ---- | ---- | ---- | ------ |
| 查看构建详情          |  x   |  x   |  x   |   x    |
| 启动、取消、重试      |      |  x   |  x   |   x    |
| 编辑构建设置          |      |      |  x   |   x    |
| 删除构建              |      |      |      |   x    |

### 团队自动构建的服务账户

> [!NOTE]
>
> 只有所有者可以为团队设置自动构建。

当你为团队设置自动构建时，你会通过绑定到特定用户账户的 OAuth 向 Docker Hub 授予访问你的源代码存储库的权限。这意味着 Docker Hub 将拥有与所关联源码提供方账户相同的访问范围。

对于组织与团队，建议创建一个专用的服务账户来授权访问源码提供方。这样可以确保当个体用户的访问权限变更时不会导致构建失败，也能避免个体用户的私人项目暴露给整个组织。

该服务账户应当能够访问所有需要构建的存储库，并且必须对源代码存储库拥有管理员访问权限，以便管理部署密钥（deploy keys）。如有需要，你也可以将其访问范围限制到某个构建所需的特定存储库集合。

如果你要构建的存储库使用了私有子模块（私有依赖），还需要为与该账户关联的自动构建添加一个覆盖用的 `SSH_PRIVATE` 环境变量。更多信息见[故障排查](troubleshoot.md#build-repositories-with-linked-private-submodules)。

1. 在源码提供方创建一个服务账户，并为其生成 SSH 密钥。
2. 在你的组织中创建一个 “build” 团队。
3. 确保新的 “build” 团队能够访问你需要构建的每个存储库与子模块。

    1. 在 GitHub 或 Bitbucket 中，打开存储库的 **Settings** 页面。
    2. 将新的 “build” 团队添加到已授权用户列表。

        - GitHub：在 **Collaborators and Teams** 中添加团队。
        - Bitbucket：在 **Access management** 中添加团队。

4. 将服务账户加入源码提供方的 “build” 团队。

5. 以组织所有者身份登录 Docker Hub，切换到该组织，并按照说明使用该服务账户[关联源代码存储库](link-source.md)。

    > [!NOTE]
    >
    > 你可能需要先退出源码提供方的个人账户，再使用服务账户完成关联。

6. 可选。使用你生成的 SSH 密钥，为包含私有子模块的构建进行设置，方式为使用服务账户并参考[前述说明](troubleshoot.md#build-repositories-with-linked-private-submodules)。

## 下一步

- 使用环境变量、hooks 等[自定义你的构建流程](advanced.md)
- [添加自动测试](automated-testing.md)
- [管理你的构建](manage-builds.md)
- [故障排查](troubleshoot.md)
