---
title: 将 Docker Compose 应用打包为 OCI 制品并部署
linkTitle: OCI 制品应用
weight: 110
description: 了解如何从符合 OCI 的仓库中打包、发布并安全运行 Docker Compose 应用。
keywords: cli, compose, oci, docker hub, artificats, publish, package, distribute, docker compose oci support
params:
  sidebar:
    badge:
      color: green
      text: New
---

{{< summary-bar feature_name="Compose OCI artifact" >}}

Docker Compose 支持与 [OCI 制品](/manuals/docker-hub/repos/manage/hub-images/oci-artifacts.md)协同工作，你可以通过容器仓库来打包并分发 Compose 应用。这意味着你可以把 Compose 文件与容器镜像一并存储，便于为多容器应用做版本管理、分享和部署。

## 将 Compose 应用发布为 OCI 制品

要将 Compose 应用以 OCI 制品的形式分发，可以使用 `docker compose publish` 命令，将其发布到符合 OCI 的仓库。
这样一来，其他人就可以直接从仓库部署你的应用。

发布功能支持 Compose 的大多数组合能力，例如 overrides、extends 或 include，但[存在一些限制](#limitations)。

### 通用步骤

1. 进入 Compose 应用的目录。  
   确保当前目录包含 `compose.yml` 文件，或通过 `-f` 标志显式指定 Compose 文件。

2. 在终端中登录 Docker 账号，以便在 Docker Hub 完成身份验证。

   ```console
   $ docker login
   ```

3. 使用 `docker compose publish` 命令将应用以 OCI 制品形式推送：

   ```console
   $ docker compose publish username/my-compose-app:latest
   ```
   如果你有多个 Compose 文件，运行：

   ```console
   $ docker compose -f compose-base.yml -f compose-production.yml publish username/my-compose-app:latest
   ```

### 高级发布选项

发布时，你可以传递以下附加选项： 
- `--oci-version`：指定 OCI 版本（默认自动判定）。
- `--resolve-image-digests`：将镜像标签固定到摘要（digest）。
- `--with-env`：在发布的 OCI 制品中包含环境变量。

Compose 会检查你的配置中是否包含敏感数据，并展示将要发布的环境变量，要求你确认是否继续。

```text
...
you are about to publish sensitive data within your OCI artifact.
please double check that you are not leaking sensitive data
AWS Client ID
"services.serviceA.environment.AWS_ACCESS_KEY_ID": xxxxxxxxxx
AWS Secret Key
"services.serviceA.environment.AWS_SECRET_ACCESS_KEY": aws"xxxx/xxxx+xxxx+"
Github authentication
"GITHUB_TOKEN": ghp_xxxxxxxxxx
JSON Web Token
"": xxxxxxx.xxxxxxxx.xxxxxxxx
Private Key
"": -----BEGIN DSA PRIVATE KEY-----
xxxxx
-----END DSA PRIVATE KEY-----
Are you ok to publish these sensitive data? [y/N]:y

you are about to publish environment variables within your OCI artifact.
please double check that you are not leaking sensitive data
Service/Config  serviceA
FOO=bar
Service/Config  serviceB
FOO=bar
QUIX=
BAR=baz
Are you ok to publish these environment variables? [y/N]: 
```

如果你拒绝，发布流程会立即停止，不会向仓库发送任何内容。

## 限制

将 Compose 应用发布为 OCI 制品存在一些限制。你不能发布以下 Compose 配置：
- 包含绑定挂载（bind mount）的服务
- 仅包含 `build` 段的服务
- 使用 `include` 属性包含了本地文件的配置。要成功发布，必须确保这些被包含的本地文件也已经作为制品发布。之后便可通过 `include` 引用这些文件，因为远程 `include` 受支持。

## 启动基于 OCI 制品的应用

要启动一个使用 OCI 制品的 Docker Compose 应用，可以通过 `-f`（或 `--file`）标志传入 OCI 制品引用。这允许你指定存储在仓库中的 Compose 文件（作为 OCI 制品）。

`oci://` 前缀表示应当从符合 OCI 的仓库拉取 Compose 文件，而不是从本地文件系统加载。

```console
$ docker compose -f oci://docker.io/username/my-compose-app:latest up
```

随后，使用 `docker compose up` 并通过 `-f` 指向你的 OCI 制品来运行该 Compose 应用：

```console
$ docker compose -f oci://docker.io/username/my-compose-app:latest up
```

### 故障排查

当你从 OCI 制品运行应用时，Compose 可能会显示警告信息，要求你对以下内容进行确认，以降低运行恶意应用的风险：

- 所有插值变量及其值的列表
- 应用使用的全部环境变量列表
- 该 OCI 制品应用是否使用了其他远程资源，例如通过 [`include`](/reference/compose-file/include/)

```text 
$ REGISTRY=myregistry.com docker compose -f oci://docker.io/username/my-compose-app:latest up

Found the following variables in configuration:
VARIABLE     VALUE                SOURCE        REQUIRED    DEFAULT
REGISTRY     myregistry.com      command-line   yes         
TAG          v1.0                environment    no          latest
DOCKERFILE   Dockerfile          default        no          Dockerfile
API_KEY      <unset>             none           no          

Do you want to proceed with these variables? [Y/n]:y

Warning: This Compose project includes files from remote sources:
- oci://registry.example.com/stack:latest
Remote includes could potentially be malicious. Make sure you trust the source.
Do you want to continue? [y/N]: 
```

如果你同意启动该应用，Compose 会显示已从 OCI 制品下载的所有资源所在的目录：

```text
...
Do you want to continue? [y/N]: y

Your compose stack "oci://registry.example.com/stack:latest" is stored in "~/Library/Caches/docker-compose/964e715660d6f6c3b384e05e7338613795f7dcd3613890cfa57e3540353b9d6d"
```

`docker compose publish` 命令支持非交互执行，你可以通过 `-y`（或 `--yes`）跳过确认提示： 

```console
$ docker compose publish -y username/my-compose-app:latest
```

## 进一步阅读

- [在 Docker Hub 中了解 OCI 制品](/manuals/docker-hub/repos/manage/hub-images/oci-artifacts.md)
- [Compose publish 命令](/reference/cli/docker/compose/publish.md)
- [理解 `include`](/reference/compose-file/include.md)
