---
description: 使用 Compose Watch 在开发过程中自动更新运行中的服务
keywords: compose, file watch, experimental
title: 使用 Compose Watch
weight: 50
aliases:
- /compose/file-watch/
---

{{< summary-bar feature_name="Compose file watch" >}}

{{% include "compose/watch.md" %}}

`watch` 遵循以下文件路径规则：
* 除忽略模式外，所有路径均相对于项目目录
* 目录会被递归监视
* 不支持 Glob 通配符
* 应用 `.dockerignore` 中的规则
  * 使用 `ignore` 选项可定义额外的忽略路径（语法相同）
  * 常见 IDE（Vim、Emacs、JetBrains 等）的临时/备份文件会被自动忽略
  * `.git` 目录会被自动忽略

你无需为 Compose 项目中的所有服务都启用 `watch`。在一些场景下，只有项目的一部分（例如 JavaScript 前端）适合自动更新。

Compose Watch 主要面向基于本地源码、通过 `build` 构建的服务。对于使用 `image` 指定预构建镜像的服务，不会跟踪其文件变更。

## Compose Watch 与绑定挂载

Compose 支持将主机目录共享到服务容器中。Watch 模式并不取代该功能，而是作为补充，更适合在容器内进行开发。

更重要的是，与绑定挂载相比，`watch` 提供了更细粒度的控制。你可以通过监视规则忽略监视树中的特定文件或整个目录。

例如，在 JavaScript 项目中忽略 `node_modules/` 目录有两点好处：
* 性能：在某些配置下，包含大量小文件的目录会带来较高的 I/O 负载
* 跨平台：当主机操作系统或架构与容器不同，编译产物无法共享

例如，在 Node.js 项目中不建议同步 `node_modules/` 目录。尽管 JavaScript 是解释型语言，`npm` 包可能包含无法跨平台移植的原生代码。

## 配置

`watch` 属性用于定义一组规则，使服务能够基于本地文件变更自动更新。

每条规则都需要指定 `path`（路径模式）以及在检测到变更时采取的 `action`（动作）。`watch` 支持多种动作，且根据动作不同，可能需要或接受额外字段。

Watch 模式适用于多种语言与框架。具体路径与规则会因项目而异，但概念是一致的。

### 先决条件

为了正常工作，`watch` 依赖一些常见可执行程序。请确保你的服务镜像包含以下二进制：
* stat
* mkdir
* rmdir

此外，`watch` 要求容器内的 `USER` 对目标路径具有写权限，以便更新文件。常见做法是在 Dockerfile 中使用 `COPY` 指令将初始内容复制到容器内。为确保这些文件归配置的用户所有，请使用 `COPY --chown` 标志：

```dockerfile
# 以非特权用户运行
FROM node:18
RUN useradd -ms /bin/sh -u 1001 app
USER app

# 安装依赖
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install

# 将源文件复制到应用目录
COPY --chown=app:app . /app
```

### `action`

#### Sync

当 `action` 设置为 `sync` 时，Compose 会确保主机上对文件的更改自动与服务容器中的对应文件保持一致。

`sync` 非常适合支持“热重载”或具备等效能力的框架。

更广义地讲，在许多开发场景中，`sync` 规则可以替代绑定挂载。

#### Rebuild

当 `action` 设置为 `rebuild` 时，Compose 会使用 BuildKit 自动构建新镜像，并替换正在运行的服务容器。

其行为等同于运行 `docker compose up --build <svc>`。

`rebuild` 适用于编译型语言，或作为当某些关键文件（如 `package.json`）发生变更且需要完整重建镜像时的兜底方案。

#### Sync + Restart

当 `action` 设置为 `sync+restart` 时，Compose 会同步你的变更至服务容器，并重启它们。

当配置文件发生更改且无需重建镜像、只需重启服务主进程时，`sync+restart` 非常适合。例如，更新数据库配置或 `nginx.conf` 文件时效果很好。

