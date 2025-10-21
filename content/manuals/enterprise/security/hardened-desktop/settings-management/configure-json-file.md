---
title: 使用JSON文件配置设置管理
linkTitle: 使用JSON文件
description: 使用admin-settings.json文件配置并强制执行Docker Desktop设置
keywords: 管理控制, 设置管理, 配置, 企业, docker desktop, json文件
weight: 10
aliases:
 - /desktop/hardened-desktop/settings-management/configure/
 - /security/for-admins/hardened-desktop/settings-management/configure/
 - /security/for-admins/hardened-desktop/settings-management/configure-json-file/
---

{{< summary-bar feature_name="Hardened Docker Desktop" >}}

设置管理允许您使用`admin-settings.json`文件在整个组织内配置并强制执行Docker Desktop设置。这可以标准化Docker Desktop环境，确保所有用户使用一致的配置。

## 先决条件

在开始之前，请确保您具备以下条件：

- 为您的组织[强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)
- Docker Business订阅

只有当身份验证和Docker Business许可证检查都成功时，Docker Desktop才会应用`admin-settings.json`文件中的设置。

> [!IMPORTANT]
>
> 用户必须登录并属于Docker Business组织。如果任一条件不满足，设置文件将被忽略。

## 步骤一：创建设置文件

您可以通过两种方式创建`admin-settings.json`文件：

