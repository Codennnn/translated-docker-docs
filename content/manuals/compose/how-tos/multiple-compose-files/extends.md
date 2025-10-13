---
description: 了解如何使用 Docker Compose 的 extends 属性在不同文件与项目间复用服务配置。
keywords: fig, composition, compose, docker, orchestration, documentation, docs, compose file modularization
title: 扩展你的 Compose 文件
linkTitle: 扩展
weight: 20
aliases:
- /compose/extends/
- /compose/multiple-compose-files/extends/
---

Docker Compose 的[`extends` 属性](/reference/compose-file/services.md#extends)
可让你在不同的文件，甚至完全不同的项目之间共享通用配置。

当你有多个服务需要复用一组公共配置时，扩展服务会非常有用。借助 `extends`，你可以在一个地方定义通用的服务选项，并在任意位置引用它。你可以引用另一份 Compose 文件，并选择其中的某个服务在自己的应用中复用，同时按需覆盖部分属性。

> [!IMPORTANT]
>
> 当你使用多个 Compose 文件时，必须确保这些文件中的所有路径都相对于“基础 Compose 文件”（即主项目目录下的那份 Compose 文件）。原因在于，用于扩展的文件不必是完整合法的 Compose 文件，它们可以只包含少量配置片段。跟踪每个服务片段到底应相对哪个路径既困难又易混淆。为便于理解与维护，所有路径都必须以基础文件为基准来定义。

## `extends` 属性如何工作

### 从其他文件扩展服务

看这个示例：

```yaml
services:
  web:
    extends:
      file: common-services.yml
      service: webapp
```

这会指示 Compose 仅复用 `common-services.yml` 中 `webapp` 服务的属性；`webapp` 服务本身不会出现在最终项目中。

如果 `common-services.yml` 长这样：

```yaml
services:
  webapp:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - "/data"
```
效果与直接在 `compose.yaml` 的 `web` 下写入相同的 `build`、`ports` 和 `volumes` 配置完全一致。

当从另一个文件扩展服务且希望在最终项目中包含 `webapp` 时，你需要在当前 Compose 文件中显式包含这两个服务。例如（仅作示例）：

```yaml
services:
  web:
    build: alpine
    command: echo
    extends:
      file: common-services.yml
      service: webapp
  webapp:
    extends:
      file: common-services.yml
      service: webapp
```

或者，你也可以使用[包含](include.md)。 

### 在同一文件中扩展服务 

如果你在同一份 Compose 文件中定义服务，并让一个服务从另一个服务扩展，那么原始服务与扩展后的服务都会出现在最终配置中。例如：

```yaml 
services:
  web:
    build: alpine
    extends: webapp
  webapp:
    environment:
      - DEBUG=1
```

### 在同一文件中扩展并同时从其他文件扩展

你还可以更进一步，在本地的 `compose.yaml` 中定义或重新定义配置：

```yaml
services:
  web:
    extends:
      file: common-services.yml
      service: webapp
    environment:
      - DEBUG=1
    cpu_shares: 5

  important_web:
    extends: web
    cpu_shares: 10
```

## 更多示例

当你有多个服务共享通用配置时，扩展单个服务非常实用。下面的示例是一个包含两个服务（一个 Web 应用和一个队列工作进程）的 Compose 应用。两个服务使用相同的代码库，并共享许多配置选项。

`common.yaml` 定义了通用配置：

```yaml
services:
  app:
    build: .
    environment:
      CONFIG_FILE_PATH: /code/config
      API_KEY: xxxyyy
    cpu_shares: 5
```

`compose.yaml` 定义了使用这些通用配置的具体服务：

```yaml
services:
  webapp:
    extends:
      file: common.yaml
      service: app
    command: /code/run_web_app
    ports:
      - 8080:8080
    depends_on:
      - queue
      - db

  queue_worker:
    extends:
      file: common.yaml
      service: app
    command: /code/run_worker
    depends_on:
      - queue
```

## 相对路径

当 `extends` 搭配 `file` 属性并指向另一个文件夹时，被扩展服务中声明的相对路径会被转换，以确保在扩展方服务中依然指向相同的文件。如下例所示：

基础 Compose 文件：
```yaml
services:
  webapp:
    image: example
    extends:
      file: ../commons/compose.yaml
      service: base
```

`commons/compose.yaml` 文件：
```yaml
services:
  base:
    env_file: ./container.env
```

最终得到的服务仍然引用 `commons` 目录下原始的 `container.env` 文件。你可以通过 `docker compose config` 来验证，它会展示实际的模型：
```yaml
services:
  webapp:
    image: example
    env_file: 
      - ../commons/container.env
```

## 参考信息

- [`extends`](/reference/compose-file/services.md#extends)
