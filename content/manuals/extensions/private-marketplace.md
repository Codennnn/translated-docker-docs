---
description: 如何配置与使用 Docker 扩展的私有市场
keywords: Docker Extensions, Docker Desktop, Linux, Mac, Windows, Marketplace, private, security, admin
title: 为扩展配置私有市场（Private marketplace）
tags: [admin]
linkTitle: 配置私有市场
weight: 30
aliases:
 - /desktop/extensions/private-marketplace/
---

{{< summary-bar feature_name="Private marketplace" >}}

了解如何为 Docker Desktop 用户配置并部署一个私有扩展市场，提供你审核与管理的扩展清单。

Docker 扩展的私有市场专为不向开发者授予本机 root 权限的组织而设计。它依托 [Settings Management](/manuals/enterprise/security/hardened-desktop/settings-management/_index.md)，使管理员能够对私有市场进行完全控制。

## 先决条件

- [下载并安装 Docker Desktop 4.26.0 或更高版本](https://docs.docker.com/desktop/release-notes/)。
- 你需要是组织的管理员。
- 你可以通过设备管理软件（如 [Jamf](https://www.jamf.com/)）将 `extension-marketplace` 目录与 `admin-settings.json` 文件下发到下文指定的位置。

## 步骤一：初始化私有市场

1. 在本地创建用于存放将分发到开发者机器上的内容的目录：

   ```console
   $ mkdir my-marketplace
   $ cd my-marketplace
   ```

2. 初始化私有市场所需的配置文件：

   {{< tabs group="os_version" >}}
   {{< tab name="Mac" >}}

   ```console
   $ /Applications/Docker.app/Contents/Resources/bin/extension-admin init
   ```

   {{< /tab >}}
   {{< tab name="Windows" >}}

   ```console
   $ C:\Program Files\Docker\Docker\resources\bin\extension-admin init
   ```

   {{< /tab >}}
   {{< tab name="Linux" >}}

   ```console
   $ /opt/docker-desktop/extension-admin init
   ```

   {{< /tab >}}
   {{< /tabs >}}

上述命令会生成 2 个文件：

- `admin-settings.json`：当该文件被应用到开发者的 Docker Desktop 后，将启用私有市场功能。
- `extensions.txt`：用于定义你的私有市场要展示的扩展列表。

## 步骤二：设置行为

生成的 `admin-settings.json` 文件包含多个可修改的配置项。

每个配置项都包含可设置的 `value`，并可通过 `locked` 字段将其锁定，使开发者无法修改。

- `extensionsEnabled`：启用 Docker Extensions。
- `extensionsPrivateMarketplace`：激活私有市场，确保 Docker Desktop 连接到由管理员定义和控制的内容，而非公共市场。
- `onlyMarketplaceExtensions`：允许或阻止开发者通过命令行安装其他扩展。正在开发新扩展的团队需要将此项解锁（`"locked": false`）以便安装和测试。
- `extensionsPrivateMarketplaceAdminContactURL`：定义开发者在私有市场中请求新增扩展时的联系链接。若 `value` 为空，Docker Desktop 不显示该链接；否则可以是 HTTP 链接或 “mailto:” 链接。例如：

  ```json
  "extensionsPrivateMarketplaceAdminContactURL": {
    "locked": true,
    "value": "mailto:admin@acme.com"
  }
  ```

关于 `admin-settings.json` 的更多信息，参见 [Settings Management](/manuals/enterprise/security/hardened-desktop/settings-management/_index.md)。

## 步骤三：列出允许的扩展

生成的 `extensions.txt` 文件用于定义你的私有市场中可用的扩展列表。

文件中的每一行代表一个允许的扩展，格式为 `org/repo:tag`。

例如，如果你希望允许 Disk Usage 扩展，可以在 `extensions.txt` 中添加：

```console
docker/disk-usage-extension:0.2.8
```

如果未提供 tag，将使用该镜像可用的最新 tag。你也可以使用 `#` 注释行，使该扩展被忽略。

该列表可包含不同来源的扩展镜像：

- 来自公共市场的扩展，或存储在 Docker Hub 的公共镜像。
- 存储在 Docker Hub 的私有镜像。开发者需登录并拥有拉取权限。
- 存储在私有镜像仓库的镜像。开发者需登录并拥有拉取权限。

> [!IMPORTANT]
>
> 开发者只能安装你在列表中指定的扩展版本。

## 步骤四：生成私有市场

准备好 `extensions.txt` 后，即可生成私有市场：

{{< tabs group="os_version" >}}
{{< tab name="Mac" >}}

```console
$ /Applications/Docker.app/Contents/Resources/bin/extension-admin generate
```

{{< /tab >}}
{{< tab name="Windows" >}}

```console
$ C:\Program Files\Docker\Docker\resources\bin\extension-admin generate
```

{{< /tab >}}
{{< tab name="Linux" >}}

```console
$ /opt/docker-desktop/extension-admin generate
```

{{< /tab >}}
{{< /tabs >}}

该命令会创建 `extension-marketplace` 目录，并为所有允许的扩展下载市场元数据。

市场内容基于扩展镜像的标签信息生成，其格式与[公共扩展相同](extensions-sdk/extensions/labels.md)，包含扩展标题、描述、截图、链接等。

## 步骤五：测试私有市场配置

建议你先在本机的 Docker Desktop 上验证私有市场配置。

1. 在终端中运行以下命令。该命令会自动将生成的文件复制到 Docker Desktop 读取配置的路径。不同操作系统对应的位置如下：

    - Mac: `/Library/Application\ Support/com.docker.docker`
    - Windows: `C:\ProgramData\DockerDesktop`
    - Linux: `/usr/share/docker-desktop`

   {{< tabs group="os_version" >}}
   {{< tab name="Mac" >}}

   ```console
   $ sudo /Applications/Docker.app/Contents/Resources/bin/extension-admin apply
   ```

   {{< /tab >}}
   {{< tab name="Windows (run as admin)" >}}

   ```console
   $ C:\Program Files\Docker\Docker\resources\bin\extension-admin apply
   ```

   {{< /tab >}}
   {{< tab name="Linux" >}}

   ```console
   $ sudo /opt/docker-desktop/extension-admin apply
   ```

   {{< /tab >}}
   {{< /tabs >}}

2. 退出并重新打开 Docker Desktop。
3. 使用 Docker 账户登录。

当你打开 **Extensions** 选项卡时，应能看到私有市场仅列出 `extensions.txt` 中允许的扩展。

![扩展私有市场](/assets/images/extensions-private-marketplace.webp)

## 步骤六：分发私有市场

确认私有市场配置生效后，最后一步是通过组织使用的 MDM 软件（例如 [Jamf](https://www.jamf.com/)）将相关文件分发到开发者的机器。

需要分发的文件包括：
* `admin-settings.json`
* 整个 `extension-marketplace` 目录及其子目录

这些文件需要放置在开发者机器上的以下路径（同前所列）：

- Mac: `/Library/Application\ Support/com.docker.docker`
- Windows: `C:\ProgramData\DockerDesktop`
- Linux: `/usr/share/docker-desktop`

确保开发者已登录 Docker Desktop，使私有市场配置生效。作为管理员，建议[启用强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)。

## 反馈

如需反馈或报告问题，请发送邮件至 `extensions@docker.com`。
