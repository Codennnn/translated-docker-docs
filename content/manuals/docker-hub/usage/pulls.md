---
description: 了解 Docker Hub 的镜像拉取使用与限制。
keywords: Docker Hub, 拉取, 使用, 限制
title: Docker Hub 拉取使用与限制
linkTitle: 拉取
weight: 10
aliases:
  - /docker-hub/usage/storage/
  - /docker-hub/usage/repositories/
---

未登录用户与 Docker 个人版用户在 Docker Hub 上受到每 6 小时一次的拉取速率限制。相比之下，Docker Pro、Team 与 Business 用户享受无限制的拉取速率。

在合理使用（Fair Use）前提下，以下拉取用量与限制将根据你的订阅级别生效：

| 用户类型                  | 每 6 小时拉取速率限制                     |
|--------------------------|-----------------------------------------|
| 商业版（已登录）          | 无限制                                  |
| 团队版（已登录）          | 无限制                                  |
| 专业版（已登录）          | 无限制                                  |
| 个人版（已登录）          | 200                                     |
| 未登录用户               | 每个 IPv4 地址或 IPv6 /64 子网 100 次    |

## 拉取的定义

一次“拉取”定义如下：

 - 一次 Docker 拉取同时包含版本检查，以及由该拉取引发的任何下载。根据不同的客户端，`docker pull` 可以通过版本检查验证镜像或标签是否存在，而无需下载。
 - 版本检查不计入用量计费。
 - 对普通镜像的拉取计为对[单个 manifest](https://github.com/opencontainers/image-spec/blob/main/manifest.md) 的一次拉取。
 - 对多架构镜像的拉取会按每个不同架构分别计为一次拉取。

## 拉取归属（Attribution）

已登录用户的拉取会归属到个人命名空间或[组织命名空间](/manuals/accounts/general-faqs.md#whats-an-organization-name-or-namespace)。

归属规则如下：

- 私有拉取：私有仓库的拉取归属为该仓库命名空间的所有者。
- 公共拉取：从公共仓库拉取时，依据域名归属与组织成员关系确定归属。
- 已验证域名：从与已验证域名关联的账户拉取镜像时，归属为该[域名](/manuals/security/faqs/single-sign-on/domain-faqs.md)的所有者。
- 单一组织成员：
   - 若该已验证域名的所有者为公司，且用户仅属于该[公司](../../admin/faqs/company-faqs.md#what-features-are-supported-at-the-company-level)内的一个组织，则拉取归属于该组织。
   - 若用户仅属于一个组织，则拉取归属于该组织。
- 多个组织成员：若用户在该公司下属于多个组织，则拉取归属于用户的个人命名空间。


### 认证

为确保拉取被正确归属，你必须在 Docker Hub 上完成认证。以下内容介绍如何登录 Docker Hub 以完成认证。

#### Docker Desktop

如果你使用 Docker Desktop，可在 Docker Desktop 菜单中登录 Docker Hub。

在 Docker Desktop 菜单中选择 **Sign in / Create Docker ID**，按屏幕提示完成登录。

#### Docker Engine

如果你使用独立的 Docker Engine，请在终端运行 `docker login` 命令以完成与 Docker Hub 的认证。关于命令用法，参见 [docker login](/reference/cli/docker/login.md)。

#### Docker Swarm

如果你运行的是 Docker Swarm，必须使用 `--with-registry-auth` 标志与 Docker Hub 进行认证。详见 [Create a service](/reference/cli/docker/service/create.md#with-registry-auth)。若通过 Docker Compose 文件部署应用栈，请参见 [docker stack deploy](/reference/cli/docker/stack/deploy.md)。

#### GitHub Actions

若使用 GitHub Actions 构建并推送 Docker 镜像到 Docker Hub，请参见 [login action](https://github.com/docker/login-action#dockerhub)。若使用其他 Action，你需要以类似方式添加用户名与访问令牌进行认证。

#### Kubernetes

如果你运行的是 Kubernetes，请参照 [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) 配置认证。

#### 第三方平台

如果你使用第三方平台，请遵循其提供商关于仓库认证的说明。

> [!NOTE]
>
> 当通过第三方平台拉取镜像时，平台可能会为多个用户使用相同的 IPv4 地址或 IPv6 /64 子网进行拉取。即使你已完成认证，归属于同一 IPv4 地址或 IPv6 /64 子网的拉取也可能触发[滥用速率限制](./_index.md#abuse-rate-limit)。

- [Artifactory](https://www.jfrog.com/confluence/display/JFROG/Advanced+Settings#AdvancedSettings-RemoteCredentials)
- [AWS CodeBuild](https://aws.amazon.com/blogs/devops/how-to-use-docker-images-from-a-private-registry-in-aws-codebuild-for-your-build-environment/)
- [AWS ECS/Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth.html)
- [Azure Pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#sep-docreg)
- [Chipper CI](https://docs.chipperci.com/builds/docker/#rate-limit-auth)
- [CircleCI](https://circleci.com/docs/2.0/private-images/)
- [Codefresh](https://codefresh.io/docs/docs/docker-registries/external-docker-registries/docker-hub/)
- [Drone.io](https://docs.drone.io/pipeline/docker/syntax/images/#pulling-private-images)
- [GitLab](https://docs.gitlab.com/ee/user/packages/container_registry/#authenticate-with-the-container-registry)
- [LayerCI](https://layerci.com/docs/advanced-workflows#logging-in-to-docker)
- [TeamCity](https://www.jetbrains.com/help/teamcity/integrating-teamcity-with-docker.html#Conforming+with+Docker+download+rate+limits)

## 查看每月拉取与包含用量

你可以在 Docker Hub 的[使用情况页面](https://hub.docker.com/usage/pulls)查看每月的拉取统计。

在该页面，你还可以将报告发送到邮箱，报告中包含一个逗号分隔的文件（CSV），其中包含以下详细信息：

| CSV 列               | 定义                                                                                                                                                                                                               | 使用建议                                                                                                                                                                            |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `datehour`           | 触发数据传输的拉取所对应的日期与小时（`yyyy/mm/dd/hh`）。                                                                                                                                                        | 有助于识别高峰时段与使用模式。                                                                                                                                                      |
| `user_name`          | 执行拉取的用户 Docker ID。                                                                                                                                                                                         | 便于组织所有者按用户跟踪数据消耗并有效管理资源。                                                                                                                                   |
| `repository`         | 被拉取镜像所在的仓库名称。                                                                                                                                                                                         | 帮助你识别访问最频繁、数据传输最多的仓库。                                                                                                                                         |
| `access_token_name`  | 通过 Docker CLI 进行认证所用访问令牌的名称。`generated` 令牌是在用户登录时由 Docker 客户端自动生成的。                                                                                                           | 个人访问令牌通常用于认证自动化工具（Docker Desktop、CI/CD 工具等）。有助于识别触发拉取的自动化系统。                                                                               |
| `ips`                | 用于拉取镜像的 IP 地址。该字段为聚合字段，因此可能出现多个 IP，代表同一日期与小时内用于拉取该镜像的所有 IP。                                                                                                     | 有助于了解数据传输来源，用于诊断与识别自动化或手动拉取的模式。                                                                                                                      |
| `repository_privacy` | 被拉取镜像仓库的隐私状态，可为 `public` 或 `private`。                                                                                                                                                           | 区分公共与私有仓库，有助于识别该拉取影响的用量阈值。                                                                                                                                |
| `tag`                | 镜像的标签。仅当本次拉取包含标签时才会提供该字段。                                                                                                                                                               | 有助于识别镜像。标签常用于标识镜像的特定版本或变体。                                                                                                                                |
| `digest`             | 镜像的唯一摘要（digest）。                                                                                                                                                                                         | 有助于识别镜像。                                                                                                                                                                    |
| `version_checks`     | 按镜像仓库在特定日期与小时累计的版本检查次数。依据客户端不同，拉取可通过版本检查验证镜像或标签是否存在，而无需下载。                                                                                             | 有助于识别版本检查的频率，可用于分析使用趋势和潜在的异常行为。                                                                                                                      |
| `pulls`              | 按镜像仓库在特定日期与小时累计的拉取次数。                                                                                                                                                                       | 有助于识别仓库拉取的频率，可用于分析使用趋势和潜在的异常行为。                                                                                                                      |

## 查看拉取速率与限制

拉取速率限制以 6 小时为单位计算。拥有付费订阅的用户或自动化系统不受拉取速率限制。未登录用户与使用 Docker 个人版的用户在镜像拉取时会受到速率限制。

当你发起拉取且超出限制时，请求 manifest 会返回 `429` 状态码，并包含如下响应体：

```text
You have reached your pull rate limit. You may increase the limit by authenticating and upgrading: https://www.docker.com/increase-rate-limits
```

该错误消息会显示在 Docker CLI 或 Docker Engine 日志中。

要查看你当前的拉取速率与限制：

> [!NOTE]
>
> 需要安装 `curl`、`grep` 与 `jq` 才能进行查询。

1. 获取 token。

   - 匿名获取 token（匿名拉取时）：

      ```console
      $ TOKEN=$(curl "https://auth.docker.io/token?service=registry.docker.io&scope=repository:ratelimitpreview/test:pull" | jq -r .token)
      ```

   - 使用账户获取 token（已完成认证时），在以下命令中填入你的用户名与密码：

      ```console
      $ TOKEN=$(curl --user 'username:password' "https://auth.docker.io/token?service=registry.docker.io&scope=repository:ratelimitpreview/test:pull" | jq -r .token)
      ```

2. 获取包含限制信息的响应头。GET 与 HEAD 请求都会返回这些响应头。使用 GET 会模拟真实拉取并计入限制；使用 HEAD 则不会。


   ```console
   $ curl --head -H "Authorization: Bearer $TOKEN" https://registry-1.docker.io/v2/ratelimitpreview/test/manifests/latest
   ```

3. 检查响应头。你应当能看到如下内容：

   ```text
   ratelimit-limit: 100;w=21600
   ratelimit-remaining: 20;w=21600
   docker-ratelimit-source: 192.0.2.1
   ```

   在上述示例中，拉取上限为每 21600 秒（6 小时）100 次，目前剩余 20 次。

   如果你未看到任何 `ratelimit` 响应头，可能是因为该镜像或你的 IP 与某个发布者、服务提供商或开源组织有合作而享受不限额；也可能是因为你当前的用户属于付费的 Docker 订阅。若未出现这些响应头，拉取该镜像将不会计入拉取速率限制。

