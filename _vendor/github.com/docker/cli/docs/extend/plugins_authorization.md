---
title: 访问授权插件
description: "如何创建授权插件以管理对 Docker 守护进程的访问控制。"
keywords: "security, authorization, authentication, docker, documentation, plugin, extend"
aliases:
- "/engine/extend/authorization/"
---

本文介绍如何在 Docker Engine 中使用访问授权（Authorization）插件。若需了解由 Docker Engine 托管的插件，请参阅《[Docker Engine 插件系统](_index.md)》。

Docker 的开箱即用授权模型是“全有或全无”。任何有权访问 Docker 守护进程的用户，都可以运行任意 Docker 客户端命令；通过 Engine API 访问守护进程的调用方也同样如此。如果你需要更细粒度的访问控制，可以编写授权插件，并把它们加入 Docker 守护进程的配置中。借助授权插件，Docker 管理员可以为守护进程配置更细粒度的访问策略。

具备相应技能的任何人都可以开发授权插件，这些基础技能包括：对 Docker 的了解、对 REST 的理解，以及扎实的编程能力。本文将说明授权插件开发所需的体系结构、状态与方法信息。

## 基本原理

Docker 的[插件基础设施](plugin_api.md)通过通用 API 加载、移除并与第三方组件通信，从而扩展 Docker 功能。访问授权子系统正是基于该机制构建。

借助该子系统，你无需重建 Docker 守护进程即可添加授权插件。你可以在已安装的 Docker 守护进程上新增插件，但添加新插件后需要重启守护进程。

授权插件会基于“当前认证上下文”与“命令上下文”来决定是否批准或拒绝对守护进程的请求。认证上下文包含所有用户详情与认证方法；命令上下文则包含与请求相关的所有数据。

