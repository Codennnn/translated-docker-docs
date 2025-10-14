---
description: 了解在 Docker 仪表板的 Containers 视图中可以执行的操作
keywords: Docker 仪表板, 管理, 容器, 图形界面, 仪表板, 镜像, 用户手册
title: 浏览 Docker Desktop 的 Containers 视图
linkTitle: 容器（Containers）
weight: 10
---

**Containers** 视图会列出所有正在运行与已停止的容器和应用。它提供简洁的界面，帮助你管理容器生命周期、与正在运行的应用交互，以及检查 Docker 对象（包括 Docker Compose 应用）。

## 容器操作

使用 **Search**（搜索）字段按名称查找特定容器。

在 **Containers** 视图中你可以：
- 启动、停止、暂停、恢复或重启容器
- 查看镜像内的软件包与漏洞（CVE）
- 删除容器
- 在 VS Code 中打开该应用
- 在浏览器中打开容器暴露的端口
- 复制 `docker run` 命令以复用或修改
- 使用 [Docker Debug](#execdebug)

## 资源使用

在 **Containers** 视图中，你可以随时间监控容器的 CPU 与内存使用。这有助于判断容器是否存在异常，或是否需要分配更多资源。

当你[检查容器](#inspect-a-container)时，**Stats**（指标）选项卡会显示更多资源利用信息。你可以查看容器随时间占用的 CPU、内存、网络与磁盘空间。

## 检查容器

选择容器后，你可以查看其详细信息。

你可以通过快捷操作按钮执行暂停、恢复、启动或停止等操作，或浏览 **Logs**（日志）、**Inspect**（检查）、**Bind mounts**（绑定挂载）、**Debug**、**Files**（文件）与 **Stats**（指标）等选项卡。

### 日志（Logs）

选择 **Logs** 可实时查看容器输出。查看日志期间，你可以：

- 使用 `Cmd + f`/`Ctrl + f` 打开搜索栏查找特定条目，匹配项会以黄色高亮。
- 按 `Enter` 或 `Shift + Enter` 分别跳转到下一个或上一个匹配项。
- 使用右上角的 **Copy** 图标复制全部日志到剪贴板。
- 显示时间戳。
- 使用右上角 **Clear terminal** 图标清空日志终端。
- 打开并查看日志中的外部链接。

你可以通过以下方式进一步筛选：

- 在运行多容器应用时，仅查看某个特定容器的日志。
- 使用正则表达式或精确匹配的搜索词。

### 检查（Inspect）

选择 **Inspect** 可查看容器的底层信息，包括本地路径、镜像版本、SHA-256、端口映射等。

### Exec/Debug

如果你尚未在设置中启用 Docker Debug，则会显示 **Exec** 选项卡。它允许你在正在运行的容器内快速执行命令。

使用 **Exec** 选项卡等同于执行以下命令之一：

- `docker exec -it <container-id> /bin/sh`
- 访问 Windows 容器时：`docker exec -it <container-id> cmd.exe`

更多详情参见 [`docker exec` CLI 参考](/reference/cli/docker/exec/)。

如果你已在设置中启用 Docker Debug，或在选项卡右侧打开 **Debug mode**，则会显示 **Debug** 选项卡。

调试模式需要 [Pro、Team 或 Business 订阅](/subscription/details/)。调试模式具有以下优势：

- 可定制的工具箱。预装了许多常用 Linux 工具，如 `vim`、`nano`、`htop`、`curl`。详见 [`docker debug` CLI 参考](/reference/cli/docker/debug/)。
- 能够访问没有 shell 的容器，例如精简或 distroless 容器。

要使用调试模式：

1. 使用拥有 Pro、Team 或 Business 订阅的账号登录 Docker Desktop。
2. 登录后，可以：

   - 将鼠标悬停在正在运行的容器上，在 **Actions**（操作）列选择 **Show container actions** 菜单，并在下拉菜单中选择 **Use Docker Debug**。
   - 或者，选择该容器后，转到 **Debug** 选项卡。

若要默认启用调试模式，请前往 **Settings** > **General**，启用 **Enable Docker Debug by default**。

### 文件（Files）

选择 **Files** 可查看正在运行或已停止容器的文件系统。你还可以：

- 查看最近添加、修改或删除的文件
- 直接在内置编辑器中编辑文件
- 在宿主机与容器之间拖放文件与文件夹
- 在文件上右键删除不需要的文件
- 将容器中的文件与文件夹直接下载到宿主机

## 其他资源

- [什么是容器](/get-started/docker-concepts/the-basics/what-is-a-container.md)
- [运行多容器应用](/get-started/docker-concepts/running-containers/multi-container-applications.md)
