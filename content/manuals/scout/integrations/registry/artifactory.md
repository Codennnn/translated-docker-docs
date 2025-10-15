---
description: 将 Artifactory 容器仓库与 Docker Scout 集成
keywords: docker scout, artifactory, integration, image analysis, security, cves
title: 将 Docker Scout 与 Artifactory 容器仓库集成
linkTitle: Artifactory Container Registry
---

{{% experimental %}}

`docker scout watch` 命令为实验性功能。

实验性功能仅用于测试与反馈，其功能或设计可能在版本间变更且不另行通知，或在未来版本中被移除。

{{% /experimental %}}

将 Docker Scout 与 JFrog Artifactory 集成后，您可以对 Artifactory 中的镜像进行索引与分析。该集成通过一个长期运行的 `docker scout watch` 进程实现。它会从您选择的仓库（可选过滤）拉取镜像、接收来自 Artifactory 的 webhook 回调，并将镜像数据推送到 Docker Scout。您可以在 Docker Scout 控制台或通过 `docker scout` CLI 查看结果。

## 工作原理

在您控制的主机上运行 [`docker scout watch`](/reference/cli/docker/scout/watch/)，并通过 `--registry "key=value,..."` 配置与 Artifactory 相关的仓库字符串。watch 进程可以：

- 监控指定仓库或整个仓库服务
- 可选地一次性导入所有现有镜像
- 定期刷新仓库列表
- 在您指定的本地端口接收来自 Artifactory 的 webhook 回调

完成集成后，Docker Scout 会自动拉取并分析推送到 Artifactory 的镜像。镜像的元数据会存储在 Docker Scout 平台上，但不会存储镜像本体。关于镜像数据处理方式，参见：[数据处理](/manuals/scout/deep-dive/data-handling.md)。

### Artifactory 专用的 registry 字符串选项

以下 `type=artifactory` 选项会覆盖 `--registry` 的通用处理：

| Key              | Required | Description                                                                            |
|------------------|:--------:|----------------------------------------------------------------------------------------|
| `type`           |   Yes    | 必须为 `artifactory`。                                                                 |
| `registry`       |   Yes    | Docker/OCI 仓库主机名（如 `example.jfrog.io`）。                                       |
| `api`            |   Yes    | Artifactory REST API 基础 URL（如 `https://example.jfrog.io/artifactory`）。           |
| `repository`     |   Yes    | 要监控的仓库（替代 `--repository` 选项）。                                             |
| `includes`       |    No    | 需要包含的通配模式（如 `*/frontend*`）。                                               |
| `excludes`       |    No    | 需要排除的通配模式（如 `*/legacy/*`）。                                                 |
| `port`           |    No    | 用于接收 webhook 回调的本地端口。                                                       |
| `subdomain-mode` |    No    | `true` 或 `false`；匹配 Artifactory 的 Docker 布局（子域名 vs 仓库路径）。             |

## 集成 Artifactory 仓库

按照以下步骤将您的 Artifactory 仓库与 Docker Scout 集成：

1. 选择运行 `docker scout watch` 的主机。

   该主机需要具备对私有仓库的本地或网络访问能力，并可通过互联网访问 Scout API（`https://api.scout.docker.com`）。如果使用 webhook 回调，还需确保 Artifactory 能够访问该主机的已配置端口。
   根据主机规模与预计负载，适当调整 `--workers`（默认 `3`）以获得最佳性能。

2. 确保正在运行的 Scout 为最新版本。

   查看当前版本：

   ```console
   $ docker scout version
   ```

   如有需要，请[安装最新版本的 Scout](https://docs.docker.com/scout/install/)。

3. 配置 Artifactory 凭据。

   为 Scout 客户端配置用于访问 Artifactory 的凭据。以下示例使用环境变量；将 `<user>` 与 `<password-or-access-token>` 替换为实际值：

   ```console
   $ export DOCKER_SCOUT_ARTIFACTORY_API_USER=<user>
   $ export DOCKER_SCOUT_ARTIFACTORY_API_PASSWORD=<password-or-access-token>
   ```

   > [!TIP]
   >
   > 最佳实践：创建一个仅读的专用用户，并优先使用访问令牌而非密码。

   配置 Artifactory 用于验证 webhook 回调的凭据。以下示例使用环境变量；将 `<random-64-128-character-secret>` 替换为实际的密钥：

   ```console
   $ export DOCKER_SCOUT_ARTIFACTORY_WEBHOOK_SECRET=<random-64-128-character-secret>
   ````

   > [!TIP]
   >
   > 最佳实践：生成 64–128 个字符、熵值较高的随机字符串。

4. 配置 Scout 凭据。

   1. 生成用于访问 Scout 的组织访问令牌。参见：[创建组织访问令牌](/enterprise/security/access-tokens/#create-an-organization-access-token)。
   2. 使用组织访问令牌登录 Docker。

       ```console
       $ docker login --username <your_organization_name>
       ```

       当提示输入密码时，粘贴上一步生成的组织访问令牌。

   3. 将本地 Docker 环境连接到组织的 Docker Scout 服务：

       ```console
       $ docker scout enroll <your_organization_name>
       ```

5. 索引现有镜像（仅需执行一次）。

    使用 `--all-images` 选项运行 `docker scout watch`，以索引指定 Artifactory 仓库中的所有镜像。例如：

   ```console
   $ docker scout watch --registry \
   "type=artifactory,registry=example.jfrog.io,api=https://example.jfrog.io/artifactory,include=*/frontend*,exclude=*/dta/*,repository=docker-local,port=9000,subdomain-mode=true" \
   --all-images
   ```

6. 在 [Scout 控制台](https://scout.docker.com/) 中确认镜像已被索引。

7. 配置 Artifactory 回调。

   在 Artifactory UI 或通过 REST API，为镜像推送/更新事件配置 webhooks。将回调地址指向运行 `docker scout watch` 的主机与端口，并包含 `DOCKER_SCOUT_ARTIFACTORY_WEBHOOK_SECRET` 用于鉴权。

   参见 [JFrog Artifactory Webhooks 文档](https://jfrog.com/help/r/jfrog-platform-administration-documentation/webhooks)
   或 [JFrog Artifactory REST API Webhooks 文档](https://jfrog.com/help/r/jfrog-rest-apis/webhooks)。

8. 持续监控新增或更新的镜像。

   使用 `--refresh-registry` 选项运行 `docker scout watch` 以持续发现需索引的新镜像。例如：

   ```console
   $ docker scout watch --registry \
   "type=artifactory,registry=example.jfrog.io,api=https://example.jfrog.io/artifactory,include=*/frontend*,exclude=*/dta/*,repository=docker-local,port=9000,subdomain-mode=true" \
   --refresh-registry
   ```

9. 可选：为常用团队协作平台设置实时通知。详见：[将 Docker Scout 与 Slack 集成](../team-collaboration/slack.md)。