---
title: 测试和调试
description: 测试和调试您的扩展。
keywords: Docker, 扩展, sdk, 预览, 更新, Chrome DevTools
aliases:
 - /desktop/extensions-sdk/build/test-debug/
 - /desktop/extensions-sdk/dev/test-debug/
weight: 10
---

为了改善开发者体验，Docker Desktop 提供了一套工具来帮助您测试和调试扩展。

### 打开 Chrome DevTools

要在您选择**扩展**标签页时为扩展打开 Chrome DevTools，请运行：

```console
$ docker extension dev debug <您的扩展名称>
```

之后每次点击扩展标签页都会打开 Chrome DevTools。要停止此行为，请运行：

```console
$ docker extension dev reset <您的扩展名称>
```

扩展部署后，还可以通过[科乐美秘技](https://en.wikipedia.org/wiki/Konami_Code)的变体从 UI 扩展部分打开 Chrome DevTools。选择**扩展**标签页，然后按下按键序列 `up, up, down, down, left, right, left, right, p, d, t`。

### UI 开发期间的热重载

在 UI 开发期间，使用热重载来测试更改而无需重新构建整个扩展会很有帮助。为此，您可以配置 Docker Desktop 从开发服务器加载 UI，例如 [Vite](https://vitejs.dev/) 在使用 `npm start` 调用时启动的服务器。

假设您的应用程序在默认端口上运行，启动您的 UI 应用程序然后运行：

```console
$ cd ui
$ npm run dev
```

这将启动一个监听 3000 端口的开发服务器。

现在您可以告诉 Docker Desktop 使用此地址作为前端源。在另一个终端中运行：

```console
$ docker extension dev ui-source <您的扩展名称> http://localhost:3000
```

关闭并重新打开 Docker Desktop 仪表板，然后转到您的扩展。前端代码的所有更改都会立即可见。

完成后，您可以将扩展配置重置为原始设置。如果您使用了 `docker extension dev debug <您的扩展名称>`，这也将重置打开 Chrome DevTools 的行为：

```console
$ docker extension dev reset <您的扩展名称>
```

## 显示扩展容器

如果您的扩展由一个或多个在 Docker Desktop VM 中作为容器运行的服务组成，您可以轻松地从 Docker Desktop 仪表板访问它们。

1. 在 Docker Desktop 中，导航到**设置**。
2. 在**扩展**标签页下，选择**显示 Docker Desktop 扩展系统容器**选项。现在您可以查看扩展容器及其日志。

## 清理

要删除扩展，请运行：

```console
$ docker extension rm <您的扩展名称>
```

## 下一步

- 构建[高级前端](/manuals/extensions/extensions-sdk/build/frontend-extension-tutorial.md)扩展。
- 了解更多关于扩展的[架构](../architecture/_index.md)。
- 探索我们的[设计原则](../design/design-principles.md)。
- 查看我们的[UI 样式指南](../design/_index.md)。
- 学习如何为您的扩展[设置 CI](continuous-integration.md)。