- 使用`--admin-settings`安装程序标志自动生成文件：
    - [macOS](/manuals/desktop/setup/install/mac-install.md#install-from-the-command-line) 安装指南
    - [Windows](/manuals/desktop/setup/install/windows-install.md#install-from-the-command-line) 安装指南
- 手动创建文件并将其放置在以下位置：
    - Mac: `/Library/Application\ Support/com.docker.docker/admin-settings.json`
    - Windows: `C:\ProgramData\DockerDesktop\admin-settings.json`
    - Linux: `/usr/share/docker-desktop/admin-settings.json`

> [!IMPORTANT]
>
> 将文件放置在受保护的目录中，防止未经授权的更改。使用Jamf等移动设备管理(MDM)工具在整个组织范围内大规模分发文件。

## 步骤二：配置设置

> [!TIP]
>
> 有关可用设置的完整列表、支持的平台以及它们适用的配置方法，请参阅[设置参考](settings-reference.md)。

`admin-settings.json`文件使用结构化键来定义可配置设置以及是否强制执行值。

每个设置都支持一个`locked`字段，用于控制用户权限：

- 当`locked`设置为`true`时，用户无法在Docker Desktop、CLI或配置文件中更改该值。
- 当`locked`设置为`false`时，该值作为默认建议，用户仍然可以更新它。

如果用户已经在`settings-store.json`、`settings.json`或`daemon.json`中自定义了该值，则在现有安装上会忽略`locked`设置为`false`的设置。

### 分组设置

Docker Desktop将一些设置分组在一起，使用单个开关控制整个部分。这些包括：

- 增强容器隔离(ECI)：使用主开关(`enhancedContainerIsolation`)启用/禁用整个功能，并为特定配置提供子设置
- Kubernetes：使用主开关(`kubernetes.enabled`)和集群配置的子设置
- Docker Scout：在`scout`对象下分组设置

配置分组设置时：

1. 设置主开关以启用功能
1. 在该组内配置子设置
1. 当您锁定主开关时，用户无法修改该组中的任何设置

`enhancedContainerIsolation`的示例：

```json
"enhancedContainerIsolation": {
  "locked": true,  // 这会锁定整个ECI部分
  "value": true,   // 这会启用ECI
  "dockerSocketMount": {  // 这些是子设置
    "imageList": {
      "images": ["docker.io/testcontainers/ryuk:*"]
    }
  }
}
```

### `admin-settings.json`文件示例

以下示例是配置了常见企业设置的`admin-settings.json`文件。您可以将此示例作为模板，参考[`admin-settings.json`配置](#admin-settingsjson-配置)：

```json {collapse=true}
{
  "configurationFileVersion": 2,
  "exposeDockerAPIOnTCP2375": {
    "locked": true,
    "value": false
  },
  "proxy": {
    "locked": true,
    "mode": "system",
    "http": "",
    "https": "",
    "exclude": [],
    "windowsDockerdPort": 65000,
    "enableKerberosNtlm": false,
    "pac": "",
    "embeddedPac": ""
  },
  "containersProxy": {
    "locked": true,
    "mode": "manual",
    "http": "",
    "https": "",
    "exclude": [],
    "pac":"",
    "embeddedPac": "",
    "transparentPorts": ""
  },
  "enhancedContainerIsolation": {
    "locked": true,
    "value": true,
    "dockerSocketMount": {
      "imageList": {
        "images": [
          "docker.io/localstack/localstack:*",
          "docker.io/testcontainers/ryuk:*"
        ]
      },
      "commandList": {
        "type": "deny",
        "commands": ["push"]
      }
    }
  },
  "linuxVM": {
    "wslEngineEnabled": {
      "locked": false,
      "value": false
    },
    "dockerDaemonOptions": {
      "locked": false,
      "value":"{\"debug\": false}"
    },
    "vpnkitCIDR": {
      "locked": false,
      "value":"192.168.65.0/24"
    }
  },
  "kubernetes": {
     "locked": false,
     "enabled": false,
     "showSystemContainers": false,
     "imagesRepository": ""
  },
  "windowsContainers": {
    "dockerDaemonOptions": {
      "locked": false,
      "value":"{\"debug\": false}"
    }
  },
  "disableUpdate": {
    "locked": false,
    "value": false
  },
  "analyticsEnabled": {
    "locked": false,
    "value": true
  },
  "extensionsEnabled": {
    "locked": true,
    "value": false
  },
  "scout": {
    "locked": false,
    "sbomIndexing": true,
    "useBackgroundIndexing": true
  },
  "allowBetaFeatures": {
    "locked": false,
    "value": false
  },
  "blockDockerLoad": {
    "locked": false,
    "value": true
  },
  "filesharingAllowedDirectories": [
    {
      "path": "$HOME",
      "sharedByDefault": true
    },
    {
      "path":"$TMP",
      "sharedByDefault": false
    }
  ],
  "useVirtualizationFrameworkVirtioFS": {
    "locked": true,
    "value": true
  },
  "useVirtualizationFrameworkRosetta": {
    "locked": true,
    "value": true
  },
  "useGrpcfuse": {
    "locked": true,
    "value": true
  },
  "displayedOnboarding": {
    "locked": true,
    "value": true
  },
  "desktopTerminalEnabled": {
    "locked": false,
    "value": false
  },
  "enableInference": {
    "locked": false,
    "value": true
  },
  "enableInferenceTCP": {
    "locked": false,
    "value": true
  },
  "enableInferenceTCPPort": {
    "locked": false,
    "value": 12434
  },
  "enableInferenceCORS": {
    "locked": false,
    "value": ""
  },
  "enableInferenceGPUVariant": {
    "locked": false,
    "value": true
  }
}
```

## 步骤三：应用设置

设置在Docker Desktop重启且用户登录后生效。

对于新安装：

1. 启动Docker Desktop。
1. 使用您的Docker账户登录。

对于现有安装：

1. 完全退出Docker Desktop。
1. 重新启动Docker Desktop。

> [!IMPORTANT]
>
> 您必须完全退出并重新打开Docker Desktop。从菜单重启是不够的。

## `admin-settings.json`配置

下表描述了`admin-settings.json`文件中所有可用的设置。

> [!NOTE]
>
> 有些设置是平台特定的或需要最低Docker Desktop版本。请检查版本列以了解要求。

### 常规设置

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
|`configurationFileVersion`|   |指定配置文件格式的版本。|   |
|`analyticsEnabled`|  |如果`value`设置为false，Docker Desktop不会向Docker发送使用统计信息。 |  |
|`disableUpdate`|  |如果`value`设置为true，则禁用检查和通知Docker Desktop更新。|  |
|`extensionsEnabled`|  |如果`value`设置为false，则禁用Docker扩展。 |  |
| `blockDockerLoad` | | 如果`value`设置为`true`，用户将无法运行[`docker load`](/reference/cli/docker/image/load/)并在尝试时收到错误。|  |
| `displayedOnboarding` |  | 如果`value`设置为`true`，新用户将不会显示入门调查。将`value`设置为`false`没有效果。 |  Docker Desktop 4.30及更高版本 |
| `desktopTerminalEnabled` |  | 如果`value`设置为`false`，开发者无法使用Docker终端与主机交互并直接从Docker Desktop执行命令。 |  |
|`exposeDockerAPIOnTCP2375`| 仅Windows| 在指定端口上公开Docker API。如果`value`设置为true，Docker API将在端口2375上公开。注意：这是未经身份验证的，只有在受到适当防火墙规则保护时才应启用。|  |
| `silentModulesUpdate` | | 如果`value`设置为`true`，Docker Desktop会自动更新不需要重启的组件。例如，Docker CLI或Docker Scout组件。 | Docker Desktop 4.46及更高版本。 |

### 文件共享和模拟

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
| `filesharingAllowedDirectories` |  | 指定您的开发者可以添加文件共享的路径。还接受`$HOME`、`$TMP`或`$TEMP`作为`path`变量。添加路径时，允许其子目录。如果`sharedByDefault`设置为`true`，则该路径将在出厂重置或Docker Desktop首次启动时添加。 |  |
| `useVirtualizationFrameworkVirtioFS`|  仅macOS | 如果`value`设置为`true`，VirtioFS被设置为文件共享机制。注意：如果`useVirtualizationFrameworkVirtioFS`和`useGrpcfuse`的`value`都设置为`true`，则VirtioFS优先。同样，如果`useVirtualizationFrameworkVirtioFS`和`useGrpcfuse`的`value`都设置为`false`，则osxfs被设置为文件共享机制。 |  |
| `useGrpcfuse` | 仅macOS | 如果`value`设置为`true`，gRPC Fuse被设置为文件共享机制。 |  |
| `useVirtualizationFrameworkRosetta`|  仅macOS | 如果`value`设置为`true`，Docker Desktop会打开Rosetta以加速Apple Silicon上的x86_64/amd64二进制模拟。注意：这也会自动启用"使用虚拟化框架"。 | Docker Desktop 4.29及更高版本。 |

### Docker Scout

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
|`scout`| | 将`useBackgroundIndexing`设置为`false`会禁用加载到镜像存储库的镜像的自动索引。将`sbomIndexing`设置为`false`会阻止用户通过在Docker Desktop中检查镜像或使用`docker scout` CLI命令来索引镜像。 |  |

### 代理设置

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
|`proxy`|   |如果`mode`设置为`system`而不是`manual`，Docker Desktop从系统获取代理值，并忽略为`http`、`https`和`exclude`设置的值。将`mode`更改为`manual`以手动配置代理服务器。如果代理端口是自定义的，请在`http`或`https`属性中指定，例如`"https": "http://myotherproxy.com:4321"`。`exclude`属性指定绕过代理的主机和域的逗号分隔列表。 |  |
| `windowsDockerdPort`| 仅Windows | 在此端口上本地公开Docker Desktop的内部代理，供Windows Docker守护程序连接。如果设置为0，则选择一个随机空闲端口。如果值大于0，则使用该确切值作为端口。默认值为-1，禁用此选项。 |  |
|`enableKerberosNtlm`|  |设置为`true`时，启用Kerberos和NTLM身份验证。默认为`false`。有关更多信息，请参阅设置文档。 | Docker Desktop 4.32及更高版本。 |
| `pac` | | 指定PAC文件URL。例如，`"pac": "http://proxy/proxy.pac"`。 | |
| `embeddedPac`  | | 指定嵌入式PAC（代理自动配置）脚本。例如，`"embeddedPac": "function FindProxyForURL(url, host) { return \"DIRECT\"; }"`。此设置优先于HTTP、HTTPS、代理绕过和PAC服务器URL。 |  Docker Desktop 4.46及更高版本。 |

### 容器代理

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
|`containersProxy` | | 创建隔离网络容器。有关更多信息，请参阅[隔离网络容器](../air-gapped-containers.md)。| Docker Desktop 4.29及更高版本。 |
| `pac` | | 指定PAC文件URL。例如，`"pac": "http://containerproxy/proxy.pac"`。 | |
| `embeddedPac`  | | 指定嵌入式PAC（代理自动配置）脚本。例如，`"embeddedPac": "function FindProxyForURL(url, host) { return \"PROXY 192.168.92.1:2003\"; }"`。此设置优先于HTTP、HTTPS、代理绕过和PAC服务器URL。 |  Docker Desktop 4.46及更高版本。 |

### Linux VM设置

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
| `linuxVM` |   |与Linux VM选项相关的参数和设置 - 为方便起见在此分组。 |  |
| &nbsp; &nbsp; &nbsp; &nbsp;`wslEngineEnabled`  | 仅Windows | 如果`value`设置为true，Docker Desktop使用基于WSL 2的引擎。这会覆盖在安装时使用`--backend=<backend name>`标志可能设置的任何内容。 |  |
| &nbsp; &nbsp; &nbsp; &nbsp;`dockerDaemonOptions` |  |如果`value`设置为true，它会覆盖Docker Engine配置文件中的选项。请参阅[Docker Engine参考](/reference/cli/dockerd/#daemon-configuration-file)。请注意，为了增加安全性，当启用增强容器隔离时，可能会覆盖一些配置属性。 |  |
| &nbsp; &nbsp; &nbsp; &nbsp;`vpnkitCIDR` |  |覆盖用于`*.docker.internal`的vpnkit DHCP/DNS的网络范围 |  |

### Windows容器

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
| `windowsContainers` |  | 与`windowsContainers`选项相关的参数和设置 - 为方便起见在此分组。 |  |
| &nbsp; &nbsp; &nbsp; &nbsp;`dockerDaemonOptions` |  | 覆盖Linux守护程序配置文件中的选项。请参阅[Docker Engine参考](/reference/cli/dockerd/#daemon-configuration-file)。|  |

> [!NOTE]
>
> 此设置无法通过Docker管理控制台进行配置。

### Kubernetes设置

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
|`kubernetes`|  | 如果`enabled`设置为true，则在Docker Desktop启动时启动Kubernetes单节点集群。如果`showSystemContainers`设置为true，Kubernetes容器将显示在Docker Desktop仪表板中以及运行`docker ps`时。[imagesRepository](../../../../desktop/features/kubernetes.md#configuring-a-custom-image-registry-for-kubernetes-control-plane-images)设置让您指定Docker Desktop从哪个存储库拉取控制平面Kubernetes镜像。 |  |

> [!NOTE]
>
> 将`imagesRepository`与增强容器隔离(ECI)一起使用时，请将这些镜像添加到[ECI Docker套接字挂载镜像列表](#增强容器隔离)：
>
> `[imagesRepository]/desktop-cloud-provider-kind:`
> `[imagesRepository]/desktop-containerd-registry-mirror:`
>
> 这些容器挂载Docker套接字，因此必须将它们添加到ECI镜像列表中。否则，ECI会阻止挂载，Kubernetes将无法启动。

### 网络设置

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
| `defaultNetworkingMode` | 仅Windows和Mac | 定义新Docker网络的默认IP协议：`dual-stack`（IPv4 + IPv6，默认）、`ipv4only`或`ipv6only`。 | Docker Desktop 4.43及更高版本。 |
| `dnsInhibition` | 仅Windows和Mac | 控制返回给容器的DNS记录过滤。选项：`auto`（推荐）、`ipv4`、`ipv6`、`none`| Docker Desktop 4.43及更高版本。 |

有关更多信息，请参阅[网络](/manuals/desktop/features/networking.md#networking-mode-and-dns-behaviour-for-mac-and-windows)。

### AI设置

| 参数                   | 操作系统            | 描述                                                                                                                                                                                                                         | 版本 |
|:----------------------------|---------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| `enableInference`           |               | 将`enableInference`设置为`true`会启用[Docker模型运行器](/manuals/ai/model-runner/_index.md)。                                                                                                                              |         |
| `enableInferenceTCP`        |               | 启用主机端TCP支持。此设置需要先启用Docker模型运行器设置。                                                                                                                                |         |
| `enableInferenceTCPPort`    |               | 指定公开的TCP端口。此设置需要先启用Docker模型运行器和启用主机端TCP支持设置。                                                                                            |         |
| `enableInferenceCORS`       |               | 指定允许的CORS源。空字符串表示拒绝所有，`*`表示接受所有，或逗号分隔的值列表。此设置需要先启用Docker模型运行器和启用主机端TCP支持设置。       |         |
| `enableInferenceGPUVariant` | 仅Windows  | 将`enableInferenceGPUVariant`设置为`true`会启用GPU支持的推理。此功能所需的额外组件默认不随Docker Desktop提供，因此它们将被下载到`~/.docker/bin/inference`。  |         |

### Beta功能

> [!IMPORTANT]
>
> 对于Docker Desktop 4.41及更早版本，其中一些设置位于**开发中功能**页面的**实验性功能**选项卡下。

| 参数                                            | 操作系统 | 描述                                                                                                                                                                                                                                               | 版本                                 |
|:-----------------------------------------------------|----|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| `allowBetaFeatures`                                  |    | 如果`value`设置为`true`，则启用beta功能。                                                                                                                                                                                                   |                                         |
| `enableDockerAI`                                     |    | 如果`allowBetaFeatures`为true，将`enableDockerAI`设置为`true`会默认启用[Docker AI (Ask Gordon)](/manuals/ai/gordon/_index.md)。您可以独立控制此设置，不受`allowBetaFeatures`设置影响。                            |                                         |
| `enableDockerMCPToolkit`                             |    | 如果`allowBetaFeatures`为true，将`enableDockerMCPToolkit`设置为`true`会默认启用[MCP工具包功能](/manuals/ai/mcp-catalog-and-toolkit/toolkit.md)。您可以独立控制此设置，不受`allowBetaFeatures`设置影响。 |                                         |
| `allowExperimentalFeatures`                          |    | 如果`value`设置为`true`，则启用实验性功能。                                                                                                                                                                                           | Docker Desktop 4.41及更早版本 |

### 增强容器隔离

|参数|操作系统|描述|版本|
|:-------------------------------|---|:-------------------------------|---|
|`enhancedContainerIsolation`|  | 如果`value`设置为true，Docker Desktop会通过Linux用户命名空间以非特权模式运行所有容器，防止它们修改Docker Desktop VM内的敏感配置，并使用其他高级技术进行隔离。更多信息，请参阅[增强容器隔离](../enhanced-container-isolation/_index.md)。|  |
| &nbsp; &nbsp; &nbsp; &nbsp;`dockerSocketMount` |  | 默认情况下，增强容器隔离会阻止将Docker Engine套接字绑定挂载到容器中（例如，`docker run -v /var/run/docker.sock:/var/run/docker.sock ...`）。这允许您以受控方式放宽此限制。更多信息，请参阅[ECI配置](../enhanced-container-isolation/config.md)。 |  |
| &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; `imageList` |  | 指示哪些容器镜像被允许绑定挂载Docker Engine套接字。 |  |
| &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; `commandList` |  | 限制容器可以通过绑定挂载的Docker Engine套接字发出的命令。 |  |
