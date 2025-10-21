---
title: 隔离容器
description: 使用自定义代理规则和网络限制，通过隔离容器控制容器网络访问
keywords: 隔离容器, 网络安全, 代理配置, 容器隔离, docker desktop
aliases:
 - /desktop/hardened-desktop/settings-management/air-gapped-containers/
 - /desktop/hardened-desktop/air-gapped-containers/
 - /security/for-admins/hardened-desktop/air-gapped-containers/
---

{{< summary-bar feature_name="Air-gapped containers" >}}

隔离容器允许您通过控制容器发送和接收数据的位置来限制容器网络访问。此功能将自定义代理规则应用于容器网络流量，有助于保护容器不应具有不受限制的互联网访问的环境。

Docker Desktop 可以配置容器网络流量以接受连接、拒绝连接或通过 HTTP 或 SOCKS 代理进行隧道传输。您可以控制策略应用于哪些 TCP 端口，以及是通过代理自动配置 (PAC) 文件使用单个代理还是按目标策略。

本页面提供隔离容器的概述和配置步骤。

## 谁应该使用隔离容器？

隔离容器帮助组织在受限环境中维护安全性：

- **安全开发环境**：防止容器访问未经授权的外部服务
- **合规要求**：满足需要网络隔离的法规标准
- **数据丢失防护**：阻止容器将敏感数据上传到外部服务
- **供应链安全**：控制容器在构建过程中可以访问哪些外部资源
- **企业网络策略**：对容器化应用程序执行现有的网络安全策略

## 隔离容器的工作原理

隔离容器通过拦截容器网络流量并应用代理规则来运行：

1. **流量拦截**：Docker Desktop 拦截来自容器的所有传出网络连接
1. **端口过滤**：只有指定端口（`transparentPorts`）上的流量受代理规则约束
1. **规则评估**：PAC 文件规则或静态代理设置确定如何处理每个连接
1. **连接处理**：根据规则直接允许流量、通过代理路由或阻止流量

一些重要注意事项包括：

- 现有的 `proxy` 设置继续应用于主机上的 Docker Desktop 应用程序流量
- 如果 PAC 文件下载失败，容器会阻止对目标 URL 的请求
- URL 参数格式为 `http://host_or_ip:port` 或 `https://host_or_ip:port`
- 端口 80 和 443 可用主机名，但其他端口只能使用 IP 地址

## 先决条件

在配置隔离容器之前，您必须具备：

- 已启用[强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)以确保用户通过您的组织进行身份验证
- Docker Business 订阅
- 已配置[设置管理](/manuals/enterprise/security/hardened-desktop/settings-management/_index.md)以管理组织策略
- 已下载 Docker Desktop 4.29 或更高版本

## 配置隔离容器

将容器代理添加到您的 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md)中。例如：

```json
{
  "configurationFileVersion": 2,
  "containersProxy": {
    "locked": true,
    "mode": "manual",
    "http": "",
    "https": "",
    "exclude": [],
    "pac": "http://192.168.1.16:62039/proxy.pac",
    "transparentPorts": "*"
  }
}
```

### 配置参数

`containersProxy` 设置控制应用于容器流量的网络策略：

| 参数 | 描述 | 值 |
|-----------|-------------|-------|
| `locked` | 防止开发者覆盖设置 | `true` (锁定), `false` (默认) |
| `mode` | 代理配置方法 | `system` (使用系统代理), `manual` (自定义) |
| `http` | HTTP 代理服务器 | URL (例如, `"http://proxy.company.com:8080"`) |
| `https` | HTTPS 代理服务器 | URL (例如, `"https://proxy.company.com:8080"`) |
| `exclude` | 绕过代理的地址 | 主机名/IP 数组 |
| `pac` | 代理自动配置文件 URL | PAC 文件的 URL |
| `transparentPorts` | 受代理规则约束的端口 | 逗号分隔的端口或通配符 (`"*"`) |

