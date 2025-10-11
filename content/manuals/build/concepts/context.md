---
title: 构建上下文
weight: 30
description: 了解如何使用构建上下文，让 Dockerfile 访问所需文件
keywords: build, buildx, buildkit, context, git, tarball, stdin
aliases:
  - /build/building/context/
---

命令 `docker build` 与 `docker buildx build` 会基于 [Dockerfile](/reference/dockerfile.md) 与构建上下文来构建 Docker 镜像。

## 什么是构建上下文？ {#what-is-a-build-context}

构建上下文是你的构建过程可访问的文件集合。
你传递给构建命令的位置参数用于指定要使用的构建上下文：

```console
$ docker build [OPTIONS] PATH | URL | -
                         ^^^^^^^^^^^^^^
```

你可以将以下任一输入用作构建的上下文：

- 指向本地目录的相对或绝对路径
- Git 仓库、tar 包或纯文本文件的远程 URL
- 通过标准输入传入到 `docker build` 的纯文本文件或 tar 包

### 文件系统上下文 {#filesystem-contexts}

当构建上下文是本地目录、远程 Git 仓库或 tar 文件时，该上下文中的内容就构成了构建器在构建期间可访问的文件集合。诸如 `COPY` 与 `ADD` 的构建指令可以引用上下文中的任意文件与目录。

文件系统类型的构建上下文会被递归处理：

- 指定本地目录或 tar 包时，其所有子目录都会被包含
- 指定远程 Git 仓库时，该仓库及其所有子模块都会被包含

关于构建可使用的不同文件系统上下文类型，参见：

