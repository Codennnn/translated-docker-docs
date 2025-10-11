---
title: 构建机密
linkTitle: 机密
weight: 30
description: 安全地管理凭据与其他机密信息
keywords: build, secrets, credentials, passwords, tokens, ssh, git, auth, http
tags: [Secrets]
---

构建机密（build secret）是指在应用构建过程中使用的任何敏感信息，例如密码或 API 令牌。

不建议通过构建参数或环境变量向构建传递机密信息，因为这些信息会保留在最终镜像中。应改用机密挂载（secret mounts）或 SSH 挂载，以更安全的方式向构建暴露机密。

## 构建机密的类型

- [机密挂载](#secret-mounts)：通用挂载方式，用于向构建传递机密。它从构建客户端获取机密，并在执行某条构建指令期间临时在构建容器中提供该机密。适用于需要与私有制品服务器或 API 通信的场景。
- [SSH 挂载](#ssh-mounts)：用于在构建中提供 SSH 套接字或密钥的专用挂载。通常在构建过程中拉取私有 Git 仓库时使用。
- [远程上下文的 Git 认证](#git-authentication-for-remote-contexts)：当你使用远程 Git 上下文且该仓库为私有时的一组预定义机密。这些属于“预检（pre-flight）”机密：它们不会在构建指令中被消费，而是用于为构建器提供获取上下文所需的凭据。

## 使用构建机密

对于机密挂载与 SSH 挂载，使用构建机密通常分两步：
1. 将机密传入 `docker build` 命令；
2. 在 Dockerfile 中消费该机密。

要向构建传递机密，请使用 [`docker build --secret` 选项](/reference/cli/docker/buildx/build.md#secret)，或使用 [Bake](../bake/reference.md#targetsecret) 的等效配置。

{{< tabs >}}
{{< tab name="CLI" >}}

```console
$ docker build --secret id=aws,src=$HOME/.aws/credentials .
```

{{< /tab >}}
{{< tab name="Bake" >}}

```hcl
variable "HOME" {
  default = null
}

target "default" {
  secret = [
    "id=aws,src=${HOME}/.aws/credentials"
  ]
}
```

{{< /tab >}}
{{< /tabs >}}

若要在构建中使用机密并使其对 `RUN` 指令可见，请在 Dockerfile 中使用
[`--mount=type=secret`](/reference/dockerfile.md#run---mounttypesecret) 参数。

```dockerfile
RUN --mount=type=secret,id=aws \
    AWS_SHARED_CREDENTIALS_FILE=/run/secrets/aws \
    aws s3 cp ...
```

## 机密挂载

机密挂载以文件或环境变量的形式向构建容器暴露机密。你可以使用机密挂载将 API 令牌、密码或 SSH 密钥等敏感信息安全地传给构建。

### 来源

机密的来源可以是一个[文件](/reference/cli/docker/buildx/build.md#file)，也可以是一个[环境变量](/reference/cli/docker/buildx/build.md#env)。
在使用 CLI 或 Bake 时，类型通常会被自动识别；也可以显式指定为 `type=file` 或 `type=env`。

下面的示例将环境变量 `KUBECONFIG` 挂载为机密 ID `kube`，并以文件的形式暴露在构建容器的 `/run/secrets/kube` 路径。

```console
$ docker build --secret id=kube,env=KUBECONFIG .
```

当机密来源于环境变量时，你可以省略 `env` 参数，以变量名作为文件名进行挂载。
下面的示例会将 `API_TOKEN` 变量的值挂载到构建容器内的 `/run/secrets/API_TOKEN`。

```console
$ docker build --secret id=API_TOKEN .
```

### 目标位置

在 Dockerfile 中消费机密时，默认会以文件的方式挂载，默认路径为 `/run/secrets/<id>`。
你可以通过 `RUN --mount` 的 `target` 与 `env` 选项自定义机密在构建容器中的挂载方式。

如下示例将机密 ID `aws` 挂载为构建容器中的文件 `/run/secrets/aws`：

```dockerfile
RUN --mount=type=secret,id=aws \
    AWS_SHARED_CREDENTIALS_FILE=/run/secrets/aws \
    aws s3 cp ...
```

若要以不同文件名挂载机密，请使用 `--mount` 的 `target` 选项：

```dockerfile
RUN --mount=type=secret,id=aws,target=/root/.aws/credentials \
    aws s3 cp ...
```

若要以环境变量而非文件的形式挂载机密，请使用 `--mount` 的 `env` 选项：

```dockerfile
RUN --mount=type=secret,id=aws-key-id,env=AWS_ACCESS_KEY_ID \
    --mount=type=secret,id=aws-secret-key,env=AWS_SECRET_ACCESS_KEY \
    --mount=type=secret,id=aws-session-token,env=AWS_SESSION_TOKEN \
    aws s3 cp ...
```

你也可以同时使用 `target` 与 `env` 选项，将机密同时挂载为文件与环境变量。

## SSH 挂载

如果你要在构建中使用的凭据是 SSH 代理套接字或密钥，可以使用 SSH 挂载而非机密挂载。拉取私有 Git 仓库是 SSH 挂载的常见用例。

下面的示例演示了如何使用 [Dockerfile SSH 挂载](/reference/dockerfile.md#run---mounttypessh) 克隆私有 GitHub 仓库。

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
ADD git@github.com:me/myprivaterepo.git /src/
```

要向构建传递 SSH 套接字，请使用 [`docker build --ssh` 选项](/reference/cli/docker/buildx/build.md#ssh)，或使用 [Bake](../bake/reference.md#targetssh) 的等效配置。

```console
$ docker buildx build --ssh default .
```

## 远程上下文的 Git 认证

BuildKit 支持两个预定义的构建机密：`GIT_AUTH_TOKEN` 与 `GIT_AUTH_HEADER`。
构建远程的私有 Git 仓库时，可通过它们设置 HTTP 认证参数，适用于：

- 使用私有 Git 仓库作为构建上下文
- 在构建中使用 `ADD` 拉取私有 Git 仓库

例如，若你有一个私有 GitLab 项目 `https://gitlab.com/example/todo-app.git`，并希望以该仓库作为构建上下文运行构建；如果未进行认证，`docker build` 将失败，因为构建器无权拉取该仓库：

```console
$ docker build https://gitlab.com/example/todo-app.git
[+] Building 0.4s (1/1) FINISHED
 => ERROR [internal] load git source https://gitlab.com/example/todo-app.git
------
 > [internal] load git source https://gitlab.com/example/todo-app.git:
0.313 fatal: could not read Username for 'https://gitlab.com': terminal prompts disabled
------
```

要让构建器通过 Git 服务器认证，请将有效的 GitLab 访问令牌设置到环境变量 `GIT_AUTH_TOKEN`，并作为机密传给构建：

```console
$ GIT_AUTH_TOKEN=$(cat gitlab-token.txt) docker build \
  --secret id=GIT_AUTH_TOKEN \
  https://gitlab.com/example/todo-app.git
```

`GIT_AUTH_TOKEN` 也可结合 `ADD` 使用，用于在构建中拉取私有 Git 仓库：

```dockerfile
FROM alpine
ADD https://gitlab.com/example/todo-app.git /src
```

### HTTP 认证方案

默认情况下，经由 HTTP 的 Git 认证使用 Bearer 方案：

```http
Authorization: Bearer <GIT_AUTH_TOKEN>
```

如果需要使用 Basic 方案（用户名/密码），可以设置 `GIT_AUTH_HEADER` 构建机密：

```console
$ export GIT_AUTH_TOKEN=$(cat gitlab-token.txt)
$ export GIT_AUTH_HEADER=basic
$ docker build \
  --secret id=GIT_AUTH_TOKEN \
  --secret id=GIT_AUTH_HEADER \
  https://gitlab.com/example/todo-app.git
```

目前 BuildKit 仅支持 Bearer 与 Basic 两种方案。

### 多主机

你可以按主机维度设置 `GIT_AUTH_TOKEN` 与 `GIT_AUTH_HEADER` 机密，以便针对不同主机名使用不同的认证参数。要指定主机名，可将其作为后缀追加到机密 ID：

```console
$ export GITLAB_TOKEN=$(cat gitlab-token.txt)
$ export GERRIT_TOKEN=$(cat gerrit-username-password.txt)
$ export GERRIT_SCHEME=basic
$ docker build \
  --secret id=GIT_AUTH_TOKEN.gitlab.com,env=GITLAB_TOKEN \
  --secret id=GIT_AUTH_TOKEN.gerrit.internal.example,env=GERRIT_TOKEN \
  --secret id=GIT_AUTH_HEADER.gerrit.internal.example,env=GERRIT_SCHEME \
  https://gitlab.com/example/todo-app.git
```
