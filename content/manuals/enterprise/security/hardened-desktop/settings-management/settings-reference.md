---
title: 设置参考
linkTitle: 设置参考
description: Docker Desktop 所有设置和配置选项的完整参考指南
keywords: docker desktop 设置, 配置参考, 管理员控制, 设置管理
aliases:
 - /security/for-admins/hardened-desktop/settings-management/settings-reference/
---

本参考文档详细介绍了 Docker Desktop 的所有设置和配置选项，帮助您了解不同配置方法和平台上的设置行为。文档结构按照 Docker Desktop 图形界面的布局组织。

每个设置包含以下信息：

- 默认值和可接受值
- 平台兼容性
- 配置方法（Docker Desktop 图形界面、管理控制台、`admin-settings.json` 文件或命令行界面）
- 企业安全建议（如适用）

## 常规设置

### 登录计算机时自动启动 Docker Desktop

| 默认值 | 可接受值 | 格式 |
|---------------|-----------------|--------|
| `false`       | `true`, `false` | 布尔值 |

- **说明：** 用户登录计算机时自动启动 Docker Desktop。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 确保 Docker Desktop 在系统启动后始终可用。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

### Docker Desktop 启动时打开 Docker 仪表板

| 默认值 | 可接受值            | 格式 |
|---------------|----------------------------|--------|
| `false`      | `true`, `false`  | 布尔值   |

- **说明：** Docker Desktop 启动时是否自动打开 Docker 仪表板。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 启动后立即访问容器、镜像和卷。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

### 选择 Docker Desktop 主题

| 默认值 | 可接受值            | 格式 |
|---------------|----------------------------|--------|
| `system`      | `light`, `dark`, `system`  | 枚举   |

- **说明：** Docker Desktop 界面的视觉外观。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 根据用户偏好或系统主题自定义界面外观。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

### 配置 Shell 自动补全

| 默认值 | 可接受值         | 格式 |
|---------------|-------------------------|--------|
| `integrated`  | `integrated`, `system`  | 字符串 |

- **说明：** Docker CLI 自动补全与用户 Shell 的集成方式。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 控制是否让 Docker 修改 Shell 配置文件以实现自动补全。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

### 选择容器终端

| 默认值 | 可接受值         | 格式 |
|---------------|-------------------------|--------|
| `integrated`  | `integrated`, `system`  | 字符串 |

- **说明：** 从 Docker Desktop 启动 Docker CLI 时使用的默认终端。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 为 Docker CLI 交互设置首选终端应用程序。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

### 启用 Docker 终端

| 默认值 | 可接受值 | 格式 |
|---------------|-----------------|--------|
| `false`       | `true`, `false` | 布尔值 |

- **说明：** 访问 Docker Desktop 的集成终端功能。如果值设置为 `false`，用户无法使用 Docker 终端与主机交互并直接从 Docker Desktop 执行命令。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 允许或限制开发者使用内置终端进行主机系统交互。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `desktopTerminalEnabled` 设置

> [!NOTE]
>
> 在加固环境中，禁用并锁定此设置以限制主机访问。

### 默认启用 Docker 调试

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **说明：** 是否默认为 Docker CLI 命令启用调试日志。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 为故障排除和支持场景提供详细输出。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

### 在 Time Machine 备份中包含虚拟机

| 默认值 | 可接受值 | 格式 |
|---------------|-----------------|--------|
| `false`       | `true`, `false` | 布尔值 |

- **说明：** Docker Desktop 虚拟机是否包含在 macOS Time Machine 备份中。
- **操作系统：** {{< badge color=blue text="仅 Mac" >}}
- **使用场景：** 平衡备份完整性与备份大小和性能。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

### 使用 containerd 拉取和存储镜像

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **说明：** Docker Desktop 使用的镜像存储后端。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 提高镜像处理性能并启用 containerd 原生功能。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

### 选择虚拟机管理器

#### Docker VMM

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

#### Apple 虚拟化框架

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **说明：** 使用 Apple 虚拟化框架运行 Docker 容器。
- **操作系统：** {{< badge color=blue text="仅 Mac" >}}
- **使用场景：** 在 Apple Silicon 上提高虚拟机性能。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中

#### Rosetta

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **说明：** 使用 Rosetta 在 Apple Silicon 上模拟 `amd64`。如果值设置为 `true`，Docker Desktop 会开启 Rosetta 以加速在 Apple Silicon 上的 x86_64/amd64 二进制模拟。
- **操作系统：** {{< badge color=blue text="仅 Mac" >}} 13+
- **使用场景：** 在 Apple Silicon 主机上运行基于 Intel 的容器。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规**设置中
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `useVirtualizationFrameworkRosetta` 设置
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **在 Apple Silicon 上使用 Rosetta 进行 x86_64/amd64 模拟** 设置

