---
title: 从 Compose 文件使用 Bake 构建
description: 使用 Bake 构建你的 Compose 服务
keywords: build, buildx, bake, buildkit, compose, yaml
aliases:
  - /build/customize/bake/compose-file/
---

Bake 支持 [Compose 文件格式](/reference/compose-file/_index.md)，可以解析 Compose 文件，并将每个服务转换为一个[目标](reference.md#target)。

```yaml
# compose.yaml
services:
  webapp-dev:
    build: &build-dev
      dockerfile: Dockerfile.webapp
      tags:
        - docker.io/username/webapp:latest
      cache_from:
        - docker.io/username/webapp:cache
      cache_to:
        - docker.io/username/webapp:cache

  webapp-release:
    build:
      <<: *build-dev
      x-bake:
        platforms:
          - linux/amd64
          - linux/arm64

  db:
    image: docker.io/username/db
    build:
      dockerfile: Dockerfile.db
```

```console
$ docker buildx bake --print
```

```json
{
  "group": {
    "default": {
      "targets": ["db", "webapp-dev", "webapp-release"]
    }
  },
  "target": {
    "db": {
      "context": ".",
      "dockerfile": "Dockerfile.db",
      "tags": ["docker.io/username/db"]
    },
    "webapp-dev": {
      "context": ".",
      "dockerfile": "Dockerfile.webapp",
      "tags": ["docker.io/username/webapp:latest"],
      "cache-from": [
        {
          "ref": "docker.io/username/webapp:cache",
          "type": "registry"
        }
      ],
      "cache-to": [
        {
          "ref": "docker.io/username/webapp:cache",
          "type": "registry"
        }
      ]
    },
    "webapp-release": {
      "context": ".",
      "dockerfile": "Dockerfile.webapp",
      "tags": ["docker.io/username/webapp:latest"],
      "cache-from": [
        {
          "ref": "docker.io/username/webapp:cache",
          "type": "registry"
        }
      ],
      "cache-to": [
        {
          "ref": "docker.io/username/webapp:cache",
          "type": "registry"
        }
      ],
      "platforms": ["linux/amd64", "linux/arm64"]
    }
  }
}
```

与 HCL 格式相比，Compose 格式存在一些限制：

- 暂不支持定义变量或全局作用域属性
- 不支持 `inherits` 服务字段，但你可以使用 [YAML 锚点](/reference/compose-file/fragments.md)
  引用其他服务，如上例中的 `&build-dev` 所示。

## `.env` 文件

你可以在名为 `.env` 的环境文件中声明默认环境变量。该文件会从执行命令的当前工作目录中加载，
并应用到通过 `-f` 传入的 Compose 定义。

```yaml
# compose.yaml
services:
  webapp:
    image: docker.io/username/webapp:${TAG:-v1.0.0}
    build:
      dockerfile: Dockerfile
```

```sh
# .env
TAG=v1.1.0
```

```console
$ docker buildx bake --print
```

```json
{
  "group": {
    "default": {
      "targets": ["webapp"]
    }
  },
  "target": {
    "webapp": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "tags": ["docker.io/username/webapp:v1.1.0"]
    }
  }
}
```

> [!NOTE]
>
> 系统环境变量的优先级高于 `.env` 文件中的同名变量。

## 使用 `x-bake` 的扩展字段

当 Compose 规范未提供某些字段时，你可以在 Compose 文件中使用
[`x-bake` 扩展字段](/reference/compose-file/extension.md) 来补充额外的构建选项：

```yaml
# compose.yaml
services:
  addon:
    image: ct-addon:bar
    build:
      context: .
      dockerfile: ./Dockerfile
      args:
        CT_ECR: foo
        CT_TAG: bar
      x-bake:
        tags:
          - ct-addon:foo
          - ct-addon:alp
        platforms:
          - linux/amd64
          - linux/arm64
        cache-from:
          - user/app:cache
          - type=local,src=path/to/cache
        cache-to:
          - type=local,dest=path/to/cache
        pull: true

  aws:
    image: ct-fake-aws:bar
    build:
      dockerfile: ./aws.Dockerfile
      args:
        CT_ECR: foo
        CT_TAG: bar
      x-bake:
        secret:
          - id=mysecret,src=./secret
          - id=mysecret2,src=./secret2
        platforms: linux/arm64
        output: type=docker
        no-cache: true
```

```console
$ docker buildx bake --print
```

```json
{
  "group": {
    "default": {
      "targets": ["addon", "aws"]
    }
  },
  "target": {
    "addon": {
      "context": ".",
      "dockerfile": "./Dockerfile",
      "args": {
        "CT_ECR": "foo",
        "CT_TAG": "bar"
      },
      "tags": ["ct-addon:foo", "ct-addon:alp"],
      "cache-from": [
        {
          "ref": "user/app:cache",
          "type": "registry"
        },
        {
          "src": "path/to/cache",
          "type": "local"
        }
      ],
      "cache-to": [
        {
          "dest": "path/to/cache",
          "type": "local"
        }
      ],
      "platforms": ["linux/amd64", "linux/arm64"],
      "pull": true
    },
    "aws": {
      "context": ".",
      "dockerfile": "./aws.Dockerfile",
      "args": {
        "CT_ECR": "foo",
        "CT_TAG": "bar"
      },
      "tags": ["ct-fake-aws:bar"],
      "secret": [
        {
          "id": "mysecret",
          "src": "./secret"
        },
        {
          "id": "mysecret2",
          "src": "./secret2"
        }
      ],
      "platforms": ["linux/arm64"],
      "output": [
        {
          "type": "docker"
        }
      ],
      "no-cache": true
    }
  }
}
```

可在 `x-bake` 中使用的字段：

- `cache-from`
- `cache-to`
- `contexts`
- `no-cache`
- `no-cache-filter`
- `output`
- `platforms`
- `pull`
- `secret`
- `ssh`
- `tags`
