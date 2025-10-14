---
description: 适用于 Amazon EBS 的卷插件
keywords: "API, Usage, plugins, documentation, developer, amazon, ebs, rexray, volume"
---

<!-- 本文件在 docker/cli GitHub 仓库维护：
     https://github.com/docker/cli/ 。请将所有 Pull Request 提交到该仓库。
     如果你在其他仓库看到本文件，请将其视为只读文件，因为它会被
     权威版本定期覆盖。在其他仓库中提交对此文件的修改将被拒绝。 -->

# 适用于 Amazon EBS 的卷插件

## Rexray 概念验证（PoC）插件

本示例将创建一个简易的 Rexray 插件，用于在带有 EBS 的 Amazon EC2 实例上使用。它并非完整的 Rexray 插件实现，仅供演示之用。

示例源码： [https://github.com/tiborvass/rexray-plugin](https://github.com/tiborvass/rexray-plugin)

了解更多 Rexray： [https://github.com/codedellemc/rexray](https://github.com/codedellemc/rexray)

## 1. 制作 Docker 镜像

下面是将 rexray 容器化所使用的 Dockerfile：

```dockerfile
FROM debian:jessie
RUN apt-get update && apt-get install -y --no-install-recommends wget ca-certificates
RUN wget https://dl.bintray.com/emccode/rexray/stable/0.6.4/rexray-Linux-x86_64-0.6.4.tar.gz -O rexray.tar.gz && tar -xvzf rexray.tar.gz -C /usr/bin && rm rexray.tar.gz
RUN mkdir -p /run/docker/plugins /var/lib/libstorage/volumes
ENTRYPOINT ["rexray"]
CMD ["--help"]
```

构建镜像可运行：`image=$(cat Dockerfile | docker build -q -)`，此时变量 `$image` 引用的是容器化后的 rexray 镜像。

## 2. 提取 rootfs

```sh
$ TMPDIR=/tmp/rexray  # 本示例的临时目录
$  # 创建容器但不运行它，从镜像中提取 rootfs
$ docker create --name rexray "$image"
$  # 将 rootfs 保存为 tar 包
$ docker export -o $TMPDIR/rexray.tar rexray
$  # 从 tar 包解压 rootfs 到目标目录
$ ( mkdir -p $TMPDIR/rootfs; cd $TMPDIR/rootfs; tar xf ../rexray.tar )
```

## 3. 添加插件配置

将以下 JSON 写入 `$TMPDIR/config.json`：

```json
{
      "Args": {
        "Description": "",
        "Name": "",
        "Settable": null,
        "Value": null
      },
      "Description": "A proof-of-concept EBS plugin (using rexray) for Docker",
      "Documentation": "https://github.com/tiborvass/rexray-plugin",
      "Entrypoint": [
        "/usr/bin/rexray", "service", "start", "-f"
      ],
      "Env": [
        {
          "Description": "",
          "Name": "REXRAY_SERVICE",
          "Settable": [
            "value"
          ],
          "Value": "ebs"
        },
        {
          "Description": "",
          "Name": "EBS_ACCESSKEY",
          "Settable": [
            "value"
          ],
          "Value": ""
        },
        {
          "Description": "",
          "Name": "EBS_SECRETKEY",
          "Settable": [
            "value"
          ],
          "Value": ""
        }
      ],
      "Interface": {
        "Socket": "rexray.sock",
        "Types": [
          "docker.volumedriver/1.0"
        ]
      },
      "Linux": {
        "AllowAllDevices": true,
        "Capabilities": ["CAP_SYS_ADMIN"],
        "Devices": null
      },
      "Mounts": [
        {
          "Source": "/dev",
          "Destination": "/dev",
          "Type": "bind",
          "Options": ["rbind"]
        }
      ],
      "Network": {
        "Type": "host"
      },
      "PropagatedMount": "/var/lib/libstorage/volumes",
      "User": {},
      "WorkDir": ""
}
```

注意事项：
- 需要 `PropagatedMount`，以便 Docker 守护进程能看到插件在容器内执行的挂载操作；否则守护进程无法挂载 Docker 卷。
- Rexray 插件需要对宿主机设备的动态访问。因此需要授予其对 `/dev` 下所有设备的访问权限，并将 `AllowAllDevices` 设为 true 才能正常工作。
- 该简化插件的用户仅能更改 3 个设置：`REXRAY_SERVICE`、`EBS_ACCESSKEY` 与 `EBS_SECRETKEY`。这是因为示例范围有限。理想情况下，还可开放更多 Rexray 参数。

## 4. 创建插件

运行 `docker plugin create tiborvass/rexray-plugin "$TMPDIR"` 创建插件。

```sh
$ docker plugin ls
ID                  NAME                             DESCRIPTION                         ENABLED
2475a4bd0ca5        tiborvass/rexray-plugin:latest   A rexray volume plugin for Docker   false
```

## 5. 测试插件

```sh
$ docker plugin set tiborvass/rexray-plugin EBS_ACCESSKEY=$AWS_ACCESSKEY EBS_SECRETKEY=$AWS_SECRETKEY
$ docker plugin enable tiborvass/rexray-plugin
$ docker volume create -d tiborvass/rexray-plugin my-ebs-volume
$ docker volume ls
DRIVER                              VOLUME NAME
tiborvass/rexray-plugin:latest      my-ebs-volume
$ docker run --rm -v my-ebs-volume:/volume busybox sh -c 'echo bye > /volume/hi'
$ docker run --rm -v my-ebs-volume:/volume busybox cat /volume/hi
bye
```

## 6. 推送插件

首先确保你已通过 `docker login` 登录。然后使用 `docker plugin push tiborvass/rexray-plugin` 将其像常规 Docker 镜像一样推送到仓库，便于他人通过如下方式安装：
`docker plugin install tiborvass/rexray-plugin EBS_ACCESSKEY=$AWS_ACCESSKEY EBS_SECRETKEY=$AWS_SECRETKEY`。
