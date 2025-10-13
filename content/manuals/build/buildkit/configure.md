---
title: 配置 BuildKit
description: 学习如何为你的 builder 配置 BuildKit。
keywords: build, buildkit, configuration, buildx, network, cni, registry
---

如果你使用 Buildx 创建 `docker-container` 或 `kubernetes` builder，
可以通过在 `docker buildx create` 命令中传入
[`--buildkitd-config` 标志](/reference/cli/docker/buildx/create.md#buildkitd-config)，
应用自定义的[BuildKit 配置](toml-configuration.md)。

## 仓库镜像（Registry mirror）

你可以为构建配置一个仓库镜像地址。这样，BuildKit 会从不同的主机名拉取镜像。
以下步骤示例演示将 `docker.io`（Docker Hub）设置为 `mirror.gcr.io` 的镜像：

1. 在 `/etc/buildkitd.toml` 创建 TOML 文件，内容如下：

   ```toml
   debug = true
   [registry."docker.io"]
     mirrors = ["mirror.gcr.io"]
   ```

   > [!NOTE]
   >
   > `debug = true` 会在 BuildKit 守护进程中开启调试请求，
   > 从而在日志中记录何时使用了镜像站点。

2. 创建一个使用该 BuildKit 配置的 `docker-container` builder：

   ```console
   $ docker buildx create --use --bootstrap \
     --name mybuilder \
     --driver docker-container \
     --buildkitd-config /etc/buildkitd.toml
   ```

3. 构建一个镜像：

   ```bash
   docker buildx build --load . -f - <<EOF
   FROM alpine
   RUN echo "hello world"
   EOF
   ```

现在，该 builder 的 BuildKit 日志会表明它使用了 GCR 镜像站。
你可以从响应消息中包含的 `x-goog-*` HTTP 头部看出来。

```console
$ docker logs buildx_buildkit_mybuilder0
```

```text
...
time="2022-02-06T17:47:48Z" level=debug msg="do request" request.header.accept="application/vnd.docker.container.image.v1+json, */*" request.header.user-agent=containerd/1.5.8+unknown request.method=GET spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg="fetch response received" response.header.accept-ranges=bytes response.header.age=1356 response.header.alt-svc="h3=\":443\"; ma=2592000,h3-29=\":443\"; ma=2592000,h3-Q050=\":443\"; ma=2592000,h3-Q046=\":443\"; ma=2592000,h3-Q043=\":443\"; ma=2592000,quic=\":443\"; ma=2592000; v=\"46,43\"" response.header.cache-control="public, max-age=3600" response.header.content-length=1469 response.header.content-type=application/octet-stream response.header.date="Sun, 06 Feb 2022 17:25:17 GMT" response.header.etag="\"774380abda8f4eae9a149e5d5d3efc83\"" response.header.expires="Sun, 06 Feb 2022 18:25:17 GMT" response.header.last-modified="Wed, 24 Nov 2021 21:07:57 GMT" response.header.server=UploadServer response.header.x-goog-generation=1637788077652182 response.header.x-goog-hash="crc32c=V3DSrg==" response.header.x-goog-hash.1="md5=d0OAq9qPTq6aFJ5dXT78gw==" response.header.x-goog-metageneration=1 response.header.x-goog-storage-class=STANDARD response.header.x-goog-stored-content-encoding=identity response.header.x-goog-stored-content-length=1469 response.header.x-guploader-uploadid=ADPycduqQipVAXc3tzXmTzKQ2gTT6CV736B2J628smtD1iDytEyiYCgvvdD8zz9BT1J1sASUq9pW_ctUyC4B-v2jvhIxnZTlKg response.status="200 OK" spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg="fetch response received" response.header.accept-ranges=bytes response.header.age=760 response.header.alt-svc="h3=\":443\"; ma=2592000,h3-29=\":443\"; ma=2592000,h3-Q050=\":443\"; ma=2592000,h3-Q046=\":443\"; ma=2592000,h3-Q043=\":443\"; ma=2592000,quic=\":443\"; ma=2592000; v=\"46,43\"" response.header.cache-control="public, max-age=3600" response.header.content-length=1471 response.header.content-type=application/octet-stream response.header.date="Sun, 06 Feb 2022 17:35:13 GMT" response.header.etag="\"35d688bd15327daafcdb4d4395e616a8\"" response.header.expires="Sun, 06 Feb 2022 18:35:13 GMT" response.header.last-modified="Wed, 24 Nov 2021 21:07:12 GMT" response.header.server=UploadServer response.header.x-goog-generation=1637788032100793 response.header.x-goog-hash="crc32c=aWgRjA==" response.header.x-goog-hash.1="md5=NdaIvRUyfar8201DleYWqA==" response.header.x-goog-metageneration=1 response.header.x-goog-storage-class=STANDARD response.header.x-goog-stored-content-encoding=identity response.header.x-goog-stored-content-length=1471 response.header.x-guploader-uploadid=ADPycdtR-gJYwC7yHquIkJWFFG8FovDySvtmRnZBqlO3yVDanBXh_VqKYt400yhuf0XbQ3ZMB9IZV2vlcyHezn_Pu3a1SMMtiw response.status="200 OK" spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg=fetch spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg=fetch spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg=fetch spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg=fetch spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg="do request" request.header.accept="application/vnd.docker.image.rootfs.diff.tar.gzip, */*" request.header.user-agent=containerd/1.5.8+unknown request.method=GET spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg="fetch response received" response.header.accept-ranges=bytes response.header.age=1356 response.header.alt-svc="h3=\":443\"; ma=2592000,h3-29=\":443\"; ma=2592000,h3-Q050=\":443\"; ma=2592000,h3-Q046=\":443\"; ma=2592000,h3-Q043=\":443\"; ma=2592000,quic=\":443\"; ma=2592000; v=\"46,43\"" response.header.cache-control="public, max-age=3600" response.header.content-length=2818413 response.header.content-type=application/octet-stream response.header.date="Sun, 06 Feb 2022 17:25:17 GMT" response.header.etag="\"1d55e7be5a77c4a908ad11bc33ebea1c\"" response.header.expires="Sun, 06 Feb 2022 18:25:17 GMT" response.header.last-modified="Wed, 24 Nov 2021 21:07:06 GMT" response.header.server=UploadServer response.header.x-goog-generation=1637788026431708 response.header.x-goog-hash="crc32c=ZojF+g==" response.header.x-goog-hash.1="md5=HVXnvlp3xKkIrRG8M+vqHA==" response.header.x-goog-metageneration=1 response.header.x-goog-storage-class=STANDARD response.header.x-goog-stored-content-encoding=identity response.header.x-goog-stored-content-length=2818413 response.header.x-guploader-uploadid=ADPycdsebqxiTBJqZ0bv9zBigjFxgQydD2ESZSkKchpE0ILlN9Ibko3C5r4fJTJ4UR9ddp-UBd-2v_4eRpZ8Yo2llW_j4k8WhQ response.status="200 OK" spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
...
```

## 设置仓库证书

如果你在 BuildKit 配置中指定了仓库证书，守护进程会将这些文件复制到容器内的 `/etc/buildkit/certs` 目录。
下面的步骤展示如何将自签名仓库证书添加到 BuildKit 配置。

1. 在 `/etc/buildkitd.toml` 添加如下配置：

   ```toml
   # /etc/buildkitd.toml
   debug = true
   [registry."myregistry.com"]
     ca=["/etc/certs/myregistry.pem"]
     [[registry."myregistry.com".keypair]]
       key="/etc/certs/myregistry_key.pem"
       cert="/etc/certs/myregistry_cert.pem"
   ```

   这会告知 builder：向 `myregistry.com` 仓库推送镜像时，使用指定位置（`/etc/certs`）下的证书。

2. 创建一个使用该配置的 `docker-container` builder：

   ```console
   $ docker buildx create --use --bootstrap \
     --name mybuilder \
     --driver docker-container \
     --buildkitd-config /etc/buildkitd.toml
   ```

3. 查看 builder 的配置文件（`/etc/buildkit/buildkitd.toml`），你会看到证书配置已被写入：

   ```console
   $ docker exec -it buildx_buildkit_mybuilder0 cat /etc/buildkit/buildkitd.toml
   ```

   ```toml
   debug = true

   [registry]

     [registry."myregistry.com"]
       ca = ["/etc/buildkit/certs/myregistry.com/myregistry.pem"]

       [[registry."myregistry.com".keypair]]
         cert = "/etc/buildkit/certs/myregistry.com/myregistry_cert.pem"
         key = "/etc/buildkit/certs/myregistry.com/myregistry_key.pem"
   ```

4. 验证证书已存在于容器中：

   ```console
   $ docker exec -it buildx_buildkit_mybuilder0 ls /etc/buildkit/certs/myregistry.com/
   myregistry.pem    myregistry_cert.pem   myregistry_key.pem
   ```

现在，你就可以使用该 builder 推送到仓库了，认证将使用这些证书：

```console
$ docker buildx build --push --tag myregistry.com/myimage:latest .
```

## CNI 网络

在并发构建时，builder 使用 CNI 网络有助于应对网络端口的竞争。
CNI 目前[尚未](https://github.com/moby/buildkit/issues/28)包含在默认的 BuildKit 镜像中，
但你可以自行构建包含 CNI 支持的镜像。

下面的 Dockerfile 示例展示了一个包含 CNI 支持的自定义 BuildKit 镜像。
它使用 BuildKit 的[集成测试 CNI 配置](https://github.com/moby/buildkit/blob/master//hack/fixtures/cni.json)作为示例；
你也可以替换为自己的 CNI 配置。

```dockerfile
# syntax=docker/dockerfile:1

ARG BUILDKIT_VERSION=v{{% param "buildkit_version" %}}
ARG CNI_VERSION=v1.0.1

FROM --platform=$BUILDPLATFORM alpine AS cni-plugins
RUN apk add --no-cache curl
ARG CNI_VERSION
ARG TARGETOS
ARG TARGETARCH
WORKDIR /opt/cni/bin
RUN curl -Ls https://github.com/containernetworking/plugins/releases/download/$CNI_VERSION/cni-plugins-$TARGETOS-$TARGETARCH-$CNI_VERSION.tgz | tar xzv

FROM moby/buildkit:${BUILDKIT_VERSION}
ARG BUILDKIT_VERSION
RUN apk add --no-cache iptables
COPY --from=cni-plugins /opt/cni/bin /opt/cni/bin
ADD https://raw.githubusercontent.com/moby/buildkit/${BUILDKIT_VERSION}/hack/fixtures/cni.json /etc/buildkit/cni.json
```

现在你可以构建该镜像，并通过
[`--driver-opt image` 选项](/reference/cli/docker/buildx/create.md#driver-opt)
基于它创建一个 builder 实例：

```console
$ docker buildx build --tag buildkit-cni:local --load .
$ docker buildx create --use --bootstrap \
  --name mybuilder \
  --driver docker-container \
  --driver-opt "image=buildkit-cni:local" \
  --buildkitd-flags "--oci-worker-net=cni"
```

## 资源限制

### 最大并行数（Max parallelism）

你可以限制 BuildKit 求解器的并行度，这对低性能机器尤为有用。可在创建 builder 时，
通过[BuildKit 配置](toml-configuration.md)与
[`--buildkitd-config` 标志](/reference/cli/docker/buildx/create.md#buildkitd-config)实现：

```toml
# /etc/buildkitd.toml
[worker.oci]
  max-parallelism = 4
```

现在你可以[创建一个 `docker-container` builder](/manuals/build/builders/drivers/docker-container.md)
它会使用该 BuildKit 配置来限制并行度：

```console
$ docker buildx create --use \
  --name mybuilder \
  --driver docker-container \
  --buildkitd-config /etc/buildkitd.toml
```

### TCP 连接数限制

每个仓库在拉取与推送镜像时的 TCP 连接数被限制为最多 4 个并发连接，
另有 1 个专用连接用于处理元数据请求。该限制可防止你的构建在拉取镜像时陷入阻塞，
而专用的元数据连接有助于降低整体构建时间。

更多信息： [moby/buildkit#2259](https://github.com/moby/buildkit/pull/2259)