> [!NOTE]
>
> 在加固环境中，禁用并锁定此设置，以便只允许 ARM 原生镜像。

> [!NOTE]
>
> Rosetta 需要启用 Apple 虚拟化框架。

#### QEMU

> [!WARNING]
>
> QEMU 在 Docker Desktop 4.44 及更高版本中已被弃用。更多信息，请参阅 [博客公告](https://www.docker.com/blog/docker-desktop-for-mac-qemu-virtualization-option-to-be-deprecated-in-90-days/)

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

### 选择文件共享实现方式

#### VirtioFS

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **说明：** 使用 VirtioFS 实现主机和容器之间的快速、原生文件共享。如果值设置为 `true`，VirtioFS 被设置为文件共享机制。如果 VirtioFS 和 gRPC 都设置为 `true`，VirtioFS 优先。
- **操作系统：** {{< badge color=blue text="仅 Mac" >}} 12.5+
- **使用场景：** 在现代 macOS 上实现更好的文件系统性能和兼容性。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规设置**中
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `useVirtualizationFrameworkVirtioFS` 设置
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **使用 VirtioFS 进行文件共享** 设置

> [!NOTE]
>
> 在加固环境中，对于 macOS 12.5 及更高版本，启用并锁定此设置。

#### gRPC FUSE

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **说明：** 启用 gRPC FUSE 进行 macOS 文件共享。如果值设置为 `true`，gRPC FUSE 被设置为文件共享机制。
- **操作系统：** {{< badge color=blue text="仅 Mac" >}}
- **使用场景：** 提供比传统 osxfs 性能更好的替代文件共享方案。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规设置**中
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `useGrpcfuse` 设置
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **使用 gRPC FUSE 进行文件共享** 设置

> [!NOTE]
>
> 在加固环境中，禁用并锁定此设置。

#### osxfs

| 默认值 | 可接受值 | 格式  |
| ------------- | --------------- | ------- |
| `false`       | `true`, `false` | 布尔值 |

- **说明：** 使用原始的 osxfs 文件共享驱动程序进行 macOS 文件共享。当设置为 true 时，Docker Desktop 使用 osxfs 而不是 VirtioFS 或 gRPC FUSE 将主机目录挂载到容器中。
- **操作系统：** {{< badge color=blue text="仅 Mac" >}}
- **使用场景：** 与需要原始文件共享实现的旧工具兼容。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规设置**中

### 发送使用统计信息

| 默认值 | 可接受值 | 格式 |
|---------------|-----------------|--------|
| `true`        | `true`, `false` | 布尔值 |

- **说明：** 控制 Docker Desktop 是否收集并向 Docker 发送本地使用统计信息和崩溃报告。此设置影响从 Docker Desktop 应用程序本身收集的遥测数据。它不影响通过 Docker Hub 或其他后端服务（如登录时间戳、拉取或构建）收集的服务器端遥测数据。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 帮助 Docker 根据使用模式改进产品。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规设置**中
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `analyticsEnabled` 设置
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **发送使用统计信息** 设置

> [!NOTE]
>
> 在加固环境中，禁用并锁定此设置。这允许您控制所有数据流，并在需要时通过安全通道收集支持日志。

> [!NOTE]
>
> 使用 Insights Dashboard 的组织可能需要启用此设置，以确保开发者活动完全可见。如果用户选择退出且设置未被锁定，他们的活动可能会被排除在分析视图之外。

### 使用增强容器隔离

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **说明：** 通过 Linux 用户命名空间和额外隔离实现高级容器安全。
- **操作系统：** {{< badge color=blue text="所有" >}}
- **使用场景：** 防止容器修改 Docker Desktop VM 配置或访问敏感主机区域。
- **配置方式：**
    - 在 [Docker Desktop 图形界面](/manuals/desktop/settings-and-maintenance/settings.md)的**常规设置**中
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enhancedContainerIsolation` 设置
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **启用增强容器隔离** 设置

> [!NOTE]
>
> 在加固环境中，禁用并锁定此设置。这允许您控制所有数据流，并在需要时通过安全通道收集支持日志。

### 显示 CLI 提示

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`       | `true`, `false` | 布尔值  |

- **描述：** 在使用 Docker 命令时，在终端中显示有用的 CLI 建议。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 通过上下文提示帮助用户发现 Docker CLI 功能。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **常规** 设置

### 启用 Scout 镜像分析

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 为容器镜像生成 Docker Scout SBOM 并进行漏洞扫描。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 启用漏洞扫描和软件物料清单分析。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **常规设置**
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `sbomIndexing` 设置
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **SBOM 索引** 设置

> [!NOTE]
>
> 在加固环境中，启用并锁定此设置以确保合规扫描始终可用。

### 启用后台 Scout SBOM 索引

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`        | `true`, `false` | 布尔值  |

- **描述：** 自动为镜像进行 SBOM 索引，无需用户交互。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 通过在空闲时间或镜像操作后进行索引，保持镜像元数据最新。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **常规设置**

> [!NOTE]
>
> 在加固环境中，启用并锁定此设置以进行持续的安全分析。

### 自动检查配置

| 默认值         | 可接受值 | 格式  |
|-----------------------|-----------------|---------|
| `CurrentSettingsVersions` | 整数         | 整数 |

- **描述：** 定期验证 Docker Desktop 配置是否未被外部应用程序修改。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 跟踪配置版本以进行兼容性和更改检测。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **常规** 设置
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `configurationFileVersion` 设置

## 资源设置

### CPU 限制

| 默认值                                 | 可接受值 | 格式  |
|-----------------------------------------------|-----------------|---------|
| 主机上可用的逻辑 CPU 核心数 | 整数         | 整数 |

- **描述：** 分配给 Docker Desktop 虚拟机的 CPU 核心数。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 平衡 Docker 性能与主机系统资源可用性。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **高级** 资源设置

### 内存限制

| 默认值              | 可接受值 | 格式  |
|---------------------------|-----------------|---------|
| 基于系统资源 | 整数         | 整数 |

- **描述：** 分配给 Docker Desktop 虚拟机的 RAM 量（以 MiB 为单位）。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 控制内存分配以优化 Docker 和主机应用程序的性能。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **高级** 资源设置

### 交换空间

| 默认值 | 可接受值 | 格式  |
|---------------|-----------------|---------|
| `1024`        | 整数         | 整数 |

- **描述：** Docker 虚拟机可用的交换空间量（以 MiB 为单位）。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 当物理 RAM 有限时，扩展容器工作负载的可用内存。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **高级** 资源设置

### 磁盘使用限制

| 默认值                  | 可接受值 | 格式  |
|-------------------------------|-----------------|---------|
| 机器的默认磁盘大小。 | 整数         | 整数 |

- **描述：** 为 Docker Desktop 数据分配的最大磁盘空间（以 MiB 为单位）。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 防止 Docker 在主机系统上消耗过多的磁盘空间。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **高级** 资源设置

### 磁盘镜像位置

| 默认值                                                                 | 可接受值 | 格式 |
|--------------------------------------------------|-----------------|--------|
| macOS: `~/Library/Containers/com.docker.docker/Data/vms/0`  <br> Windows: `%USERPROFILE%\AppData\Local\Docker\wsl\data` | 文件路径       | 字符串 |

- **描述：** Docker Desktop 存储虚拟机数据的文件系统路径。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 将 Docker 数据移动到自定义存储位置以进行性能或空间管理。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **高级** 资源设置

### 启用资源节省器

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 当空闲时自动暂停 Docker Desktop 以节省系统资源。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 当 Docker Desktop 未被主动使用时减少 CPU 和内存使用。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **高级** 资源设置

### 文件共享目录

| 默认值                           | 可接受值                 | 格式                  |
|----------------------------------------|---------------------------------|--------------------------|
| 因操作系统而异                           | 字符串形式的文件路径列表   | 字符串数组列表   |

- **描述：** 可以作为卷挂载到容器中的主机目录。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 定义容器可以访问的主机目录，用于开发工作流。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **文件共享** 资源设置
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `filesharingAllowedDirectories` 设置
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **允许的文件共享目录** 设置

> [!NOTE]
>
> 在加固环境中，锁定到明确的允许列表并禁用最终用户编辑。

### 代理排除

| 默认值 | 可接受值    | 格式 |
|---------------|--------------------|--------|
| `""`          | 地址列表  | 字符串 |

- **描述：** 容器在使用代理设置时应绕过的网络地址。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 为内部服务或特定域定义代理例外。
- **配置方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md) 中的 **代理** 资源设置
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `proxy` 设置，使用 `manual` 和 `exclude` 模式
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **代理** 部分

> [!NOTE]
>
> 在加固环境中，禁用并锁定此设置以保持严格的代理控制。

### Docker 子网

| 默认值     | 可接受值 | 格式     |
|-------------------|-----------------|--------|
| `192.168.65.0/24` | IP 地址      | 字符串     |

- **描述：** 覆盖用于 `*.docker.internal` 的 vpnkit DHCP/DNS 的网络范围。
- **操作系统：** {{< badge color=blue text="仅 Mac" >}}
- **使用场景：** 自定义用于 Docker 容器网络的子网。
- **配置方式：**
    - 设置管理：在 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `vpnkitCIDR` 设置
    - 设置管理：在 [管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的 **VPN Kit CIDR** 设置

### 为 UDP 使用内核网络

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 使用主机的内核网络堆栈处理 UDP 流量，而不是 Docker 的虚拟网络驱动程序。 这可以实现更快、更直接的 UDP 通信，但可能会绕过一些容器隔离功能。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 提高实时媒体、DNS 或游戏等 UDP 密集型应用程序的性能。
- **配置此设置的方式：**
    - **网络**资源设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中

### 启用主机网络

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 实验性支持，允许容器直接使用主机的网络堆栈。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 允许容器在特定场景下绕过 Docker 的网络隔离。
- **配置此设置的方式：**
    - **网络**资源设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中

### 网络模式

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `dual-stack` | `ipv4only`, `ipv6only` | 字符串  |

- **描述：** Docker 创建新网络时使用的默认 IP 协议。
- **操作系统：** {{< badge color=blue text="Windows 和 Mac" >}}
- **使用场景：** 与仅支持 IPv4 或 IPv6 的网络基础设施保持一致。
- **配置此设置的方式：**
    - **网络**资源设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `defaultNetworkingMode` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**默认网络 IP 模式**

更多信息，请参阅[网络](/manuals/desktop/features/networking.md#networking-mode-and-dns-behaviour-for-mac-and-windows)。

#### 抑制 IPv4/IPv6 的 DNS 解析

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `auto` | `ipv4`, `ipv6`, `none` | 字符串  |

- **描述：** 过滤不支持的 DNS 记录类型。需要 Docker Desktop 4.43 及更高版本。
- **操作系统：** {{< badge color=blue text="Windows 和 Mac" >}}
- **使用场景：** 控制 Docker 如何过滤返回给容器的 DNS 记录，在仅支持 IPv4 或 IPv6 的环境中提高可靠性。
- **配置此设置的方式：**
    - **网络**资源设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `dnsInhibition` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**DNS 过滤行为**

For more information, see [Networking](/manuals/desktop/features/networking.md#networking-mode-and-dns-behaviour-for-mac-and-windows).

### 启用 WSL 引擎

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 如果值设置为 `true`，Docker Desktop 使用基于 WSL2 的引擎。这会覆盖安装时可能使用 `--backend=<backend name>` 标志设置的任何内容。
- **操作系统：** {{< badge color=blue text="仅 Windows" >}} + WSL
- **使用场景：** 使用 WSL 2 后端在 Windows 上运行 Linux 容器以获得更好的性能。
- **配置此设置的方式：**
    - **WSL 集成**资源设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `wslEngineEnabled` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**Windows 子系统 for Linux (WSL) 引擎**设置

> [!NOTE]
>
> 在加固环境中，启用并锁定此设置以提高安全性和性能。

## Docker 引擎设置

Docker 引擎设置允许您通过原始 JSON 对象配置低级守护进程设置。这些设置直接传递给为 Docker Desktop 中的容器管理提供支持的 dockerd 进程。

| 键                    | 示例                        | 描述                                        | 可接受值 / 格式       | 默认值 |
| --------------------- | --------------------------- | -------------------------------------------------- | ------------------------------ | ------- |
| --------------------- | --------------------------- | -------------------------------------------------- | ------------------------------ | ------- |
| `debug`               | `true`                      | 在 Docker 守护进程中启用详细日志        | 布尔值                        | `false` |
| `experimental`        | `true`                      | 启用实验性 Docker CLI 和守护进程功能 | 布尔值                        | `false` |
| `insecure-registries` | `["myregistry.local:5000"]` | 允许从没有 TLS 的 HTTP 注册表拉取     | 字符串数组 (`host:port`) | `[]`    |
| `registry-mirrors`    | `["https://mirror.gcr.io"]` | 定义替代注册表端点              | URL 数组                  | `[]`    |

- **描述：** 使用直接传递给 dockerd 的结构化 JSON 配置自定义 Docker 守护进程的行为。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 配置注册表访问、启用调试日志或打开实验性功能。
- **配置此设置的方式：**
    - **Docker 引擎**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中

> [!NOTE]
>
> 在加固环境中，提供经过审查的配置并锁定它以防止
未经授权的守护进程修改。

> [!IMPORTANT]
>
> 此设置的值按原样传递给 Docker 守护进程。无效或不支持的字段可能会阻止 Docker Desktop 启动。

## 构建器设置

构建器设置允许您管理 Buildx 构建器实例，用于高级镜像构建场景，包括多平台构建和自定义后端。

| 键         | 示例                          | 描述                                                                | 可接受值 / 格式  | 默认值   |
| ----------- | -------------------------------- | -------------------------------------------------------------------------- | ------------------------- | --------- |
| `name`      | `"my-builder"`                   | 构建器实例的名称                                               | 字符串                    | —         |
| `driver`    | `"docker-container"`             | 构建器使用的后端 (`docker`, `docker-container`, `remote`, 等) | 字符串                    | `docker`  |
| `platforms` | `["linux/amd64", "linux/arm64"]` | 构建器支持的目标平台                                  | 平台字符串数组 | 主机架构 |

- **描述：** 用于高级镜像构建场景的 Buildx 构建器实例。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 设置跨平台构建、远程构建器或自定义构建环境。
- **配置此设置的方式：**
    - **构建器**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中

> [!NOTE]
>
> 构建器定义被结构化为对象数组，每个对象描述一个构建器实例。冲突或不支持的配置可能会导致构建错误。

## AI 设置

### 启用 Docker 模型运行器

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 用于在容器中运行 AI 模型的 Docker 模型运行器功能。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 使用 Docker 基础设施运行和管理 AI/ML 模型。
- **配置此设置的方式：**
    - **AI**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enableInference` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**启用 Docker 模型运行器**设置

#### 启用主机端 TCP 支持

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** Docker 模型运行器服务的 TCP 连接。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 允许外部应用程序通过 TCP 连接到模型运行器。
- **配置此设置的方式：**
    - **AI**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enableInferenceTCP` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**主机端 TCP 支持**设置

> [!NOTE]
>
> 此设置需要先启用 Docker 模型运行器设置。

##### 端口

| 默认值 | 可接受值 | 格式  |
|---------------|-----------------|---------|
| 12434         | Integer         | 整数 |

- **描述：** 用于模型运行器 TCP 连接的特定端口。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 自定义模型运行器 TCP 连接的端口。
- **配置此设置的方式：**
    - **AI**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enableInferenceTCPPort` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**主机端 TCP 端口**设置

> [!NOTE]
>
> 此设置需要先启用 Docker 模型运行器和主机端 TCP 支持设置。

##### CORS 允许的源

| 默认值 | 可接受值                                                                 | 格式 |
|---------------|---------------------------------------------------------------------------------|--------|
| 空字符串  | 空字符串拒绝所有，`*` 接受所有，或逗号分隔的值列表 | 字符串 |

- **描述：** 模型运行器 Web 集成的跨源资源共享设置。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 允许 Web 应用程序连接到模型运行器服务。
- **配置此设置的方式：**
    - **AI**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enableInferenceCORS` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**CORS 允许的源**设置

> [!NOTE]
> 此设置需要先启用 Docker 模型运行器和主机端 TCP 支持设置。

#### 启用 GPU 支持的推理

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** GPU 支持的推理。
- **操作系统：** {{< badge color=blue text="仅限 Windows" >}}
- **使用场景：** 启用 GPU 支持的推理。其他组件将下载到 ~/.docker/bin/inference。
- **配置此设置的方式：**
    - **AI**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enableInferenceGPUVariant` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**启用 GPU 支持的推理**设置

## Kubernetes 设置

### 启用 Kubernetes

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 本地 Kubernetes 集群与 Docker Desktop 集成。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 提供本地 Kubernetes 开发环境用于测试和开发。
- **配置此设置的方式：**
    - **Kubernetes**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `kubernetes` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**允许 Kubernetes**设置

> [!NOTE]
>
> 在强化环境中，除非特别需要 Kubernetes 开发，否则禁用并锁定此设置。

> [!IMPORTANT]
>
> 当通过设置管理策略启用 Kubernetes 时，仅支持
`kubeadm` 集群配置方法。设置管理尚不支持 `kind` 配置
方法。

### 选择集群配置方法

| 默认值 | 可接受值 | 格式 |
|---------------|-----------------|--------|
| `kubeadm`     | `kubeadm`, `kind`  | 字符串 |

- **描述：** Kubernetes 集群拓扑和节点配置。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 在单节点 (`kubeadm`) 或多节点 (`kind`) 集群配置之间选择，以满足不同的开发需求。
- **配置此设置的方式：**
    - **Kubernetes**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中

### Kubernetes 节点数 (kind 配置)

| 默认值 | 可接受值 | 格式  |
|---------------|-----------------|---------|
| `1`           | Integer         | 整数 |

- **描述：** 多节点 Kubernetes 集群中的节点数。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 扩展集群大小以测试分布式应用程序或集群功能。
- **配置此设置的方式：**
    - **Kubernetes**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中

### Kubernetes 节点版本 (kind 配置)

| 默认值 | 可接受值               | 格式 |
|---------------|-------------------------------|--------|
| `1.31.1`      | Semantic version (e.g., 1.29.1) | 字符串 |

- **描述：** 用于集群节点的 Kubernetes 版本。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 固定特定的 Kubernetes 版本以保持一致性或满足兼容性要求。
- **配置此设置的方式：**
    - **Kubernetes**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中

### 显示系统容器

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** Kubernetes 系统容器在 Docker Desktop 仪表板中的可见性。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 允许开发人员查看和调试 kube-system 容器。
- **配置此设置的方式：**
    - **Kubernetes**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中

> [!NOTE]
>
> 在强化环境中，禁用并锁定此设置以减少界面复杂性。

## 软件更新设置

### 自动检查更新

| 默认值 | 可接受值 | 格式 |
|---------------|-----------------|--------|
| `false`       | `true`, `false` | 布尔值 |

- **描述：** Docker Desktop 是否检查并通知可用更新。如果
值设置为 `true`，则禁用检查更新和关于 Docker
Desktop 更新的通知。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 控制更新通知和自动版本检查。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `disableUpdate` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**禁用更新**设置

> [!NOTE]
>
> 在强化环境中，启用此设置并锁定。这保证了
仅安装经过内部审查的版本。

### 始终下载更新

| 默认值 | 可接受值 | 格式 |
|---------------|-----------------|--------|
| `false`       | `true`, `false` | 布尔值 |

- **描述：** Docker Desktop 更新可用时自动下载。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 管理带宽使用并控制何时下载更新。
- **配置此设置的方式：**
    - **软件更新**设置，位于[Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**禁用更新**设置

### 自动更新组件

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 允许 Docker Desktop 自动更新不需要重启的组件。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 自动更新关键的 Docker Desktop 组件，如 Docker Compose、Docker Scout、Docker CLI。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md#software-updates)中的**常规设置**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `silentModulesUpdate` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**自动更新组件**设置

## 扩展设置

### 启用 Docker 扩展

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 访问 Docker 扩展市场和已安装的扩展。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 控制用户是否可以安装和运行 Docker 扩展。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**扩展设置**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `extensionsEnabled` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**允许扩展**设置

> [!NOTE]
>
> 在强化环境中，禁用并锁定此设置。这可以防止
> 安装第三方或未经审查的插件。

### 仅允许通过 Docker Marketplace 分发的扩展

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 将 Docker 扩展限制为仅可通过官方市场获取的扩展。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 防止安装第三方或本地开发的扩展。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**扩展设置**

### 显示 Docker 扩展系统容器

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** Docker 扩展使用的系统容器在容器列表中的可见性。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 帮助开发者通过查看底层容器来排查扩展问题。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**扩展设置**

## Beta 功能设置

> [!IMPORTANT]
>
> 对于 Docker Desktop 4.41 及更早版本，这些设置位于**开发中功能**页面的**实验性功能**选项卡下。

### 启用 Docker AI

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** Docker AI 功能，包括"询问 Gordon"功能。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 在 Docker Desktop 中启用 AI 驱动的辅助和建议。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**Beta 设置**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enableDockerAI` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**启用 Docker AI**设置

### 启用 Docker MCP 工具包

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 在 Docker Desktop 中启用 [Docker MCP 工具包](/manuals/ai/mcp-catalog-and-toolkit/_index.md)。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 为 AI 模型开发工作流启用 MCP 工具包功能。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**Beta 设置**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enableDockerMCPToolkit` 设置

### 启用 Docker 卸载

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 在 Docker Desktop 中启用 [Docker 卸载](/offload/)。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 将构建和运行容器卸载到云端。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**Beta 设置**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enableCloud` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**启用 Docker 云**设置

### 启用 Wasm

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 启用 [Wasm](/manuals/desktop/features/wasm.md) 以运行 Wasm 工作负载。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 在 Docker 容器内运行 WebAssembly 应用程序和模块。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**Beta 设置**

## 通知设置

### 任务和进程的状态更新

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 在 Docker Desktop 中显示的一般信息性消息。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 控制操作状态消息和进程更新的可见性。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**通知设置**

### Docker 的推荐

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 在 Docker Desktop 中显示的推广内容和功能推荐。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 管理对 Docker 营销内容和功能推广的曝光。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**通知设置**

### Docker 公告

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 在 Docker Desktop 中显示的一般公告和新闻。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 控制 Docker 范围内的公告和重要更新的可见性。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**通知设置**

### Docker 调查

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 显示给用户的调查邀请和反馈请求。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 管理用户对 Docker 产品反馈和研究的参与。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**通知设置**

### Docker Scout 通知弹窗

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 来自 Docker Scout 漏洞扫描的应用程序内通知。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 控制漏洞扫描结果和安全建议的可见性。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**通知设置**

### Docker Scout 操作系统通知

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 来自 Docker Scout 的操作系统级别通知。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 通过系统通知中心接收 Scout 安全警报。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**通知设置**

## 高级设置

### 配置 Docker CLI 的安装

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `system`      | 文件路径       | 字符串   |

- **描述：** 安装 Docker CLI 二进制文件的文件系统位置。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 为合规性或工具集成要求自定义 CLI 安装位置。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**高级**设置

### 允许使用默认 Docker 套接字

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 默认情况下，增强容器隔离会阻止将 Docker Engine 套接字绑定挂载到容器中（例如，`docker run -v /var/run/docker.sock:/var/run/docker.sock ...`）。这允许您以受控方式放宽此限制。有关更多信息，请参阅 ECI 配置。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 支持 Docker-in-Docker 场景、CI 代理或 Testcontainers 等工具，同时保持增强容器隔离。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**高级**设置
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `dockerSocketMount` 设置

### 允许特权端口映射

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 允许将容器端口绑定到主机上的特权端口（1-1024）的权限。
- **操作系统：** {{< badge color=blue text="仅限 Mac" >}}
- **使用场景：** 允许容器使用标准服务端口，如 HTTP (80) 或 HTTPS (443)。
- **配置此设置的方式：**
    - [Docker Desktop GUI](/manuals/desktop/settings-and-maintenance/settings.md)中的**高级**设置

## 仅通过设置管理可用的设置

以下设置不在 Docker Desktop GUI 中显示。您只能通过管理控制台或 `admin-settings.json` 文件的设置管理来配置它们。

### 启用 Docker Cloud GPU 支持

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `true`        | `true`, `false` | 布尔值  |

- **描述：** 为 Docker Cloud 功能启用 GPU 支持。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **配置此设置的方式：**
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**启用 Docker Cloud GPU 支持**设置

### 阻止 `docker load`

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 阻止用户使用 `docker load` 命令加载本地 Docker 镜像。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 通过要求所有镜像来自注册表来强制执行镜像来源。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `blockDockerLoad` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**阻止 Docker 加载**设置

> [!NOTE]
>
> In hardened environments, enable and lock this setting. This forces all images
to come from your secure, scanned registry.

### 隐藏入门调查

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 阻止向新用户显示入门调查。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `displayedOnboarding` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**隐藏入门调查**设置

### 在 TCP 2375 上暴露 Docker API

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 在端口 2375 上通过未经身份验证的 TCP 套接字暴露 Docker API。仅推荐用于隔离和受保护的环境。
- **操作系统：** {{< badge color=blue text="仅限 Windows" >}}
- **使用场景：** 支持需要 TCP API 访问的旧版集成。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `exposeDockerAPIOnTCP2375`
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**暴露 Docker API**设置

> [!NOTE]
>
> In hardened environments, disable and lock this setting. This ensures the
Docker API is only reachable via the secure internal socket.

### 隔离环境容器代理

| 默认值 | 可接受值 | 格式      |
| ------------- | --------------- | ----------- |
| 参见示例   | 对象          | JSON 对象 |

- **描述：** 隔离环境中容器的 HTTP/HTTPS 代理配置。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 为离线或受限网络环境中的容器提供受控的网络访问。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `containersProxy` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**容器代理**部分

#### Example

```json
"containersProxy": {
  "locked": true,
  "mode": "manual",
  "http": "",
  "https": "",
  "exclude": [],
  "pac": "",
  "transparentPorts": ""
}
```

### Docker 套接字访问控制（ECI 例外）

| 默认值 | 可接受值 | 格式      |
| ------------- | --------------- | ----------- |
| -           | 对象          | JSON 对象 |

- **描述：** 当增强容器隔离处于活动状态时，允许使用 Docker 套接字的特定镜像和命令。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 支持需要 Docker 套接字访问的工具，如 Testcontainers、LocalStack 或 CI 系统，同时保持安全性。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `enhancedContainerIsolation` > `dockerSocketMount`
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**命令列表**

#### Example

```json
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
}
```

### 允许 Beta 功能

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `false`       | `true`, `false` | 布尔值  |

- **描述：** 访问处于公开 Beta 阶段的 Docker Desktop 功能。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 提供对开发中功能的早期访问，以便测试和反馈。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `allowBetaFeatures` 设置
    - 设置管理：**访问 Beta 功能**

> [!NOTE]
>
> In hardened environments, disable and lock this setting.

### Docker 守护进程选项（Linux 或 Windows）

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `{}`          | JSON 对象     | 字符串化 JSON |

- **描述：** 覆盖 Linux 或 Windows 容器中使用的 Docker 守护进程配置。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 配置高级守护进程选项，而无需修改本地配置文件。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `linuxVM.dockerDaemonOptions` 或 `windowsContainers.dockerDaemonOptions`

> [!NOTE]
>
> In hardened environments, provide a vetted JSON config and lock it so no
overrides are possible.

### VPNkit CIDR

| 默认值     | 可接受值 | 格式 |
|-------------------|-----------------|--------|
| `192.168.65.0/24` | CIDR 表示法   | 字符串 |

- **描述：** 用于 Docker Desktop 内部 VPNKit DHCP/DNS 服务的网络子网。
- **操作系统：** {{< badge color=blue text="仅限 Mac" >}}
- **使用场景：** 防止具有重叠网络子网的环境中的 IP 地址冲突。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `vpnkitCIDR` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**VPN Kit CIDR**设置

> [!NOTE]
>
> In hardened environments, lock to an approved, non-conflicting CIDR.

### 启用 Kerberos 和 NTLM 身份验证

| 默认值 | 可接受值 | 格式 |
|---------------|-----------------|--------|
| `false`       | `true`, `false` | 布尔值 |

- **描述：** 企业代理身份验证支持 Kerberos 和 NTLM 协议。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 支持需要 Kerberos 或 NTLM 身份验证的企业代理服务器。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `proxy.enableKerberosNtlm`
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**Kerberos NTLM**设置

### PAC 文件 URL

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `""`          | PAC 文件 URL    | 字符串   |

- **描述：** 指定 PAC 文件 URL。例如，`"pac": "http://proxy/proxy.pac"`。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `pac`
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**PAC 文件**设置

### 嵌入式 PAC 脚本

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `""`          | 嵌入式 PAC 脚本  | 字符串   |

- **描述：** 指定嵌入式 PAC（代理自动配置）脚本。例如，`"embeddedPac": "function FindProxyForURL(url, host) { return \"DIRECT\"; }"`。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `embeddedPac`
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**嵌入式 PAC 脚本**设置


### 自定义 Kubernetes 镜像仓库

| 默认值 | 可接受值 | 格式   |
|---------------|-----------------|----------|
| `""`          | 注册表 URL    | 字符串   |

- **描述：** 用于 Kubernetes 控制平面镜像的注册表，而不是 Docker Hub。这允许 Docker Desktop 从私有注册表或镜像而不是 Docker Hub 拉取 Kubernetes 系统镜像。此设置会覆盖镜像名称的 `[registry[:port]/][namespace]` 部分。
- **操作系统：** {{< badge color=blue text="全部" >}}
- **使用场景：** 支持隔离环境或当 Docker Hub 访问受限时。
- **配置此设置的方式：**
    - 设置管理：[`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中的 `KubernetesImagesRepository` 设置
    - 设置管理：[管理控制台](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)中的**Kubernetes 镜像仓库**设置

> [!NOTE]
>
> 镜像必须从 Docker Hub 镜像，并带有匹配的标签。所需镜像取决于集群配置方法。

> [!IMPORTANT]
>
> 将自定义镜像仓库与增强容器隔离一起使用时，请将这些镜像添加到 ECI 允许列表：`[imagesRepository]/desktop-cloud-provider-kind:*` 和 `[imagesRepository]/desktop-containerd-registry-mirror:*`。