### 配置示例

阻止所有外部访问：

```json
"containersProxy": {
  "locked": true,
  "mode": "manual",
  "http": "",
  "https": "",
  "exclude": [],
  "transparentPorts": "*"
}
```

允许特定的内部服务：

```json
"containersProxy": {
  "locked": true,
  "mode": "manual",
  "http": "",
  "https": "",
  "exclude": ["internal.company.com", "10.0.0.0/8"],
  "transparentPorts": "80,443"
}
```

通过企业代理路由：

```json
"containersProxy": {
  "locked": true,
  "mode": "manual",
  "http": "http://corporate-proxy.company.com:8080",
  "https": "http://corporate-proxy.company.com:8080",
  "exclude": ["localhost", "*.company.local"],
  "transparentPorts": "*"
}
```

## 代理自动配置 (PAC) 文件

PAC 文件通过为不同目标定义规则，提供对容器网络访问的精细控制。

### 基本 PAC 文件结构

```javascript
function FindProxyForURL(url, host) {
	if (localHostOrDomainIs(host, 'internal.corp')) {
		return "PROXY 10.0.0.1:3128";
	}
	if (isInNet(host, "192.168.0.0", "255.255.255.0")) {
	    return "DIRECT";
	}
    return "PROXY reject.docker.internal:1234";
}
```

### PAC 文件返回值

| 返回值 | 操作 |
|--------------|--------|
| `PROXY host:port` | 通过指定主机和端口的 HTTP 代理路由 |
| `SOCKS5 host:port` | 通过指定主机和端口的 SOCKS5 代理路由 |
| `DIRECT` | 允许直接连接而不通过代理 |
| `PROXY reject.docker.internal:any_port` | 完全阻止请求 |

### 高级 PAC 文件示例

```javascript
function FindProxyForURL(url, host) {
  // 允许访问 Docker Hub 以获取批准的基础镜像
  if (dnsDomainIs(host, ".docker.io") || host === "docker.io") {
    return "PROXY corporate-proxy.company.com:8080";
  }

  // 允许内部包仓库
  if (localHostOrDomainIs(host, 'nexus.company.com') ||
      localHostOrDomainIs(host, 'artifactory.company.com')) {
    return "DIRECT";
  }

  // 允许特定端口上的开发工具
  if (url.indexOf(":3000") > 0 || url.indexOf(":8080") > 0) {
    if (isInNet(host, "10.0.0.0", "255.0.0.0")) {
      return "DIRECT";
    }
  }

  // 阻止访问开发者的本地主机
  if (host === "host.docker.internal" || host === "localhost") {
    return "PROXY reject.docker.internal:1234";
  }

  // 阻止所有其他外部访问
  return "PROXY reject.docker.internal:1234";
}
```

## 验证隔离容器配置

应用配置后，测试容器网络限制是否正常工作：

测试被阻止的访问：

```console
$ docker run --rm alpine wget -O- https://www.google.com
# 应根据您的代理规则失败或超时
```

测试允许的访问：

```console
$ docker run --rm alpine wget -O- https://internal.company.com
# 如果 internal.company.com 在您的排除列表或 PAC 规则中，应该成功
```

测试代理路由：

```console
$ docker run --rm alpine wget -O- https://docker.io
# 如果通过批准的代理路由，应该成功
```

## 安全注意事项

- **网络策略执行**：隔离容器在 Docker Desktop 级别工作。高级用户可能通过各种方式绕过限制，因此对于高安全性环境，请考虑额外的网络级别控制。
- **开发工作流影响**：过于严格的策略可能会破坏合法的开发工作流。彻底测试并为必要的服务提供明确的例外。
- **PAC 文件管理**：将 PAC 文件托管在可靠的内部基础设施上。失败的 PAC 下载会导致容器网络访问被阻止。
- **性能考虑**：具有许多规则的复杂 PAC 文件可能会影响容器网络性能。保持规则简单高效。