- [本地文件](#local-context)
- [Git 仓库](#git-repositories)
- [远程 tar 包](#remote-tarballs)

### 文本文件上下文 {#text-file-contexts}

当构建上下文是一个纯文本文件时，构建器会将该文件视为 Dockerfile。这种方式下，构建不会使用文件系统上下文。

详情参见 [empty build context](#empty-context)。

## 本地上下文 {#local-context}

要使用本地构建上下文，可以在 `docker build` 命令中指定相对或绝对路径。以下示例展示了将当前目录（`.`）作为构建上下文：

```console
$ docker build .
...
#16 [internal] load build context
#16 sha256:23ca2f94460dcbaf5b3c3edbaaa933281a4e0ea3d92fe295193e4df44dc68f85
#16 transferring context: 13.16MB 2.2s done
...
```

这会让当前工作目录中的文件与目录可供构建器访问。构建器会在需要时从构建上下文中加载所需文件。

你也可以将本地 tar 包作为构建上下文，通过管道把 tar 包内容传给 `docker build`。参见 [Tarballs](#local-tarballs)。

### 本地目录

考虑以下目录结构：

```text
.
├── index.ts
├── src/
├── Dockerfile
├── package.json
└── package-lock.json
```

如果你将该目录作为构建上下文，Dockerfile 指令就可以在构建中引用并包含这些文件。

```dockerfile
# syntax=docker/dockerfile:1
FROM node:latest
WORKDIR /src
COPY package.json package-lock.json .
RUN npm ci
COPY index.ts src .
```

```console
$ docker build .
```

### 从标准输入读取 Dockerfile 的本地上下文 {#local-context-with-dockerfile-from-stdin}

若使用本地文件系统中的文件作为上下文，同时又希望从标准输入提供 Dockerfile，可使用以下语法：

```console
$ docker build -f- <PATH>
```

该语法通过 -f（或 --file）选项指定要使用的 Dockerfile，并使用连字符（-）作为文件名来指示 Docker 从标准输入读取 Dockerfile。

下面的示例将当前目录（.）作为构建上下文，并通过 heredoc 通过标准输入传入 Dockerfile 来构建镜像：

```bash
# create a directory to work in
mkdir example
cd example

# create an example file
touch somefile.txt

# build an image using the current directory as context
# and a Dockerfile passed through stdin
docker build -t myimage:latest -f- . <<EOF
FROM busybox
COPY somefile.txt ./
RUN cat /somefile.txt
EOF
```

### 本地 tar 包 {#local-tarballs}

当你将 tar 包通过管道传给构建命令时，构建会使用该 tar 包的内容作为文件系统上下文。

例如，给定以下项目目录：

```text
.
├── Dockerfile
├── Makefile
├── README.md
├── main.c
├── scripts
├── src
└── test.Dockerfile
```

你可以先将该目录打包为 tar 包，再通过管道传给构建命令作为上下文：

```console
$ tar czf foo.tar.gz *
$ docker build - < foo.tar.gz
```

构建会从 tar 包上下文中解析 Dockerfile。你可以通过 `--file` 指定 Dockerfile 在 tar 包根目录下的相对路径与名称。如下命令使用 tar 包中的 `test.Dockerfile` 进行构建：

```console
$ docker build --file test.Dockerfile - < foo.tar.gz
```

## 远程上下文 {#remote-context}

你可以将远程 Git 仓库、tar 包或纯文本文件的地址指定为构建上下文。

- 对于 Git 仓库，构建器会自动克隆该仓库。参见 [Git repositories](#git-repositories)。
- 对于 tar 包，构建器会下载并解压其内容。参见 [Tarballs](#remote-tarballs)。

如果远程 tar 包是一个文本文件，构建器将不会接收到[文件系统上下文](#filesystem-contexts)，而是将远程文件视为 Dockerfile。参见 [Empty build context](#empty-context)。

### Git 仓库 {#git-repositories}

当你将指向某个 Git 仓库位置的 URL 作为参数传给 `docker build` 时，构建器会使用该仓库作为构建上下文。

构建器会对仓库进行浅克隆，只下载 HEAD 提交，而非完整历史。

构建器会递归克隆该仓库以及其中包含的任何子模块。

```console
$ docker build https://github.com/user/myrepo.git
```

默认情况下，构建器会克隆你指定仓库的默认分支上的最新提交。

#### URL 片段 {#url-fragments}

你可以在 Git 仓库地址后附加 URL 片段，使构建器克隆仓库中的特定分支、标签以及子目录。

URL 片段的格式为 `#ref:dir`，其中：

- `ref` 表示分支名、标签名或提交哈希
- `dir` 表示仓库中的一个子目录

例如，以下命令将 `container` 分支及其下的 `docker` 子目录作为构建上下文：

```console
$ docker build https://github.com/user/myrepo.git#container:docker
```

下表展示了所有有效后缀及其对应的构建上下文：

| 构建语法后缀                   | 所用提交                      | 采用的构建上下文   |
| ------------------------------ | ----------------------------- | ------------------ |
| `myrepo.git`                   | `refs/heads/<default branch>` | `/`                |
| `myrepo.git#mytag`             | `refs/tags/mytag`             | `/`                |
| `myrepo.git#mybranch`          | `refs/heads/mybranch`         | `/`                |
| `myrepo.git#pull/42/head`      | `refs/pull/42/head`           | `/`                |
| `myrepo.git#:myfolder`         | `refs/heads/<default branch>` | `/myfolder`        |
| `myrepo.git#master:myfolder`   | `refs/heads/master`           | `/myfolder`        |
| `myrepo.git#mytag:myfolder`    | `refs/tags/mytag`             | `/myfolder`        |
| `myrepo.git#mybranch:myfolder` | `refs/heads/mybranch`         | `/myfolder`        |

当你在 URL 片段中使用提交哈希作为 `ref` 时，应使用完整的 40 字符 SHA-1 哈希。短哈希（例如截断为 7 个字符）不被支持。

```bash
# ✅ The following works:
docker build github.com/docker/buildx#d4f088e689b41353d74f1a0bfcd6d7c0b213aed2
# ❌ The following doesn't work because the commit hash is truncated:
docker build github.com/docker/buildx#d4f088e
```

#### URL 查询 {#url-queries}

{{< summary-bar feature_name="Build URL Queries" >}}

与 [URL fragments](#url-fragments) 相比，URL 查询参数结构更清晰，且更为推荐：

```console
$ docker buildx build 'https://github.com/user/myrepo.git?branch=container&subdir=docker'
```

| 构建语法后缀                                  | 所用提交                      | 采用的构建上下文   |
| -------------------------------------------- | ----------------------------- | ------------------ |
| `myrepo.git`                                 | `refs/heads/<default branch>` | `/`                |
| `myrepo.git?tag=mytag`                       | `refs/tags/mytag`             | `/`                |
| `myrepo.git?branch=mybranch`                 | `refs/heads/mybranch`         | `/`                |
| `myrepo.git?ref=pull/42/head`                | `refs/pull/42/head`           | `/`                |
| `myrepo.git?subdir=myfolder`                 | `refs/heads/<default branch>` | `/myfolder`        |
| `myrepo.git?branch=master&subdir=myfolder`   | `refs/heads/master`           | `/myfolder`        |
| `myrepo.git?tag=mytag&subdir=myfolder`       | `refs/tags/mytag`             | `/myfolder`        |
| `myrepo.git?branch=mybranch&subdir=myfolder` | `refs/heads/mybranch`         | `/myfolder`        |

你可以通过 `checksum`（或别名 `commit`）查询参数指定提交哈希，并与 `tag`、`branch` 或 `ref` 组合，用于校验引用解析到的是否为期望的提交：

```console
$ docker buildx build 'https://github.com/moby/buildkit.git?tag=v0.21.1&checksum=66735c67'
```

若校验不匹配，构建会失败：

```console
$ docker buildx build 'https://github.com/user/myrepo.git?tag=v0.1.0&commit=deadbeef'
...
#3 [internal] load git source https://github.com/user/myrepo.git?tag=v0.1.0-rc1&commit=deadbeef
#3 0.484 bb41e835b6c3523c7c45b248cf4b45e7f862bc42       refs/tags/v0.1.0
#3 ERROR: expected checksum to match deadbeef, got bb41e835b6c3523c7c45b248cf4b45e7f862bc42
```

> [!NOTE]
>
> 使用 `checksum`（或别名 `commit`）查询参数时可以使用短提交哈希；但对 `ref` 而言，仅支持完整提交哈希。

#### 保留 `.git` 目录 {#keep-git-directory}

默认情况下，使用 Git 上下文时 BuildKit 不会保留 `.git` 目录。你可以通过设置
[`BUILDKIT_CONTEXT_KEEP_GIT_DIR` 构建参数](/reference/dockerfile.md#buildkit-built-in-build-args) 让 BuildKit 保留该目录。
如果你需要在构建过程中获取 Git 信息，这会非常有用：

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
WORKDIR /src
RUN --mount=target=. \
  make REVISION=$(git rev-parse HEAD) build
```

```console
$ docker build \
  --build-arg BUILDKIT_CONTEXT_KEEP_GIT_DIR=1
  https://github.com/user/myrepo.git#main
```

#### 私有仓库 {#private-repositories}

当你指定的 Git 上下文来自私有仓库时，构建器需要你提供必要的认证凭据。你可以使用 SSH 或基于令牌的认证。

如果你提供的 Git 上下文是 SSH 或 Git 地址，Buildx 会自动检测并使用 SSH 凭据。默认使用 `$SSH_AUTH_SOCK`。
你也可以通过 [`--ssh` 选项](/reference/cli/docker/buildx/build.md#ssh) 配置要使用的 SSH 凭据。

```console
$ docker buildx build --ssh default git@github.com:user/private.git
```

如果你希望改用基于令牌的认证，可以通过
[`--secret` 选项](/reference/cli/docker/buildx/build.md#secret) 传递令牌。

```console
$ GIT_AUTH_TOKEN=<token> docker buildx build \
  --secret id=GIT_AUTH_TOKEN \
  https://github.com/user/private.git
```

> [!NOTE]
>
> 不要使用 `--build-arg` 传递机密信息。

### 从标准输入读取 Dockerfile 的远程上下文 {#remote-context-with-dockerfile-from-stdin}

若使用本地文件系统中的文件作为上下文，同时又希望从标准输入提供 Dockerfile，可使用以下语法：

```console
$ docker build -f- <URL>
```

该语法通过 -f（或 --file）选项指定要使用的 Dockerfile，并使用连字符（-）作为文件名来指示 Docker 从标准输入读取 Dockerfile。

当你想要从一个不包含 Dockerfile 的仓库构建镜像，或希望使用自定义 Dockerfile 而不维护仓库分支（fork）时，这种方式非常有用。

下面示例使用来自标准输入的 Dockerfile 构建镜像，并从 GitHub 的 [hello-world](https://github.com/docker-library/hello-world)
仓库中添加 `hello.c` 文件：

```bash
docker build -t myimage:latest -f- https://github.com/docker-library/hello-world.git <<EOF
FROM busybox
COPY hello.c ./
EOF
```

### 远程 tar 包 {#remote-tarballs}

如果你传递的是远程 tar 包的 URL，该 URL 会被直接发送给构建器。

```console
$ docker build http://server/context.tar.gz
#1 [internal] load remote build context
#1 DONE 0.2s

#2 copy /context /
#2 DONE 0.1s
...
```

下载操作会在运行 BuildKit 守护进程的主机上完成。注意：如果你使用的是远程 Docker 上下文或远程构建器，该主机不一定与发起构建命令的主机相同。BuildKit 会获取 `context.tar.gz` 并将其用作构建上下文。tar 包上下文必须是符合标准 `tar` Unix 格式的归档文件，并可使用 `xz`、`bzip2`、`gzip` 或 `identity`（不压缩）等格式进行压缩。

## 空构建上下文 {#empty-context}

当你使用文本文件作为构建上下文时，构建器会将该文件视作 Dockerfile。此时构建没有文件系统上下文。

当你的 Dockerfile 不依赖任何本地文件时，可以使用空的构建上下文进行构建。

### 如何在没有上下文的情况下构建 {#how-to-build-without-a-context}

你可以通过标准输入传递文本文件，或指定远程文本文件的 URL。

{{< tabs >}}
{{< tab name="Unix pipe" >}}

```console
$ docker build - < Dockerfile
```

{{< /tab >}}
{{< tab name="PowerShell" >}}

```powershell
Get-Content Dockerfile | docker build -
```

{{< /tab >}}
{{< tab name="Heredocs" >}}

```bash
docker build -t myimage:latest - <<EOF
FROM busybox
RUN echo "hello world"
EOF
```

{{< /tab >}}
{{< tab name="Remote file" >}}

```console
$ docker build https://raw.githubusercontent.com/dvdksn/clockbox/main/Dockerfile
```

{{< /tab >}}
{{< /tabs >}}

当你在没有文件系统上下文的情况下构建时，诸如 `COPY` 的 Dockerfile 指令将无法引用本地文件：

```console
$ ls
main.c
$ docker build -<<< $'FROM scratch\nCOPY main.c .'
[+] Building 0.0s (4/4) FINISHED
 => [internal] load build definition from Dockerfile       0.0s
 => => transferring dockerfile: 64B                        0.0s
 => [internal] load .dockerignore                          0.0s
 => => transferring context: 2B                            0.0s
 => [internal] load build context                          0.0s
 => => transferring context: 2B                            0.0s
 => ERROR [1/1] COPY main.c .                              0.0s
------
 > [1/1] COPY main.c .:
------
Dockerfile:2
--------------------
   1 |     FROM scratch
   2 | >>> COPY main.c .
   3 |
--------------------
ERROR: failed to solve: failed to compute cache key: failed to calculate checksum of ref 7ab2bb61-0c28-432e-abf5-a4c3440bc6b6::4lgfpdf54n5uqxnv9v6ymg7ih: "/main.c": not found
```

## `.dockerignore` 文件 {#dockerignore-files}

你可以使用 `.dockerignore` 文件将某些文件或目录排除在构建上下文之外。

```text
# .dockerignore
node_modules
bar
```

这有助于避免将不需要的文件与目录发送给构建器，从而提升构建速度，尤其在使用远程构建器时效果显著。

### 文件名与位置

当你运行构建命令时，构建客户端会在上下文根目录查找名为 `.dockerignore` 的文件。如果存在，符合匹配模式的文件与目录会在发送给构建器之前从构建上下文中移除。

如果你使用多个 Dockerfile，可以为每个 Dockerfile 使用不同的忽略文件。忽略文件采用特殊的命名约定：将忽略文件放在该 Dockerfile 所在目录，并以该 Dockerfile 的文件名作为前缀，如下示例所示：

```text
.
├── index.ts
├── src/
├── docker
│   ├── build.Dockerfile
│   ├── build.Dockerfile.dockerignore
│   ├── lint.Dockerfile
│   ├── lint.Dockerfile.dockerignore
│   ├── test.Dockerfile
│   └── test.Dockerfile.dockerignore
├── package.json
└── package-lock.json
```

当二者同时存在时，特定 Dockerfile 的忽略文件优先于构建上下文根目录下的 `.dockerignore`。

### 语法

`.dockerignore` 文件由若干以换行分隔的模式组成，类似于 Unix shell 的文件通配符。忽略模式中的首尾斜杠会被忽略。以下这些模式都会排除构建上下文根目录下 `foo` 子目录中的名为 `bar` 的文件或目录：

- `/foo/bar/`
- `/foo/bar`
- `foo/bar/`
- `foo/bar`

若 `.dockerignore` 文件的某行在首列以 `#` 开头，则该行被视为注释，在 CLI 解析前会被忽略。

```gitignore
#/this/is/a/comment
```

想了解 `.dockerignore` 模式匹配的精确细节，可以查看 GitHub 上的
[moby/patternmatcher 仓库](https://github.com/moby/patternmatcher/tree/main/ignorefile)，其中包含相关源码。

#### 匹配规则

下面的代码片段展示了一个示例 `.dockerignore` 文件。

```text
# comment
*/temp*
*/*/temp*
temp?
```
<!-- vale off -->

该文件会导致如下构建行为：

| 规则        | 行为                                                                                                                                                                                                           |
| :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `# comment` | 忽略。                                                                                                                                                                                                         |
| `*/temp*`   | 排除根目录下任意一级子目录中名称以 `temp` 开头的文件与目录。例如，`/somedir/temporary.txt` 与目录 `/somedir/temp` 都会被排除。                                                                                |
| `*/*/temp*` | 排除根目录下任意二级子目录中名称以 `temp` 开头的文件与目录。例如，`/somedir/subdir/temporary.txt` 会被排除。                                                                                                   |
| `temp?`     | 排除根目录下名称为 `temp` 的单字符扩展变体（一个字符后缀）的文件与目录。例如，`/tempa` 与 `/tempb` 会被排除。                                                                                                   |

<!-- vale on -->

匹配规则基于 Go 的
[`filepath.Match` 函数](https://golang.org/pkg/path/filepath#Match)。
预处理步骤会使用 Go 的
[`filepath.Clean` 函数](https://golang.org/pkg/path/filepath/#Clean)
来裁剪空白并移除 `.` 与 `..`。
预处理后为空的行会被忽略。

> [!NOTE]
>
> 出于历史原因，模式 `.` 会被忽略。

除 Go 的 `filepath.Match` 规则外，Docker 还支持特殊通配符 `**`，可匹配任意层级目录（包括零级）。例如，`**/*.go` 会排除构建上下文中任意位置的所有 `.go` 文件。

你可以在 `.dockerignore` 中排除 `Dockerfile` 与 `.dockerignore` 本身。这些文件仍会被发送给构建器（以便构建运行），但你不能通过 `ADD`、`COPY` 或挂载的方式将它们复制进镜像。

#### 取反匹配

你可以在模式前添加 `!`（感叹号）来指定排除规则的例外。下面是一个使用该机制的 `.dockerignore` 示例：

```text
*.md
!README.md
```

上下文根目录下除 `README.md` 外的所有 Markdown 文件都会被排除。注意，子目录下的 Markdown 文件仍会被包含。

`!` 例外规则的位置会影响行为：对于某个文件，`.dockerignore` 中最后一个匹配到它的规则将决定该文件被包含还是被排除。如下例：

```text
*.md
!README*.md
README-secret.md
```

上下文中不会包含任何 Markdown 文件，除了 README 文件，但不包括 `README-secret.md`。

再看如下示例：

```text
*.md
README-secret.md
!README*.md
```

所有 README 文件都会被包含。中间那一行不起作用，因为 `!README*.md` 匹配了 `README-secret.md`，且位于最后。

## 命名上下文 {#named-contexts}

除了默认的构建上下文（`docker build` 命令的位置参数）外，你还可以向构建传递额外的命名上下文。

命名上下文通过 `--build-context` 选项指定，后接 `名称=值` 的形式。它可以在构建期间引入来自多处的文件与目录，同时在逻辑上保持分离。

```console
$ docker build --build-context docs=./docs .
```

在该示例中：

- 名为 `docs` 的上下文指向 `./docs` 目录。
- 默认上下文（`.`）指向当前工作目录。

### 在 Dockerfile 中使用命名上下文 {#using-named-contexts-in-a-dockerfile}

Dockerfile 指令可以像引用多阶段构建中的阶段那样引用命名上下文。

例如，下面这个 Dockerfile：

1. 使用 `COPY` 指令将默认上下文中的文件复制到当前构建阶段。
2. 以绑定挂载的方式引入命名上下文中的文件，将其作为构建的一部分进行处理。

```dockerfile
# syntax=docker/dockerfile:1
FROM buildbase
WORKDIR /app

# Copy all files from the default context into /app/src in the build container
COPY . /app/src
RUN make bin

# Mount the files from the named "docs" context to build the documentation
RUN --mount=from=docs,target=/app/docs \
    make manpages
```

### 命名上下文的使用场景 {#use-cases-for-named-contexts}

使用命名上下文可以在构建镜像时获得更高的灵活性与效率。以下是一些常见的适用场景：

#### 示例：组合本地与远程来源

你可以针对不同来源定义独立的命名上下文。例如，在一个项目中，应用源码位于本地，而部署脚本存放在 Git 仓库中：

```console
$ docker build --build-context scripts=https://github.com/user/deployment-scripts.git .
```

在 Dockerfile 中，你可以分别使用这些上下文：

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine:latest

# Copy application code from the main context
COPY . /opt/app

# Run deployment scripts using the remote "scripts" context
RUN --mount=from=scripts,target=/scripts /scripts/main.sh
```

#### 示例：使用自定义依赖的动态构建

在某些场景下，你可能需要从外部来源动态注入配置文件或依赖。命名上下文允许你在不修改默认上下文的情况下挂载不同配置，从而简化这一过程。

```console
$ docker build --build-context config=./configs/prod .
```

示例 Dockerfile：

```dockerfile
# syntax=docker/dockerfile:1
FROM nginx:alpine

# Use the "config" context for environment-specific configurations
COPY --from=config nginx.conf /etc/nginx/nginx.conf
```

#### 示例：固定或覆盖镜像版本

你可以像引用镜像一样在 Dockerfile 中引用命名上下文。这意味着你可以通过命名上下文来覆盖 Dockerfile 中的镜像引用。例如，给定以下 Dockerfile：

```dockerfile
FROM alpine:{{% param example_alpine_version %}}
```

如果你希望在不修改 Dockerfile 的前提下强制将镜像引用解析为不同版本，可以在构建时传入同名上下文。例如：

```console
docker buildx build --build-context alpine:{{% param example_alpine_version %}}=docker-image://alpine:edge .
```

前缀 `docker-image://` 用于将上下文标记为镜像引用。该引用可以是本地镜像，也可以是仓库中的镜像。

### 在 Bake 中使用命名上下文 {#named-contexts-with-bake}

[Bake](/manuals/build/bake/_index.md) 是集成于 `docker build` 的工具，可通过配置文件管理构建配置。Bake 完全支持命名上下文。

在 Bake 文件中定义命名上下文：

```hcl {title=docker-bake.hcl}
target "app" {
  contexts = {
    docs = "./docs"
  }
}
```

这等价于使用如下 CLI 命令：

```console
$ docker build --build-context docs=./docs .
```

#### 使用命名上下文串联目标 {#linking-targets-with-named-contexts}

除了让复杂构建更易管理之外，Bake 还在 CLI `docker build` 的能力之上提供了额外功能。
你可以通过命名上下文创建构建流水线，使某个目标依赖并基于另一个目标构建。例如，考虑这样一个构建设置：你有两个 Dockerfile：

- `base.Dockerfile`：用于构建基础镜像
- `app.Dockerfile`：用于构建应用镜像

`app.Dockerfile` 使用 `base.Dockerfile` 产出的镜像作为其基础镜像：

```dockerfile {title=app.Dockerfile}
FROM mybaseimage
```

通常你需要先构建基础镜像，然后将其加载到 Docker Engine 本地镜像存储或推送到仓库。有了 Bake，你可以直接引用其他目标，在 `app` 目标与 `base` 目标之间创建依赖关系。

```hcl {title=docker-bake.hcl}
target "base" {
  dockerfile = "base.Dockerfile"
}

target "app" {
  dockerfile = "app.Dockerfile"
  contexts = {
    # the target: prefix indicates that 'base' is a Bake target
    mybaseimage = "target:base"
  }
}
```

通过该配置，`app.Dockerfile` 中对 `mybaseimage` 的引用会使用 `base` 目标的构建结果。在需要时，构建 `app` 目标也会触发对 `mybaseimage` 的重新构建：

```console
$ docker buildx bake app
```

### 延伸阅读 {#further-reading}

关于命名上下文的更多信息，参见：

- [`--build-context` CLI reference](/reference/cli/docker/buildx/build.md#build-context)
- [Using Bake with additional contexts](/manuals/build/bake/contexts.md)
