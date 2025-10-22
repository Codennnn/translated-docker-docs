---
title: 启用增强容器隔离
linkTitle: 启用 ECI
description: 启用增强容器隔离以保护 Docker Desktop 中的容器安全
keywords: 增强容器隔离, 启用 eci, 容器安全, docker desktop 设置
weight: 15
---

{{< summary-bar feature_name="Hardened Docker Desktop" >}}

ECI 可防止恶意容器破坏 Docker Desktop，同时保持完整的开发人员生产力。

本页面介绍如何启用增强容器隔离（ECI）并验证其是否正常工作。

## 先决条件

在开始之前，您必须具备：

- Docker Business 订阅
- Docker Desktop 4.13 或更高版本
- [强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)（仅适用于管理组织范围设置的管理员）

## 启用增强容器隔离

### 面向开发人员

在 Docker Desktop 设置中启用 ECI：

1. 在 Docker Desktop 中登录您的组织。您的组织必须具有 Docker Business 订阅。
1. 停止并删除所有现有容器：

    ```console
    $ docker stop $(docker ps -q)
    $ docker rm $(docker ps -aq)
    ```

1. 在 Docker Desktop 中，转到 **设置** > **常规**。
1. 选择 **使用增强容器隔离** 复选框。
1. 选择 **应用并重启**。

> [!IMPORTANT]
>
> ECI 不会保护在启用该功能之前创建的容器。在启用 ECI 之前，请删除现有容器。

### 面向管理员

使用设置管理在组织范围内配置增强容器隔离：

{{< tabs >}}
{{< tab name="管理控制台" >}}

1. 登录 [Docker Home](https://app.docker.com) 并从左上角账户下拉菜单中选择您的组织。
1. 转到 **管理控制台** > **桌面设置管理**。
1. [创建或编辑设置策略](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)。
1. 将 **增强容器隔离** 设置为 **始终启用**。

{{< /tab >}}
{{< tab name="JSON 文件" >}}

1. 创建一个 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md) 并添加：

      ```json
      {
        "configurationFileVersion": 2,
        "enhancedContainerIsolation": {
          "value": true,
          "locked": true
        }
      }
      ```

1. 根据需要配置以下选项：
    - `"value": true`：默认启用 ECI（必需）
    - `"locked": true`：防止开发人员关闭 ECI
    - `"locked": false`：允许开发人员控制该设置

{{< /tab >}}
{{< /tabs >}}

### 应用配置

要使 ECI 设置生效：

- **新安装**：用户启动 Docker Desktop 并登录
- **现有安装**：用户必须完全退出 Docker Desktop 并重新启动

> [!IMPORTANT]
>
> 从 Docker Desktop 菜单重启是不够的。用户必须完全退出并重新打开 Docker Desktop。

您还可以为需要 Docker API 访问的可信镜像配置 [Docker 套接字挂载权限](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/config.md)。

## 验证增强容器隔离是否激活

启用 ECI 后，使用这些方法验证其是否正常工作。

### 检查用户命名空间映射

运行容器并检查用户命名空间映射：

```console
$ docker run --rm alpine cat /proc/self/uid_map
```

启用 ECI 时：

```text
0     100000      65536
```

这表明容器的 root 用户（0）映射到 Docker Desktop VM 中的一个非特权用户（100000），范围为 64K 个用户 ID。每个容器都会获得一个独占的用户 ID 范围以实现隔离。

关闭 ECI 时：

```text
0          0 4294967295
```

这表明容器的 root 用户（0）直接映射到 VM 的 root 用户（0），提供的隔离较少。

### 检查容器运行时

验证正在使用的容器运行时：

```console
$ docker inspect --format='{{.HostConfig.Runtime}}' <container_name>
```

启用 ECI 时，它返回 `sysbox-runc`。关闭 ECI 时，它返回 `runc`。

### 测试安全限制

验证 ECI 安全限制是否激活。

测试命名空间共享：

```console
$ docker run -it --rm --pid=host alpine
```

启用 ECI 时，此命令会失败，并显示关于 Sysbox 容器无法与主机共享命名空间的错误。

测试 Docker 套接字访问：

```console
$ docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock alpine
```

启用 ECI 时，除非您已为可信镜像配置了 Docker 套接字异常，否则此命令会失败。

## 强制启用 ECI 时用户看到的内容

当管理员通过设置管理强制启用增强容器隔离时：

- **使用增强容器隔离** 设置在 Docker Desktop 设置中显示为已启用。
- 如果设置为 `"locked": true`，该设置将被锁定并显示为灰色。
- 所有新容器自动使用 Linux 用户命名空间。
- 现有开发工作流程无需修改即可继续工作。
- 用户在 `docker inspect` 输出中看到 `sysbox-runc` 作为容器运行时。

## 下一步

- 查看[配置 Docker 套接字异常和高级设置](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/config.md)。
- 查看[增强容器隔离限制](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/limitations.md)。
