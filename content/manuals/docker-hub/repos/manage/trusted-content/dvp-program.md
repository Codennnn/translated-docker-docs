---
description: 了解 Docker 验证发布者（DVP）计划及其运作方式
title: Docker 验证发布者（DVP）计划
aliases:
- /docker-hub/publish/publish/
- /docker-hub/publish/customer_faq/
- /docker-hub/publish/publisher_faq/
- /docker-hub/publish/certify-images/
- /docker-hub/publish/certify-plugins-logging/
- /docker-hub/publish/trustchain/
- /docker-hub/publish/byol/
- /docker-hub/publish/publisher-center-migration/
- /docker-hub/publish/
- /docker-hub/publish/repository-logos/
- /docker-hub/dvp-program/
- /trusted-content/dvp-program/
---

[Docker 验证发布者计划](https://hub.docker.com/search?q=&image_filter=store) 提供由 Docker 验证的商业发布者发布的高质量镜像。

这些镜像帮助开发团队构建安全的软件供应链，在早期阶段将暴露于恶意内容的风险降到最低，从而在后续节省时间与成本。

属于该计划的镜像在 Docker Hub 上会带有特殊徽章，便于用户识别由 Docker 验证为高质量的商业发布者所发布的内容。

![Docker 验证发布者徽章](../../../images/verified-publisher-badge-iso.png)

Docker 验证发布者（DVP）计划为 Docker Hub 的发布者提供多项功能与权益。
根据参与等级，计划将授予以下权益：

- 存储库徽标
- 验证发布者徽章
- 在 Docker Hub 中的搜索优先级
- 洞察与分析
- 漏洞分析
- 额外的 Docker Business 席位
- 取消对开发者的速率限制
- 联合市场推广机会

### 存储库徽标

DVP 组织可以在 Docker Hub 为单个存储库上传自定义图像。
这允许你在存储库级别覆盖组织级默认徽标。

只有对该存储库拥有管理权限的用户（所有者或拥有管理员权限的团队成员）
可以更改存储库徽标。

#### 图像要求

- 徽标图像支持 JPEG 与 PNG 格式。
- 最小允许尺寸为 120×120 像素。
- 最大允许尺寸为 1000×1000 像素。
- 最大允许文件大小为 5MB。

#### 设置存储库徽标

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 前往你希望更改徽标的存储库页面。
3. 选择上传徽标按钮（以相机图标表示，{{< inline-image
   src="../../../images/upload_logo_sm.png" alt="camera icon" >}}），位于当前存储库徽标之上。
4. 在弹出的对话框中，选择要上传的 PNG 图像，将其设置为该存储库的徽标。

#### 移除徽标

选择 **Clear** 按钮（{{< inline-image src="../../../images/clear_logo_sm.png"
alt="clear button" >}}）即可移除徽标。

移除徽标后，若组织设置了组织徽标，则存储库将回退为组织徽标；否则将使用以下默认徽标。

![Default logo which is a 3D grey cube](../../../images/default_logo_sm.png)

### 验证发布者徽章

属于该计划的镜像在 Docker Hub 上会带有徽章，便于开发者识别由 Docker 验证为高质量、且内容可信的发布者。

![带有验证发布者徽章的 Docker, Inc. 组织](../../../images/verified-publisher-badge.png)

### 洞察与分析

[洞察与分析](./insights-analytics.md) 服务提供社区如何使用 Docker 镜像的使用指标，
帮助理解用户行为。

这些指标展示按标签或按摘要统计的镜像拉取次数，以及按地理位置、云服务商、客户端等维度的细分数据。

你可以选择要查看分析数据的时间范围；也可以将数据导出为摘要或原始格式。

### 漏洞分析

[Docker Scout](/scout/) 为发布到 Docker Hub 的 DVP 镜像提供自动漏洞分析。
对镜像进行扫描可确保发布内容安全，并向开发者证明他们可以信任该镜像。

你可以按存储库粒度启用该分析功能。有关使用方法，参见[基础漏洞扫描](/docker-hub/repos/manage/vulnerability-scanning/)。

### 谁有资格成为验证发布者？

任何在 Docker Hub 分发软件的独立软件供应商（ISV）都可以加入验证发布者计划。
更多信息请访问 [Docker 验证发布者计划](https://www.docker.com/partners/programs) 页面。
