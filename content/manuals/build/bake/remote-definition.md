---
title: 远程 Bake 文件定义
description: 使用 Git 或 HTTP 的远程定义，通过 Bake 执行构建
keywords: build, buildx, bake, file, remote, git, http
---

你可以直接从远程 Git 仓库或 HTTPS URL 加载并构建 Bake 文件：

```console
$ docker buildx bake "https://github.com/docker/cli.git#v20.10.11" --print
#1 [internal] load git source https://github.com/docker/cli.git#v20.10.11
#1 0.745 e8f1871b077b64bcb4a13334b7146492773769f7       refs/tags/v20.10.11
#1 2.022 From https://github.com/docker/cli
#1 2.022  * [new tag]         v20.10.11  -> v20.10.11
#1 DONE 2.9s
```

该命令会从指定的远程位置获取 Bake 定义，并执行文件中定义的分组或目标。
如果远程 Bake 定义未显式指定构建上下文，Bake 会将上下文自动设置为对应的 Git 远程。
例如，[此示例](https://github.com/docker/cli/blob/2776a6d694f988c0c1df61cad4bfac0f54e481c8/docker-bake.hcl#L17-L26)
使用了 `https://github.com/docker/cli.git`：

```json
{
  "group": {
    "default": {
      "targets": ["binary"]
    }
  },
  "target": {
    "binary": {
      "context": "https://github.com/docker/cli.git#v20.10.11",
      "dockerfile": "Dockerfile",
      "args": {
        "BASE_VARIANT": "alpine",
        "GO_STRIP": "",
        "VERSION": ""
      },
      "target": "binary",
      "platforms": ["local"],
      "output": ["build"]
    }
  }
}
```

## 在远程定义中使用本地上下文

当使用远程 Bake 定义进行构建时，可能希望使用相对于命令执行目录的本地文件。
你可以通过 `cwd://` 前缀，将上下文定义为相对于命令上下文的路径。

```hcl {title="https://github.com/dvdksn/buildx/blob/bake-remote-example/docker-bake.hcl"}
target "default" {
  context = "cwd://"
  dockerfile-inline = <<EOT
FROM alpine
WORKDIR /src
COPY . .
RUN ls -l && stop
EOT
}
```

```console
$ touch foo bar
$ docker buildx bake "https://github.com/dvdksn/buildx.git#bake-remote-example"
```

```text
...
 > [4/4] RUN ls -l && stop:
#8 0.101 total 0
#8 0.102 -rw-r--r--    1 root     root             0 Jul 27 18:47 bar
#8 0.102 -rw-r--r--    1 root     root             0 Jul 27 18:47 foo
#8 0.102 /bin/sh: stop: not found
```

如果需要将特定本地目录作为上下文，可以在 `cwd://` 前缀后追加路径。
注意：追加的路径必须位于命令执行时的工作目录之内；若使用绝对路径，或相对路径指向工作目录之外，Bake 将报错。

### 本地命名上下文

你也可以使用 `cwd://` 前缀，将 Bake 执行上下文中的本地目录定义为命名上下文。

下面示例将当前工作目录下的 `./src/docs/content` 定义为名为 `docs` 的上下文：

```hcl {title=docker-bake.hcl}
target "default" {
  contexts = {
    docs = "cwd://src/docs/content"
  }
  dockerfile = "Dockerfile"
}
```

相反，如果省略 `cwd://` 前缀，路径会相对于“构建上下文”进行解析。

## 指定要使用的 Bake 定义

当从远程 Git 仓库加载 Bake 文件时，如果该仓库包含多个 Bake 文件，可以使用 `--file` 或 `-f` 指定要使用的定义：

```console
docker buildx bake -f bake.hcl "https://github.com/crazy-max/buildx.git#remote-with-local"
```

```text
...
#4 [2/2] RUN echo "hello world"
#4 0.270 hello world
#4 DONE 0.3s
```

## 组合本地与远程 Bake 定义

你也可以结合使用远程定义与本地定义，配合 `-f` 与 `cwd://` 前缀。

假设当前工作目录中有如下本地 Bake 定义：

```hcl
# local.hcl
target "default" {
  args = {
    HELLO = "foo"
  }
}
```

下面的示例使用 `-f` 指定两个 Bake 定义：

- `-f bake.hcl`：相对于 Git URL 加载该定义。
- `-f cwd://local.hcl`：相对于运行 Bake 命令的当前工作目录加载该定义。

```console
docker buildx bake -f bake.hcl -f cwd://local.hcl "https://github.com/crazy-max/buildx.git#remote-with-local" --print
```

```json
{
  "target": {
    "default": {
      "context": "https://github.com/crazy-max/buildx.git#remote-with-local",
      "dockerfile": "Dockerfile",
      "args": {
        "HELLO": "foo"
      },
      "target": "build",
      "output": [
        {
          "type": "cacheonly"
        }
      ]
    }
  }
}
```

有时需要组合本地与远程定义，例如在 GitHub Actions 中使用远程 Bake 定义构建，同时配合
[metadata-action](https://github.com/docker/metadata-action) 生成标签、注解或标签（label）。
该 Action 会在 Runner 的本地 Bake 执行上下文中生成一个 Bake 文件。若要同时使用远程定义与本地“仅包含元数据”的 Bake 文件，请同时指定两者，并为元数据 Bake 文件使用 `cwd://` 前缀：

```yml
      - name: Build
        uses: docker/bake-action@v6
        with:
          files: |
            ./docker-bake.hcl
            cwd://${{ steps.meta.outputs.bake-file }}
          targets: build
```

## 私有仓库中的远程定义

如果要使用位于私有仓库中的远程定义，拉取定义时可能需要为 Bake 指定凭据。

如果你可以通过默认的 `SSH_AUTH_SOCK` 认证访问该私有仓库，则无需为 Bake 额外指定认证参数；
Bake 会自动使用默认代理套接字。

对于使用 HTTP Token 或自定义 SSH 代理的认证方式，可通过以下环境变量配置 Bake 的认证策略：

- [`BUILDX_BAKE_GIT_AUTH_TOKEN`](../building/variables.md#buildx_bake_git_auth_token)
- [`BUILDX_BAKE_GIT_AUTH_HEADER`](../building/variables.md#buildx_bake_git_auth_header)
- [`BUILDX_BAKE_GIT_SSH`](../building/variables.md#buildx_bake_git_ssh)
