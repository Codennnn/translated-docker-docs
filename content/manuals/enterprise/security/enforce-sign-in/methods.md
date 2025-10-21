---
title: 配置强制登录
linkTitle: 配置
description: 使用注册表项、配置文件、plist 文件或 registry.json 文件配置 Docker Desktop 的强制登录功能
keywords: 身份验证, registry.json, 配置, 强制登录, docker desktop, 安全, .plist, 注册表项, mac, windows, linux
tags: [admin]
aliases:
 - /security/for-admins/enforce-sign-in/methods/
---

{{< summary-bar feature_name="Enforce sign-in" >}}

您可以使用多种方法强制 Docker Desktop 用户登录。请选择最适合您组织基础设施和安全要求的方法。

## 选择适合的方法

| 方法 | 平台 |
|:-------|:---------|
| 注册表项 | 仅限 Windows |
| 配置文件 | 仅限 macOS |
| `plist` 文件 | 仅限 macOS |
| `registry.json` | 所有平台 |

> [!TIP]
>
> 对于 macOS，配置文件提供最高的安全性，因为它们受到 Apple 系统完整性保护（SIP）的保护。

## Windows：注册表项方法

{{< tabs >}}
{{< tab name="手动设置" >}}

手动配置注册表项方法：

1. 创建注册表项：

   ```console
   $ HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Docker\Docker Desktop
   ```
1. 创建一个多字符串值，名称为 `allowedOrgs`。
1. 使用您的组织名称作为字符串数据：
   - 仅使用小写字母
   - 每个组织单独一行
   - 不要使用空格或逗号作为分隔符
1. 重启 Docker Desktop。
1. 验证 Docker Desktop 中是否出现"需要登录！"提示。

> [!IMPORTANT]
>
> Docker Desktop 4.36 及更高版本支持添加多个组织。
在 4.35 及更早版本中，添加多个组织会导致强制登录静默失败。

{{< /tab >}}
{{< tab name="组策略部署" >}}

使用组策略在组织中部署注册表项：

1. 创建包含所需键结构的注册表脚本。
1. 在组策略管理中，创建或编辑 GPO。
1. 导航到**计算机配置** > **首选项** > **Windows 设置** > **注册表**。
1. 右键单击**注册表** > **新建** > **注册表项**。
1. 配置注册表项：
   - 操作：**更新**
   - 路径：`HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Docker\Docker Desktop`
   - 值名称：`allowedOrgs`
   - 值数据：您的组织名称
1. 将 GPO 链接到目标组织单位。
1. 使用 `gpupdate/force` 在小范围内测试。
1. 验证后在整个组织内部署。

{{< /tab >}}
{{< /tabs >}}

## macOS：配置文件方法（推荐）

{{< summary-bar feature_name="Enforce sign-in" >}}

配置文件为 macOS 提供最安全的强制执行方法，因为它们受到 Apple 系统完整性保护的保护。

有效载荷是一个键值字典。Docker Desktop 支持以下键：

- `allowedOrgs`：在一个字符串中设置组织列表，每个组织用分号分隔。

在 Docker Desktop 4.48 及更高版本中，还支持以下键： 

- `overrideProxyHTTP`：设置用于传出 HTTP 请求的 HTTP 代理 URL。
- `overrideProxyHTTPS`：设置用于传出 HTTPS 请求的 HTTP 代理 URL。
- `overrideProxyExclude`：绕过指定主机和域的代理设置。使用逗号分隔的列表。
- `overrideProxyPAC`：设置 PAC 文件所在的文件路径。它的优先级高于所选代理上的远程 PAC 文件。
- `overrideProxyEmbeddedPAC`：设置内存中 PAC 文件的内容。它的优先级高于 `overrideProxyPAC`。

通过配置文件覆盖至少一个代理设置将自动锁定这些设置，因为它们由 macOS 管理。 

1. 创建名为 `docker.mobileconfig` 的文件，内容如下：
   ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
      <dict>
        <key>PayloadContent</key>
        <array>
          <dict>
            <key>PayloadType</key>
            <string>com.docker.config</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
            <key>PayloadIdentifier</key>
            <string>com.docker.config</string>
            <key>PayloadUUID</key>
            <string>eed295b0-a650-40b0-9dda-90efb12be3c7</string>
            <key>PayloadDisplayName</key>
            <string>Docker Desktop Configuration</string>
            <key>PayloadDescription</key>
            <string>Configuration profile to manage Docker Desktop settings.</string>
            <key>PayloadOrganization</key>
            <string>Your Company Name</string>
            <key>allowedOrgs</key>
            <string>first_org;second_org</string>
            <key>overrideProxyHTTP</key>
            <string>http://company.proxy:80</string>
            <key>overrideProxyHTTPS</key>
            <string>https://company.proxy:443</string>
          </dict>
        </array>
        <key>PayloadType</key>
        <string>Configuration</string>
        <key>PayloadVersion</key>
        <integer>1</integer>
        <key>PayloadIdentifier</key>
        <string>com.yourcompany.docker.config</string>
        <key>PayloadUUID</key>
        <string>0deedb64-7dc9-46e5-b6bf-69d64a9561ce</string>
        <key>PayloadDisplayName</key>
        <string>Docker Desktop Config Profile</string>
        <key>PayloadDescription</key>
        <string>Config profile to enforce Docker Desktop settings for allowed organizations.</string>
        <key>PayloadOrganization</key>
        <string>Your Company Name</string>
      </dict>
    </plist>
   ```
1. 替换占位符：
   - 将 `com.yourcompany.docker.config` 更改为您的公司标识符
   - 将 `Your Company Name` 替换为您的组织名称
   - 更新 `allowedOrgs` 值为您的组织名称（用分号分隔）
1. 使用您的 MDM 解决方案部署配置文件。
1. 验证配置文件是否出现在**系统设置** > **通用** > **设备管理**下的**设备（托管）**配置文件中。

## macOS：plist 文件方法

此替代方法适用于 Docker Desktop 4.32 及更高版本的 macOS。

{{< tabs >}}
{{< tab name="手动创建" >}}

1. 创建文件 `/Library/Application Support/com.docker.docker/desktop.plist`。
1. 添加以下内容，将 `myorg1` 和 `myorg2` 替换为您的组织名称：
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
     <dict>
	     <key>allowedOrgs</key>
	     <array>
             <string>myorg1</string>
             <string>myorg2</string>
         </array>
     </dict>
   </plist>
   ```