授权插件必须遵循《[Docker 插件 API](plugin_api.md)》中的规则。每个插件都必须位于[插件发现](plugin_api.md#plugin-discovery)一节所述的目录中。

> [!NOTE]
> 缩写 `AuthZ` 与 `AuthN` 分别表示授权（authorization）与认证（authentication）。

## 默认用户授权机制

如果在[Docker 守护进程](https://docs.docker.com/engine/security/https/)中启用了 TLS，默认的用户授权流程会从证书的主题名称中提取用户信息。也就是说，`User` 字段会被设置为客户端证书的 Common Name，而 `AuthenticationMethod` 字段会被设置为 `TLS`。

## 基本架构

你需要在 Docker 守护进程启动流程中注册你的插件。可以安装多个插件并将它们串联起来，且该链路可以定义顺序。每个到达守护进程的请求都会按顺序经过整条链路。只有当所有插件都授权访问时，请求才会被允许。

当通过 CLI 或 Engine API 向 Docker 守护进程发起 HTTP 请求时，认证子系统会将该请求转发给已安装的认证插件。请求中包含用户（调用方）与命令上下文。插件负责决定是允许还是拒绝该请求。

下图分别展示允许与拒绝两种授权流程：

![Authorization Allow flow](images/authz_allow.png)

![Authorization Deny flow](images/authz_deny.png)

发送给插件的每个请求都包含已认证用户、HTTP 头与请求/响应体。插件只会接收到用户名与所用认证方法，特别地，不会传递任何用户凭据或令牌。并且，并非所有请求/响应体都会发送给授权插件，只有当 `Content-Type` 为 `text/*` 或 `application/json` 时才会发送。

对于可能“劫持”HTTP 连接（`HTTP Upgrade`）的命令（如 `exec`），授权插件只会在初始 HTTP 请求阶段被调用；一旦插件批准该命令，后续数据流不再进行授权检查，流式数据也不会传递给授权插件。对于返回分块 HTTP 响应的命令（如 `logs` 与 `events`），仅 HTTP 请求会发送给授权插件。

在处理请求/响应期间，某些授权流程可能需要对 Docker 守护进程发起额外查询。为完成此类流程，插件可以像普通用户那样调用守护进程 API。为启用这些额外查询，插件必须为管理员提供必要的配置手段，以设置合适的认证与安全策略。

## Docker 客户端流程

要启用并配置授权插件，插件开发者必须支持本节所述的 Docker 客户端交互方式。

### 配置 Docker 守护进程

通过专用命令行参数启用授权插件，格式为 `--authorization-plugin=PLUGIN_ID`。其中 `PLUGIN_ID` 可以是插件的套接字或规范文件的路径。授权插件支持在无需重启守护进程的情况下加载。更多信息参见[`dockerd` 文档](https://docs.docker.com/reference/cli/dockerd/#configuration-reload-behavior)。

```console
$ dockerd --authorization-plugin=plugin1 --authorization-plugin=plugin2,...
```

Docker 的授权子系统支持传入多个 `--authorization-plugin` 参数。

### 调用被允许的命令（allow）

```console
$ docker pull centos
<...>
f1b10cd84249: Pull complete
<...>
```

### 调用被拒绝的命令（deny）

```console
$ docker pull centos
<...>
docker: Error response from daemon: authorization denied by plugin PLUGIN_NAME: volumes are not allowed.
```

### 插件返回的错误

```console
$ docker pull centos
<...>
docker: Error response from daemon: plugin PLUGIN_NAME failed with error: AuthZPlugin.AuthZReq: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
```

## API 架构与实现

除了 Docker 标准的插件注册方式之外，每个授权插件还应实现以下两个方法：

- `/AuthZPlugin.AuthZReq`：在 Docker 守护进程处理客户端请求之前调用，用于请求授权。
- `/AuthZPlugin.AuthZRes`：在 Docker 守护进程将响应返回给客户端之前调用，用于响应授权。

#### /AuthZPlugin.AuthZReq

请求：

```json
{
    "User":              "The user identification",
    "UserAuthNMethod":   "The authentication method used",
    "RequestMethod":     "The HTTP method",
    "RequestURI":        "The HTTP request URI",
    "RequestBody":       "Byte array containing the raw HTTP request body",
    "RequestHeader":     "Byte array containing the raw HTTP request header as a map[string][]string "
}
```

响应：

```json
{
    "Allow": "Determined whether the user is allowed or not",
    "Msg":   "The authorization message",
    "Err":   "The error message if things go wrong"
}
```

#### /AuthZPlugin.AuthZRes

请求：

```json
{
    "User":              "The user identification",
    "UserAuthNMethod":   "The authentication method used",
    "RequestMethod":     "The HTTP method",
    "RequestURI":        "The HTTP request URI",
    "RequestBody":       "Byte array containing the raw HTTP request body",
    "RequestHeader":     "Byte array containing the raw HTTP request header as a map[string][]string",
    "ResponseBody":      "Byte array containing the raw HTTP response body",
    "ResponseHeader":    "Byte array containing the raw HTTP response header as a map[string][]string",
    "ResponseStatusCode":"Response status code"
}
```

响应：

```json
{
   "Allow":              "Determined whether the user is allowed or not",
   "Msg":                "The authorization message",
   "Err":                "The error message if things go wrong"
}
```

### 请求授权

每个插件都必须支持两类请求授权消息格式：一类是守护进程发往插件；另一类是插件返回给守护进程。如下表列出了各字段及其含义。

#### 守护进程 -> 插件

Name                   | Type              | Description
-----------------------|-------------------|-------------------------------------------------------
User                   | string            | The user identification
Authentication method  | string            | The authentication method used
Request method         | enum              | The HTTP method (GET/DELETE/POST)
Request URI            | string            | The HTTP request URI including API version (e.g., v.1.17/containers/json)
Request headers        | map[string]string | Request headers as key value pairs (without the authorization header)
Request body           | []byte            | Raw request body

#### 插件 -> 守护进程

Name    | Type   | Description
--------|--------|----------------------------------------------------------------------------------
Allow   | bool   | Boolean value indicating whether the request is allowed or denied
Msg     | string | Authorization message (will be returned to the client in case the access is denied)
Err     | string | Error message (will be returned to the client in case the plugin encounter an error. The string value supplied may appear in logs, so should not include confidential information)

### 响应授权

插件同样必须支持两类响应授权消息格式：一类是守护进程发往插件；另一类是插件返回给守护进程。如下表列出了各字段及其含义。

#### 守护进程 -> 插件

Name                    | Type              | Description
----------------------- |------------------ |----------------------------------------------------
User                    | string            | The user identification
Authentication method   | string            | The authentication method used
Request method          | string            | The HTTP method (GET/DELETE/POST)
Request URI             | string            | The HTTP request URI including API version (e.g., v.1.17/containers/json)
Request headers         | map[string]string | Request headers as key value pairs (without the authorization header)
Request body            | []byte            | Raw request body
Response status code    | int               | Status code from the Docker daemon
Response headers        | map[string]string | Response headers as key value pairs
Response body           | []byte            | Raw Docker daemon response body

#### 插件 -> 守护进程

Name    | Type   | Description
--------|--------|----------------------------------------------------------------------------------
Allow   | bool   | Boolean value indicating whether the response is allowed or denied
Msg     | string | Authorization message (will be returned to the client in case the access is denied)
Err     | string | Error message (will be returned to the client in case the plugin encounter an error. The string value supplied may appear in logs, so should not include confidential information)
