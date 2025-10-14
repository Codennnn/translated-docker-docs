---
description: 了解在 Docker 仪表板的 Volumes 视图中可以执行的操作
keywords: Docker Desktop 仪表板, 管理, 容器, 图形界面, 仪表板, 卷, 用户手册
title: 浏览 Docker Desktop 的 Volumes 视图
linkTitle: 卷（Volumes）
weight: 30
---

Docker Desktop 的 **Volumes** 视图可用于创建、检查、删除、克隆、清空、导出与导入 [Docker 卷](/manuals/engine/storage/volumes.md)。你还可以浏览卷中的文件与文件夹，并查看哪些容器正在使用它们。

## 查看你的卷

你可以查看以下卷信息：

- Name：卷名。
- Status：卷当前是否被容器使用。
- Created：卷创建至今的时间。
- Size：卷大小。
- Scheduled exports：是否启用了计划导出。

默认情况下，**Volumes** 视图会显示所有卷的列表。

你可以通过以下方式筛选与排序卷，并自定义显示的列：

- 按名称筛选：使用 **Search**（搜索）字段。
- 按状态筛选：在搜索栏右侧按 **In use** 或 **Unused** 过滤。
- 排序：选择某列表头进行排序。
- 自定义列：在搜索栏右侧选择需要展示的卷信息。

## 创建卷