1. 设置文件权限以防止非管理员用户编辑。
1. 重启 Docker Desktop。
1. 验证 Docker Desktop 中是否出现"需要登录！"提示。

{{< /tab >}}
{{< tab name="Shell 脚本部署" >}}

创建并部署脚本以在组织内部分发：

```bash
#!/bin/bash

# 如果目录不存在则创建
sudo mkdir -p "/Library/Application Support/com.docker.docker"

# 写入 plist 文件
sudo defaults write "/Library/Application Support/com.docker.docker/desktop.plist" allowedOrgs -array "myorg1" "myorg2"

# 设置适当的权限
sudo chmod 644 "/Library/Application Support/com.docker.docker/desktop.plist"
sudo chown root:admin "/Library/Application Support/com.docker.docker/desktop.plist"
```

使用 SSH、远程支持工具或您偏好的部署方法部署此脚本。

{{< /tab >}}
{{< /tabs >}}

## 所有平台：registry.json 方法

registry.json 方法适用于所有平台，并提供灵活的部署选项。

### 文件位置

在适当的位置创建 `registry.json` 文件：

| 平台 | 位置 |
| --- | --- |
| Windows | `/ProgramData/DockerDesktop/registry.json` |
| Mac | `/Library/Application Support/com.docker.docker/registry.json` |
| Linux | `/usr/share/docker-desktop/registry/registry.json` |

### 基本设置

{{< tabs >}}
{{< tab name="手动创建" >}}

1. 确保用户是您的 Docker 组织的成员。
1. 在适合您平台的位置创建 `registry.json` 文件。
1. 添加以下内容，将组织名称替换为您自己的：
      ```json
      {
         "allowedOrgs": ["myorg1", "myorg2"]
      }
      ```
1. 设置文件权限以防止用户编辑。
1. 重启 Docker Desktop。
1. 验证 Docker Desktop 中是否出现"需要登录！"提示。

> [!TIP]
>
> 如果用户在强制登录后启动 Docker Desktop 时遇到问题，
他们可能需要更新到最新版本。

{{< /tab >}}
{{< tab name="命令行设置" >}}

#### Windows（以管理员身份运行 PowerShell）

```shell
Set-Content /ProgramData/DockerDesktop/registry.json '{"allowedOrgs":["myorg1","myorg2"]}'
```

#### macOS

```console
sudo mkdir -p "/Library/Application Support/com.docker.docker"
echo '{"allowedOrgs":["myorg1","myorg2"]}' | sudo tee "/Library/Application Support/com.docker.docker/registry.json"
```

#### Linux

```console
sudo mkdir -p /usr/share/docker-desktop/registry
echo '{"allowedOrgs":["myorg1","myorg2"]}' | sudo tee /usr/share/docker-desktop/registry/registry.json
```

{{< /tab >}}
{{< tab name="安装时设置" >}}

在 Docker Desktop 安装过程中创建 registry.json 文件：

#### Windows

```shell
# PowerShell
Start-Process '.\Docker Desktop Installer.exe' -Wait 'install --allowed-org=myorg'

# 命令提示符
"Docker Desktop Installer.exe" install --allowed-org=myorg
```

#### macOS

```console
sudo hdiutil attach Docker.dmg
sudo /Volumes/Docker/Docker.app/Contents/MacOS/install --allowed-org=myorg
sudo hdiutil detach /Volumes/Docker
```

{{< /tab >}}
{{< /tabs >}}

## 方法优先级

当同一系统上存在多种配置方法时，Docker Desktop 使用以下优先级顺序：

1. 注册表项（仅限 Windows）
2. 配置文件（仅限 macOS）
3. plist 文件（仅限 macOS）
4. registry.json 文件

> [!IMPORTANT]
>
> Docker Desktop 4.36 及更高版本支持在单个配置中添加多个组织。早期版本（4.35 及以下）在指定多个组织时会静默失败。

## 强制登录故障排除

如果强制登录不起作用：

- 验证文件位置和权限
- 检查组织名称是否使用小写字母
- 重启 Docker Desktop 或重启系统
- 确认用户是指定组织的成员
- 将 Docker Desktop 更新到最新版本
