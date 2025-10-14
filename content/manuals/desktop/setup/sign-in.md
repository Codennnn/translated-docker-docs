---
description: 了解学习中心，并理解登录 Docker Desktop 的好处
keywords: Docker 仪表板, 管理, 容器, 图形界面, 仪表板, 镜像, 用户手册,
  学习中心, 指南, 登录
title: 登录 Docker Desktop
linkTitle: 登录
weight: 40
aliases:
- /desktop/linux/
- /desktop/linux/index/
- /desktop/mac/
- /desktop/mac/index/
- /desktop/windows/
- /desktop/windows/index/
- /docker-for-mac/
- /docker-for-mac/index/
- /docker-for-mac/osx/
- /docker-for-mac/started/
- /docker-for-windows/
- /docker-for-windows/index/
- /docker-for-windows/started/
- /mac/
- /mackit/
- /mackit/getting-started/
- /win/
- /windows/
- /winkit/
- /winkit/getting-started/
- /desktop/get-started/
---

Docker 建议你通过 Docker 仪表板右上角的 **Sign in** 选项进行登录。

在限制管理员访问权限的大型企业环境中，管理员可以[强制启用登录](/manuals/enterprise/security/enforce-sign-in/_index.md)。

> [!TIP]
>
> 了解 [Docker 的核心订阅方案](https://www.docker.com/pricing/)，看看还能为你提供哪些能力。

## 登录的好处

- 可直接通过 Docker Desktop 访问你的 Docker Hub 仓库。

- 相比匿名用户，提升你的镜像拉取速率限制。参见[使用与限制](/manuals/docker-hub/usage/_index.md)。

- 使用 [Hardened Desktop](/manuals/enterprise/security/hardened-desktop/_index.md) 增强组织在容器化开发中的安全态势。

> [!NOTE]
>
> Docker Desktop 会在 90 天后或连续 30 天未使用后自动将你登出。

## 在 Linux 版 Docker Desktop 上登录

Linux 版 Docker Desktop 依赖 [`pass`](https://www.passwordstore.org/) 将凭据保存在 GPG 加密文件中。
在使用你的 [Docker ID](/accounts/create-account/) 登录 Docker Desktop 之前，你必须先初始化 `pass`。
如果未正确配置 `pass`，Docker Desktop 会显示警告。

1. 生成 GPG 密钥。你可以使用 GPG 密钥来初始化 pass。生成 GPG 密钥，运行：

   ``` console
   $ gpg --generate-key
   ``` 
2. 按提示输入你的姓名与邮箱。

   确认后，GPG 会创建一对密钥。查找包含你 GPG ID 的 `pub` 行，例如：

   ```text
   ...
   pubrsa3072 2022-03-31 [SC] [expires: 2024-03-30]
    3ABCD1234EF56G78
   uid          Molly <molly@example.com>
   ```
3. 复制该 GPG ID，并用它初始化 `pass`：

   ```console
   $ pass init <your_generated_gpg-id_public_key>
   ``` 

   你将看到类似如下的输出：

   ```text
   mkdir: created directory '/home/molly/.password-store/'
   Password store initialized for <generated_gpg-id_public_key>
   ```

初始化 `pass` 后，你即可登录并拉取你的私有镜像。
当 Docker CLI 或 Docker Desktop 使用凭据时，可能会弹出提示，要求输入你在生成 GPG 密钥时设置的密码。

```console
$ docker pull molly/privateimage
Using default tag: latest
latest: Pulling from molly/privateimage
3b9cc81c3203: Pull complete 
Digest: sha256:3c6b73ce467f04d4897d7a7439782721fd28ec9bf62ea2ad9e81a5fb7fb3ff96
Status: Downloaded newer image for molly/privateimage:latest
docker.io/molly/privateimage:latest
```

## 进一步阅读

- [探索 Docker Desktop](/manuals/desktop/use-desktop/_index.md) 及其功能。
- 更改你的 [Docker Desktop 设置](/manuals/desktop/settings-and-maintenance/settings.md)。
- [浏览常见问题](/manuals/desktop/troubleshoot-and-support/faqs/general.md)。
