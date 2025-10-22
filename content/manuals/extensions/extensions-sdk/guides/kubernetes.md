---
title: 从扩展与 Kubernetes 交互
linkTitle: 与 Kubernetes 交互
description: 如何从扩展连接到 Kubernetes 集群
keywords: Docker, 扩展, sdk, Kubernetes
aliases:
 - /desktop/extensions-sdk/dev/kubernetes/
 - /desktop/extensions-sdk/guides/kubernetes/
---

扩展 SDK 不提供任何直接与 Docker Desktop 管理的 Kubernetes 集群或其他使用 KinD 等工具创建的集群交互的 API 方法。但是，本页面为您提供了使用其他 SDK API 从扩展中间接与 Kubernetes 集群交互的方法。

如果您希望请求直接与 Docker Desktop 管理的 Kubernetes 交互的 API，可以在 Extensions SDK GitHub 仓库中为[此问题](https://github.com/docker/extensions-sdk/issues/181)投票。

## 先决条件

### 启用 Kubernetes

您可以使用 Docker Desktop 内置的 Kubernetes 来启动 Kubernetes 单节点集群。
`kubeconfig` 文件用于配置对 Kubernetes 的访问权限，通常与 `kubectl` 命令行工具或其他客户端一起使用。
Docker Desktop 会在用户的主目录区域方便地提供预配置的 `kubeconfig` 文件和 `kubectl` 命令。对于希望通过 Docker Desktop 利用 Kubernetes 的用户来说，这是一种快速访问的便捷方式。

## 将 `kubectl` 作为扩展的一部分

如果您的扩展需要与 Kubernetes 集群交互，建议您将 `kubectl` 命令行工具包含在扩展中。这样做后，安装您的扩展的用户将在其主机上安装 `kubectl`。

要了解如何将多平台的 `kubectl` 命令行工具作为 Docker 扩展镜像的一部分进行分发，请参阅[构建多架构扩展](../extensions/multi-arch.md#adding-multi-arch-binaries)。

## 示例

以下代码片段来自 [Kubernetes 示例扩展](https://github.com/docker/extensions-sdk/tree/main/samples/kubernetes-sample-extension)。它展示了如何通过分发 `kubectl` 命令行工具与 Kubernetes 集群交互。

### 检查 Kubernetes API 服务器是否可达

当 `kubectl` 命令行工具在 `Dockerfile` 中添加到扩展镜像中，并在 `metadata.json` 中定义后，扩展框架会在安装扩展时将 `kubectl` 部署到用户的主机上。

您可以使用 JS API `ddClient.extension.host?.cli.exec` 来发出 `kubectl` 命令，例如检查给定上下文下的 Kubernetes API 服务器是否可达：

```typescript
const output = await ddClient.extension.host?.cli.exec("kubectl", [
  "cluster-info",
  "--request-timeout",
  "2s",
  "--context",
  "docker-desktop",
]);
```

### 列出 Kubernetes 上下文

```typescript
const output = await ddClient.extension.host?.cli.exec("kubectl", [
  "config",
  "view",
  "-o",
  "jsonpath='{.contexts}'",
]);
```

### 列出 Kubernetes 命名空间

```typescript
const output = await ddClient.extension.host?.cli.exec("kubectl", [
  "get",
  "namespaces",
  "--no-headers",
  "-o",
  'custom-columns=":metadata.name"',
  "--context",
  "docker-desktop",
]);
```

## 持久化 kubeconfig 文件

以下是几种从主机文件系统持久化和读取 `kubeconfig` 文件的不同方法。用户可以随时向 `kubeconfig` 文件添加、编辑或删除 Kubernetes 上下文。

> 警告
>
> `kubeconfig` 文件非常敏感，如果被发现可能会让攻击者获得 Kubernetes 集群的管理访问权限。

### 扩展的后端容器

如果您需要扩展在读取 `kubeconfig` 文件后将其持久化，可以让后端容器暴露一个 HTTP POST 端点，将文件内容存储在内存或容器文件系统中的某个位置。这样，当用户从扩展导航到 Docker Desktop 的其他部分然后再回来时，您就不需要再次读取 `kubeconfig` 文件。

```typescript
export const updateKubeconfig = async () => {
  const kubeConfig = await ddClient.extension.host?.cli.exec("kubectl", [
    "config",
    "view",
    "--raw",
    "--minify",
    "--context",
    "docker-desktop",
  ]);
  if (kubeConfig?.stderr) {
    console.log("error", kubeConfig?.stderr);
    return false;
  }

  // 调用后端容器将检索到的 kubeconfig 存储到容器的内存或文件系统中
  try {
    await ddClient.extension.vm?.service?.post("/store-kube-config", {
      data: kubeConfig?.stdout,
    });
  } catch (err) {
    console.log("error", JSON.stringify(err));
  }
};
```

### Docker 卷

卷是持久化 Docker 容器生成和使用的数据的首选机制。您可以利用它们来持久化 `kubeconfig` 文件。
通过在卷中持久化 `kubeconfig`，当扩展面板关闭时您无需再次读取 `kubeconfig` 文件。这使得在导航到 Docker Desktop 的其他部分时持久化数据成为理想选择。

```typescript
const kubeConfig = await ddClient.extension.host?.cli.exec("kubectl", [
  "config",
  "view",
  "--raw",
  "--minify",
  "--context",
  "docker-desktop",
]);
if (kubeConfig?.stderr) {
  console.log("error", kubeConfig?.stderr);
  return false;
}

await ddClient.docker.cli.exec("run", [
  "--rm",
  "-v",
  "my-vol:/tmp",
  "alpine",
  "/bin/sh",
  "-c",
  `"touch /tmp/.kube/config && echo '${kubeConfig?.stdout}' > /tmp/.kube/config"`,
]);
```

### 扩展的 `localStorage`

`localStorage` 是浏览器 Web 存储的机制之一。它允许用户在浏览器中以键值对的形式保存数据供以后使用。
当浏览器（扩展面板）关闭时，`localStorage` 不会清除数据。这使得在导航到 Docker Desktop 的其他部分时持久化数据成为理想选择。

```typescript
localStorage.setItem("kubeconfig", kubeConfig);
```

```typescript
localStorage.getItem("kubeconfig");
```
