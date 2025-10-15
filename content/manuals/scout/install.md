---
title: 安装 Docker Scout
linkTitle: 安装
weight: 10
description: Docker Scout CLI 插件的安装指南
keywords: scout, cli, install, download
---

Docker Scout CLI 插件在 Docker Desktop 中已预装。

如果你在没有 Docker Desktop 的情况下使用 Docker Engine，
Docker Scout 不会预装，
但你可以将其作为独立二进制安装。

## 安装脚本

要安装插件的最新版本，运行以下命令：

```console
$ curl -fsSL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh -o install-scout.sh
$ sh install-scout.sh
```

> [!NOTE]
>
> 在本地执行任何从互联网下载的脚本前，请先审阅其内容。
> 安装前请了解该便捷脚本可能带来的风险与限制。

## 手动安装

{{< tabs >}}
{{< tab name="Linux" >}}

1. 从[发布页](https://github.com/docker/scout-cli/releases)下载最新版本。
2. 在 `$HOME/.docker` 下创建名为 `scout` 的子目录。

   ```console
   $ mkdir -p $HOME/.docker/scout
   ```

3. 解压压缩包，并将 `docker-scout` 二进制移动到 `$HOME/.docker/scout` 目录。
4. 赋予可执行权限：`chmod +x $HOME/.docker/scout/docker-scout`。
5. 在 `.docker/config.json` 中将 `scout` 子目录添加为插件目录：

   ```json
   {
     "cliPluginsExtraDirs": [
       "/home/<USER>/.docker/scout"
     ]
   }
   ```

   将 `<USER>` 替换为你的系统用户名。

   > [!NOTE]
   > `cliPluginsExtraDirs` 的路径必须是绝对路径。

{{< /tab >}}
{{< tab name="macOS" >}}

1. 从[发布页](https://github.com/docker/scout-cli/releases)下载最新版本。
2. 在 `$HOME/.docker` 下创建名为 `scout` 的子目录。

   ```console
   $ mkdir -p $HOME/.docker/scout
   ```

3. 解压压缩包，并将 `docker-scout` 二进制移动到 `$HOME/.docker/scout` 目录。
4. 赋予可执行权限：

   ```console
   $ chmod +x $HOME/.docker/scout/docker-scout
   ```

5. 在 macOS 上取消隔离以允许执行：

   ```console
   xattr -d com.apple.quarantine $HOME/.docker/scout/docker-scout
   ```

6. 在 `.docker/config.json` 中将 `scout` 子目录添加为插件目录：

   ```json
   {
     "cliPluginsExtraDirs": [
       "/Users/<USER>/.docker/scout"
     ]
   }
   ```

   将 `<USER>` 替换为你的系统用户名。

   > [!NOTE]
   > `cliPluginsExtraDirs` 的路径必须是绝对路径。

{{< /tab >}}
{{< tab name="Windows" >}}

1. 从[发布页](https://github.com/docker/scout-cli/releases)下载最新版本。
2. 在 `%USERPROFILE%/.docker` 下创建名为 `scout` 的子目录。

   ```console
   % mkdir %USERPROFILE%\.docker\scout
   ```

3. 解压压缩包，并将 `docker-scout.exe` 移动到 `%USERPROFILE%\.docker\scout` 目录。
4. 在 `.docker\config.json` 中将 `scout` 子目录添加为插件目录：

   ```json
   {
     "cliPluginsExtraDirs": [
       "C:\Users\<USER>\.docker\scout"
     ]
   }
   ```

   将 `<USER>` 替换为你的系统用户名。

   > [!NOTE]
   > `cliPluginsExtraDirs` 的路径必须是绝对路径。

{{< /tab >}}
{{< /tabs >}}

## 容器镜像

Docker Scout CLI 插件也提供[容器镜像](https://hub.docker.com/r/docker/scout-cli)。
你可以使用 `docker/scout-cli` 运行 `docker scout` 命令，而无需在宿主机安装 CLI 插件。

```console
$ docker run -it \
  -e DOCKER_SCOUT_HUB_USER=<your Docker Hub user name> \
  -e DOCKER_SCOUT_HUB_PASSWORD=<your Docker Hub PAT>  \
  docker/scout-cli <command>
```

## GitHub Action

Docker Scout CLI 插件也提供 [GitHub Action](https://github.com/docker/scout-action)。
你可以在 GitHub 工作流中使用它，在每次推送时自动分析镜像并评估策略合规性。

Docker Scout 还可与更多 CI/CD 工具集成，例如 Jenkins、GitLab 与 Azure DevOps。
进一步了解 Docker Scout 的[可用集成](./integrations/_index.md)。
