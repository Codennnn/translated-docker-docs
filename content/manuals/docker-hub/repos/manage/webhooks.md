---
description: Docker Hub Webhooks
keywords: Docker, webhooks, hub, builds
title: Webhooks（Webhook 回调）
weight: 80
aliases:
- /docker-hub/webhooks/
---

你可以使用 Webhook 在存储库发生推送（push）事件时，通知并触发其他服务的动作。
Webhook 会向你在 Docker Hub 中配置的目标 URL 发送 POST 请求。

## 创建 Webhook

创建 Webhook：
1. 在目标存储库中，进入 **Webhooks** 选项卡。
2. 输入 Webhook 的名称。
3. 填写目标 Webhook URL（POST 请求将发送至此地址）。
4. 点击 **Create**。

## 查看 Webhook 投递历史

查看某个 Webhook 的历史记录：
1. 在 **Current Webhooks** 区域，鼠标悬停于目标 Webhook。
2. 点击 **Menu options** 图标。
3. 选择 **View History**。

随后你可以查看投递历史，以及每次 POST 请求是否成功。

## Webhook 负载示例

Webhook 的负载（payload）通常为如下 JSON 格式：

```json
{
  "callback_url": "https://registry.hub.docker.com/u/svendowideit/testhook/hook/2141b5bi5i5b02bec211i4eeih0242eg11000a/",
  "push_data": {
    "pushed_at": 1417566161,
    "pusher": "trustedbuilder",
    "tag": "latest"
  },
  "repository": {
    "comment_count": 0,
    "date_created": 1417494799,
    "description": "",
    "dockerfile": "#\n# BUILD\u0009\u0009docker build -t svendowideit/apt-cacher .\n# RUN\u0009\u0009docker run -d -p 3142:3142 -name apt-cacher-run apt-cacher\n#\n# and then you can run containers with:\n# \u0009\u0009docker run -t -i -rm -e http_proxy http://192.168.1.2:3142/ debian bash\n#\nFROM\u0009\u0009ubuntu\n\n\nVOLUME\u0009\u0009[/var/cache/apt-cacher-ng]\nRUN\u0009\u0009apt-get update ; apt-get install -yq apt-cacher-ng\n\nEXPOSE \u0009\u00093142\nCMD\u0009\u0009chmod 777 /var/cache/apt-cacher-ng ; /etc/init.d/apt-cacher-ng start ; tail -f /var/log/apt-cacher-ng/*\n",
    "full_description": "Docker Hub based automated build from a GitHub repo",
    "is_official": false,
    "is_private": true,
    "is_trusted": true,
    "name": "testhook",
    "namespace": "svendowideit",
    "owner": "svendowideit",
    "repo_name": "svendowideit/testhook",
    "repo_url": "https://registry.hub.docker.com/u/svendowideit/testhook/",
    "star_count": 0,
    "status": "Active"
  }
}
```
