---
title: 持续集成（CI）
description: 自动测试和验证您的扩展。
keywords: Docker, 扩展, sdk, CI, 测试, 回归测试
aliases: 
 - /desktop/extensions-sdk/dev/continuous-integration/
weight: 20
---

为了帮助验证您的扩展并确保其功能正常，扩展 SDK 提供了工具来帮助您为扩展设置持续集成。

> [!IMPORTANT]
>
> [Docker Desktop Action](https://github.com/docker/desktop-action) 和 [extension-test-helper 库](https://www.npmjs.com/package/@docker/extension-test-helper)目前都处于[实验阶段](https://docs.docker.com/release-lifecycle/#experimental)。

## 使用 GitHub Actions 设置 CI 环境

您需要 Docker Desktop 才能安装和验证您的扩展。
您可以使用 [Docker Desktop Action](https://github.com/docker/desktop-action) 在 GitHub Actions 中启动 Docker Desktop，只需在工作流文件中添加以下内容：

```yaml
steps:
  - id: start_desktop
    uses: docker/desktop-action/start@v0.1.0
```

> [!NOTE]
>
> 此操作目前仅支持 GitHub Actions 的 macOS 运行器。您需要为端到端测试指定 `runs-on: macOS-latest`。

步骤执行完成后，后续步骤将使用 Docker Desktop 和 Docker CLI 来安装和测试扩展。

## 使用 Puppeteer 验证您的扩展

Docker Desktop 在 CI 中启动后，您可以使用 Jest 和 Puppeteer 来构建、安装和验证您的扩展。

首先，在测试中构建并安装扩展：

```ts
import { DesktopUI } from "@docker/extension-test-helper";
import { exec as originalExec } from "child_process";
import * as util from "util";

export const exec = util.promisify(originalExec);

// 保留应用句柄以便在测试结束时停止
let dashboard: DesktopUI;

beforeAll(async () => {
  await exec(`docker build -t my/extension:latest .`, {
    cwd: "my-extension-src-root",
  });

  await exec(`docker extension install -f my/extension:latest`);
});
```

然后打开 Docker Desktop Dashboard 并在扩展的用户界面中运行一些测试：

```ts
describe("测试我的扩展", () => {
  test("应该功能正常", async () => {
    dashboard = await DesktopUI.start();

    const eFrame = await dashboard.navigateToExtension("my/extension");

    // 使用 puppeteer API 操作界面，点击按钮，期望视觉显示并验证您的扩展
    await eFrame.waitForSelector("#someElementId");
  });
});
```

最后，关闭 Docker Desktop Dashboard 并卸载您的扩展：

```ts
afterAll(async () => {
  dashboard?.stop();
  await exec(`docker extension uninstall my/extension`);
});
```

## 下一步

- 构建[高级前端](/manuals/extensions/extensions-sdk/build/frontend-extension-tutorial.md)扩展。
- 了解更多关于扩展的[架构](../architecture/_index.md)。
- 学习如何[发布您的扩展](../extensions/_index.md)。