>[!TIP]
>
> 通过 [镜像层缓存](/build/cache) 与 [多阶段构建](/build/building/multi-stage/) 优化 `Dockerfile`，以加快增量重建。

### `path` 与 `target`

`target` 字段用于控制路径在容器内的映射方式。

对于 `path: ./app/html` 且当 `./app/html/index.html` 发生更改时：

* `target: /app/html` -> `/app/html/index.html`
* `target: /app/static` -> `/app/static/index.html`
* `target: /assets` -> `/assets/index.html`

### `ignore`

`ignore` 模式相对于当前 `watch` 动作中定义的 `path`，而非项目目录。在下方“示例 1”中，忽略路径相对于 `path` 属性指定的 `./web` 目录。

### `initial_sync`

当使用 `sync+x` 类动作时，`initial_sync` 属性用于告知 Compose 在开始新的监视会话前，先确保 `path` 范围内的文件已同步到最新。

## 示例 1

这是一个最小示例，目标是具有如下结构的 Node.js 应用：
```text
myproject/
├── web/
│   ├── App.jsx
│   ├── index.js
│   └── node_modules/
├── Dockerfile
├── compose.yaml
└── package.json
```

```yaml
services:
  web:
    build: .
    command: npm start
    develop:
      watch:
        - action: sync
          path: ./web
          target: /src/web
          initial_sync: true
          ignore:
            - node_modules/
        - action: rebuild
          path: package.json
```

在该示例中，运行 `docker compose up --watch` 后，会基于项目根目录的 `Dockerfile` 构建的镜像启动 `web` 服务容器。
`web` 服务执行 `npm start`，随后由打包器（Webpack、Vite、Turbopack 等）启用热模块重载，运行应用的开发版本。

服务启动后，监视模式会开始监听目标目录与文件。
每当 `web/` 目录中的源文件发生修改，Compose 会将该文件同步到容器内 `/src/web` 下的对应位置。
例如，`./web/App.jsx` 会被复制到 `/src/web/App.jsx`。

同步完成后，打包器会在无需重启的情况下更新正在运行的应用。

在此场景下，`ignore` 规则会应用到 `myproject/web/node_modules/`，而非 `myproject/node_modules/`。

与源代码文件不同，新增依赖无法“即时生效”。因此，每当 `package.json` 发生变化，Compose 会重建镜像并重建 `web` 服务容器。

该模式适用于多种语言与框架，例如使用 Flask 的 Python：Python 源文件可以同步，而修改 `requirements.txt` 则应触发重建。

## 示例 2 

基于前述示例，演示 `sync+restart`：

```yaml
services:
  web:
    build: .
    command: npm start
    develop:
      watch:
        - action: sync
          path: ./web
          target: /app/web
          ignore:
            - node_modules/
        - action: sync+restart
          path: ./proxy/nginx.conf
          target: /etc/nginx/conf.d/default.conf

  backend:
    build:
      context: backend
      target: builder
```

该设置展示了如何在 Docker Compose 中使用 `sync+restart` 高效开发与测试包含前端 Web 服务器与后端服务的 Node.js 应用。该配置确保应用代码与配置文件的更改能被快速同步并应用，并在需要时重启 `web` 服务以反映变更。

## 使用 `watch`

{{% include "compose/configure-watch.md" %}}

> [!NOTE]
>
> 如果不希望应用日志与（重）建日志及文件系统同步事件混杂在一起，也可以使用专门的 `docker compose watch` 命令单独启用 Watch。

> [!TIP]
>
> 可参考 [`dockersamples/avatars`](https://github.com/dockersamples/avatars)，
> 或 [Docker 文档的本地搭建指南](https://github.com/docker/docs/blob/main/CONTRIBUTING.md)
> 了解 Compose `watch` 的演示。

## 参考

- [Compose 开发规范](/reference/compose-file/develop.md)
