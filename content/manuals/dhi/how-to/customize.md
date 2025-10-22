---
title: 自定义 Docker 加固镜像
linkTitle: 自定义镜像
weight: 25
keywords: 调试, 加固镜像, DHI, 自定义, 证书, 工件
description: 学习如何自定义 Docker 加固镜像（DHI）。
---

您可以使用 Docker Hub 界面自定义 Docker 加固镜像（DHI）以满足您的特定需求。这允许您选择基础镜像、添加软件包、添加工件和配置设置。此外，构建流水线确保您的自定义镜像安全构建并包含证明。

要将自定义的 Docker 加固镜像添加到您的组织，组织所有者必须首先将 DHI 仓库[镜像](./mirror.md)到您的组织。一旦仓库被镜像，任何有权限访问镜像 DHI 仓库的用户都可以创建自定义镜像。

## 自定义 Docker 加固镜像

要自定义 Docker 加固镜像，请按照以下步骤操作：

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **我的 Hub**。
3. 在命名空间下拉菜单中，选择具有镜像 DHI 仓库的组织。
4. 选择 **加固镜像** > **管理**。
5. 对于您想要自定义的镜像 DHI 仓库，选择最右侧列的菜单图标。
6. 选择 **自定义**。

   此时，屏幕上的说明将指导您完成自定义过程。您可以继续以下步骤以获取更多详细信息。

7. 选择您想要自定义的镜像版本。
8. 添加软件包。

   1. 在 **软件包** 下拉菜单中，选择您想要添加到镜像的软件包。

      下拉菜单中可用的软件包是所选镜像变体的操作系统系统软件包。例如，如果您正在自定义 Python DHI 的 Alpine 变体，列表将包含所有 Alpine 系统软件包。

   2. 在 **OCI 工件** 下拉菜单中，首先选择包含 OCI 工件镜像的仓库。然后，从该仓库中选择您想要使用的标签。最后，指定您想要从 OCI 工件镜像中包含的特定路径。

      OCI 工件是您之前构建并推送到与镜像 DHI 相同命名空间中的仓库的镜像。例如，您可以添加自定义根 CA 证书或包含您需要的工具的另一个镜像，比如向 Node.js 镜像中添加 Python。有关如何创建 OCI 工件镜像的更多详细信息，请参阅[创建 OCI 工件镜像](#创建-oci-工件镜像)。

      当组合包含相同路径的目录和文件的镜像时，列表中后面的镜像将覆盖前面镜像的文件。为了管理这一点，您必须为每个 OCI 工件镜像选择要包含的路径，并可选择排除路径。这允许您控制哪些文件包含在最终的自定义镜像中。

      默认情况下，不会从 OCI 工件镜像中包含任何文件。您必须明确包含您想要的路径。包含路径后，您可以明确排除其下的文件或目录。

      > [!注意]
      >
      > 当 OCI 工件覆盖运行所需的文件时，镜像构建仍然会成功，但您在运行镜像时可能会遇到问题。

9. 选择 **下一步：配置**，然后配置以下选项。

   1. 指定附加到自定义镜像标签的后缀。例如，如果您在自定义 `dhi-python:3.13` 镜像时指定 `custom`，自定义镜像将被标记为 `dhi-python:3.13_custom`。
   2. 选择您想要构建镜像的平台。
   3. 向镜像添加 [`ENTRYPOINT`](/reference/dockerfile/#entrypoint) 和 [`CMD`](/reference/dockerfile/#cmd) 参数。这些参数将附加到基础镜像的入口点和命令。
   4. 指定要添加到镜像的用户。
   5. 指定要添加到镜像的用户组。
   6. 选择以哪个[用户](/reference/dockerfile/#user)运行镜像。
   7. 指定镜像将包含的[环境变量](/reference/dockerfile/#env)及其值。
   8. 向镜像添加[注解](/build/metadata/annotations/)。
   9. 向镜像添加[标签](/reference/dockerfile/#label)。
10. 选择 **创建自定义**。

    自定义摘要出现。镜像构建可能需要一些时间。构建完成后，它将出现在仓库的 **标签** 选项卡中，您的团队成员可以像拉取任何其他镜像一样拉取它。

## 编辑或删除 Docker 加固镜像自定义

要编辑或删除 Docker 加固镜像自定义，请按照以下步骤操作：

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **我的 Hub**。
3. 在命名空间下拉菜单中，选择具有镜像 DHI 的组织。
4. 选择 **加固镜像** > **管理**。
5. 选择 **自定义**。

6. 对于您想要管理的自定义 DHI 仓库，选择最右侧列的菜单图标。
   从这里，您可以：

   - **编辑**：编辑自定义镜像。
   - **新建**：基于源仓库创建新的自定义镜像。
   - **删除**：删除自定义镜像。

7. 按照屏幕上的说明完成编辑或删除。

## 创建 OCI 工件镜像

OCI 工件镜像是包含您想要包含在自定义 Docker 加固镜像（DHI）中的文件或目录的 Docker 镜像。这可以包括额外的工具、库或配置文件。

在创建用作 OCI 工件的镜像时，它应该尽可能最小化，并且只包含必要的文件。

例如，要分发作为受信任 CA 捆绑包一部分的自定义根 CA 证书，您可以使用多阶段构建。这种方法向系统注册您的证书并输出更新的 CA 捆绑包，可以提取到最小化的最终镜像中：

```dockerfile
# syntax=docker/dockerfile:1

FROM <your-namespace>/dhi-bash:5-dev AS certs

ENV DEBIAN_FRONTEND=noninteractive

RUN mkdir -p /usr/local/share/ca-certificates/my-rootca
COPY certs/rootCA.crt /usr/local/share/ca-certificates/my-rootca

RUN update-ca-certificates

FROM scratch
COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
```

您可以遵循此模式创建其他 OCI 工件，比如包含您想要包含在自定义 DHI 中的工具或库的镜像。在第一阶段安装必要的工具或库，然后将相关文件复制到使用 `FROM scratch` 的最终阶段。这确保您的 OCI 工件最小化并且只包含必要的文件。

构建并推送 OCI 工件镜像到您组织命名空间中的仓库，当您选择要添加到自定义 Docker 加固镜像的 OCI 工件时，它会自动出现在自定义工作流中。