使用以下步骤可创建一个空卷。或者，当你[以一个尚不存在的卷启动容器](/manuals/engine/storage/volumes.md#start-a-container-with-a-volume)时，Docker 会自动为你创建该卷。

创建卷：

1. 在 **Volumes** 视图中选择 **Create** 按钮。
2. 在 **New Volume** 对话框中指定卷名，然后选择 **Create**。

要在容器中使用该卷，参见[使用卷](/manuals/engine/storage/volumes.md#start-a-container-with-a-volume)。

## 检查卷

要查看某个卷的详细信息，请在列表中选择该卷，即可打开详情视图。

**Container in-use** 选项卡会显示正在使用该卷的容器名称、镜像名、容器使用的端口号以及 target。target 指的是容器内访问卷中文件的路径。

**Stored data** 选项卡会显示卷中的文件、文件夹及其大小。要保存某个文件或文件夹，可在其上右键打开菜单，选择 **Save as...** 并指定下载位置。

要从卷中删除文件或文件夹，可在其上右键打开菜单，选择 **Delete**，并再次选择 **Delete** 以确认。

**Exports** 选项卡支持你[导出该卷](#export-a-volume)。

## 克隆卷

克隆卷会创建一个新卷，并复制被克隆卷中的全部数据。当被克隆卷正被一个或多个容器使用时，Docker 会在克隆期间暂时停止这些容器，并在克隆完成后重新启动。

克隆卷：

1. 登录 Docker Desktop。克隆卷需要先登录。
2. 在 **Volumes** 视图中，在目标卷的 **Actions** 列选择 **Clone** 图标。
3. 在 **Clone a volume** 对话框中指定 **Volume name**，然后选择 **Clone**。

## 删除一个或多个卷

删除卷会一并删除其中所有数据。当某个卷正被容器使用时，即使该容器已停止，你也无法删除此卷。你需要先停止并移除使用该卷的所有容器，才能删除该卷。

删除卷：

1. 在 **Volumes** 视图中，在目标卷的 **Actions** 列选择 **Delete** 图标。
2. 在 **Delete volume?** 对话框中选择 **Delete forever**。

删除多个卷：

1. 在 **Volumes** 视图中，勾选所有需要删除的卷。
2. 选择 **Delete**。
3. 在 **Delete volumes?** 对话框中选择 **Delete forever**。

## 清空卷

清空卷会删除其中所有数据，但不会删除该卷本身。当清空被一个或多个容器使用的卷时，Docker 会在清空期间暂时停止这些容器，并在清空完成后重新启动。

清空卷：

1. 登录 Docker Desktop。清空卷需要先登录。
2. 在 **Volumes** 视图中选择需要清空的卷。
3. 在 **Import** 旁选择 **More volume actions** 图标，然后选择 **Empty volume**。
4. 在 **Empty a volume?** 对话框中选择 **Empty**。

## 导出卷

你可以将卷的内容导出到本地文件、本地镜像、Docker Hub 上的镜像，或受支持的云服务提供商。当导出被一个或多个容器使用的卷内容时，Docker 会在导出期间暂时停止这些容器，并在导出完成后重新启动。

你可以[立即导出](#export-a-volume-now)或[计划定期导出](#schedule-a-volume-export)。

### 立即导出卷

1. 登录 Docker Desktop。导出卷需要先登录。
2. 在 **Volumes** 视图中选择需要导出的卷。
3. 选择 **Exports** 选项卡。
4. 选择 **Quick export**。
5. 选择导出到 **Local or Hub storage** 或 **External cloud storage**，并根据选择填写以下信息。

   {{< tabs >}}
   {{< tab name="Local or Hub storage" >}}
   
   - **Local file**：指定文件名并选择文件夹。
   - **Local image**：选择一个本地镜像用于接收导出内容。镜像中的现有数据将被导出内容替换。
   - **New image**：为新镜像指定名称。
   - **Registry**：指定一个 Docker Hub 仓库。

   {{< /tab >}}
   {{< tab name="External cloud storage" >}}

   导出到外部云提供商需要 [Docker Business 订阅](../../subscription/details.md)。

   选择你的云提供商，并指定用于上传到存储的 URL。参考下列文档以获取 URL：

   - Amazon Web Services：[使用 AWS SDK 创建 Amazon S3 预签名 URL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example_s3_Scenario_PresignedUrl_section.html)
   - Microsoft Azure：[生成 SAS 令牌与 URL](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/connection-strings/generate-sas-token)
   - Google Cloud：[创建用于上传对象的签名 URL](https://cloud.google.com/storage/docs/access-control/signing-urls-with-helpers#upload-object)

   {{< /tab >}}
   {{< /tabs >}}

6. 选择 **Save**。

### 计划导出卷

1. 登录 Docker Desktop。计划导出需要先登录并具备付费的 [Docker 订阅](../../subscription/details.md)。
2. 在 **Volumes** 视图中选择需要导出的卷。
3. 选择 **Exports** 选项卡。
4. 选择 **Schedule export**。
5. 在 **Recurrence** 中选择导出的频率，并根据选择填写以下信息。

   - **Daily**：指定每天备份的时间。
   - **Weekly**：指定每周的某一天或多天，以及备份时间。
   - **Monthly**：指定每月的哪一天与备份时间。

6. 选择导出到 **Local or Hub storage** 或 **External cloud storage**，并根据选择填写以下信息。
   
   {{< tabs >}}
   {{< tab name="Local or Hub storage" >}}
   
   - **Local file**：指定文件名并选择文件夹。
   - **Local image**：选择一个本地镜像用于接收导出内容。镜像中的现有数据将被导出内容替换。
   - **New image**：为新镜像指定名称。
   - **Registry**：指定一个 Docker Hub 仓库。

   {{< /tab >}}
   {{< tab name="External cloud storage" >}}

   导出到外部云提供商需要 [Docker Business 订阅](../../subscription/details.md)。

   选择你的云提供商，并指定用于上传到存储的 URL。参考下列文档以获取 URL：

   - Amazon Web Services：[使用 AWS SDK 创建 Amazon S3 预签名 URL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example_s3_Scenario_PresignedUrl_section.html)
   - Microsoft Azure：[生成 SAS 令牌与 URL](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/connection-strings/generate-sas-token)
   - Google Cloud：[创建用于上传对象的签名 URL](https://cloud.google.com/storage/docs/access-control/signing-urls-with-helpers#upload-object)

   {{< /tab >}}
   {{< /tabs >}}

7. 选择 **Save**。

## 导入卷

你可以导入本地文件、本地镜像或来自 Docker Hub 的镜像。导入后，卷中任何现有数据都会被新内容替换。当向被一个或多个容器使用的卷导入内容时，Docker 会在导入期间暂时停止这些容器，并在导入完成后重新启动。

导入卷：

1. 登录 Docker Desktop。导入卷需要先登录。
2. 可选：先[创建](#create-a-volume) 一个新卷用于导入内容。
3. 选择需要导入内容的卷。
4. 选择 **Import**。
5. 选择内容来源，并根据选择填写以下信息：

   - **Local file**：选择包含内容的本地文件。
   - **Local image**：选择包含内容的本地镜像。
   - **Registry**：指定包含内容的 Docker Hub 镜像。

6. 选择 **Import**。

## 其他资源

- [持久化容器数据](/get-started/docker-concepts/running-containers/persisting-container-data.md)
- [使用卷](/manuals/engine/storage/volumes.md)
