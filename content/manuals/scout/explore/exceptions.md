---
title: 管理漏洞例外（Exceptions）
description: |
  通过例外（Exceptions），你可以为漏洞如何影响制品提供额外上下文与说明，
  并将不适用的漏洞标记为忽略，从而抑制其在分析中的干扰
keywords: scout, cves, suppress, vex, exceptions
---

容器镜像中发现的漏洞，有时需要补充上下文来判断风险。
仅仅因为镜像包含某个存在漏洞的软件包，并不代表该漏洞一定可被利用。
Docker Scout 的 **Exceptions（例外）** 能帮助你标记可接受的风险，
或处理镜像分析中的误报（false positive）。

通过忽略不适用的漏洞，你和镜像的下游使用者都能更容易在镜像语境中理解这些漏洞的实际安全影响。

在 Docker Scout 中，例外会自动参与分析计算。
如果镜像中存在将某个 CVE 标注为“不适用”的例外，
该 CVE 将不会出现在分析结果中。

## 创建例外

你可以通过以下方式为镜像创建例外：

- 在 Docker Scout Dashboard 或 Docker Desktop 的
  [GUI](/manuals/scout/how-tos/create-exceptions-gui.md) 中创建例外。
- 创建并将 [VEX](/manuals/scout/how-tos/create-exceptions-vex.md) 文档附加到镜像。

推荐使用 Docker Scout Dashboard 或 Docker Desktop 创建例外。
GUI 提供了友好的交互界面，且支持针对多个镜像或整个组织批量创建例外。

## 查看例外

查看镜像的例外需要相应权限。

- 通过 [GUI](/manuals/scout/how-tos/create-exceptions-gui.md) 创建的例外仅对你的 Docker 组织成员可见。
  未认证用户或非组织成员无法查看这些例外。
- 通过 [VEX 文档](/manuals/scout/how-tos/create-exceptions-vex.md) 创建的例外，
  对任何能够拉取该镜像的用户可见，因为 VEX 文档存储在镜像清单或镜像的文件系统中。

### 在 Docker Scout Dashboard 或 Docker Desktop 中查看例外

在 Docker Scout Dashboard 的 Vulnerabilities 页面中，
[**Exceptions** 选项卡](https://scout.docker.com/reports/vulnerabilities/exceptions)
会列出组织内所有镜像的全部例外。你可以在此查看每条例外的详情、被抑制的 CVE、
例外适用的镜像、例外类型及其创建方式等。

对通过 [GUI](/manuals/scout/how-tos/create-exceptions-gui.md) 创建的例外，
可通过操作菜单进行编辑或移除。

如需查看某个镜像标签的全部例外：

{{< tabs >}}
{{< tab name="Docker Scout Dashboard" >}}

1. 进入 [Images 页面](https://scout.docker.com/reports/images)。
2. 选择你要查看的标签。
3. 打开 **Exceptions** 选项卡。

{{< /tab >}}
{{< tab name="Docker Desktop" >}}

1. 在 Docker Desktop 中打开 **Images** 视图。
2. 打开 **Hub** 选项卡。
3. 选择你要查看的标签。
4. 打开 **Exceptions** 选项卡。

{{< /tab >}}
{{< /tabs >}}

### 在 CLI 中查看例外

{{< summary-bar feature_name="Docker Scout exceptions" >}}

当你运行 `docker scout cves <image>` 时，CLI 会高亮显示漏洞例外。
若某个 CVE 被例外抑制，会在该 CVE ID 旁出现 `SUPPRESSED` 标记，同时显示该例外的详细信息。

![CLI 输出中的 SUPPRESSED 标记](/scout/images/suppressed-cve-cli.png)

> [!IMPORTANT]
> 要在 CLI 中查看例外，你必须将 CLI 配置为与创建例外时相同的 Docker 组织。
>
> 为 CLI 配置组织：
>
> ```console
> $ docker scout configure organization <organization>
> ```
>
> 将 `<organization>` 替换为你的 Docker 组织名称。
>
> 你也可以在单次命令中通过 `--org` 参数指定组织：
>
> ```console
> $ docker scout cves --org <organization> <image>
> ```

如需在输出中排除已被抑制的 CVE，可使用 `--ignore-suppressed` 参数：

```console
$ docker scout cves --ignore-suppressed <image>
```
