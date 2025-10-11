---
title: 在 Docker 中使用 CA 证书
linkTitle: CA 证书
description: 了解如何在 Docker 主机与 Linux 容器中安装与使用 CA 证书
keywords: docker, networking, ca, certs, host, container, proxy
---

> [!CAUTION]
> 在生产环境容器中使用中间人（MITM）CA 证书时务必遵循最佳实践。一旦证书被攻破，攻击者可能拦截敏感数据、伪造受信任服务或发动中间人攻击。在继续之前，请咨询你的安全团队。

如果你的公司使用会拦截/检查 HTTPS 流量的代理，你可能需要将相应的根证书添加到主机与 Docker 容器/镜像中。因为 Docker 及其容器在拉取镜像或发起网络请求时，需要信任该代理的证书。

在主机上添加根证书，可确保 Docker 命令（如 `docker pull`）正常运行。对于容器，你需要在构建阶段或运行时将根证书加入容器的信任存储，从而保证容器内的应用可以通过代理通信，而不会出现安全告警或连接失败。

## 向主机添加 CA 证书

以下内容描述如何在 macOS 或 Windows 主机上安装 CA 证书。Linux 环境请参考各发行版文档。

### macOS

1. 下载 MITM 代理软件的 CA 证书。
2. 打开“钥匙串访问”（Keychain Access）。
3. 选择 **System**，切换到 **Certificates** 选项卡。
4. 将下载的证书拖拽到证书列表中（若提示请输入密码）。
5. 找到新添加的证书，双击并展开 **Trust** 区域。
6. 将其设置为 **Always Trust**（总是信任），如有提示输入密码。
7. 启动 Docker Desktop，验证 `docker pull` 可用（前提是 Docker Desktop 已配置使用该 MITM 代理）。

### Windows

你可以选择使用 Microsoft 管理控制台（MMC）或 Web 浏览器安装证书。

{{< tabs >}}
{{< tab name="MMC" >}}

1. 下载 MITM 代理的 CA 证书。
2. 打开 Microsoft 管理控制台（`mmc.exe`）。
3. 在 MMC 中添加“证书”管理单元（Certificates Snap-In）。
   1. 选择 **File** → **Add/Remove Snap-in**，选择 **Certificates** → **Add >**。
   2. 选择 **Computer Account** → **Next**。
   3. 选择 **Local computer** → **Finish**。
4. 导入 CA 证书：
   1. 在 MMC 中展开 **Certificates (Local Computer)**。
   2. 展开 **Trusted Root Certification Authorities**。
   3. 右键 **Certificates**，选择 **All Tasks** → **Import…**。
   4. 按向导导入 CA 证书。
5. 选择 **Finish**，然后 **Close**。
6. 启动 Docker Desktop，验证 `docker pull` 成功（前提是 Docker Desktop 已配置为使用 MITM 代理）。

> [!NOTE]
> Depending on the SDK and/or runtime/framework in use, further steps may be
> required beyond adding the CA certificate to the operating system's trust
> store.

{{< /tab >}}
{{< tab name="Web browser" >}}

1. 下载 MITM 代理的 CA 证书。
2. 打开浏览器进入 **Settings**，打开 **Manage certificates**。
3. 选择 **Trusted Root Certification Authorities** 选项卡。
4. 选择 **Import**，并选择已下载的 CA 证书。
5. 选择 **Open**，然后勾选 **Place all certificates in the following store**。
6. 确认选择 **Trusted Root Certification Authorities**，点击 **Next**。
7. 选择 **Finish**，然后 **Close**。
8. 启动 Docker Desktop，验证 `docker pull` 成功（前提是 Docker Desktop 已配置使用 MITM 代理）。

{{< /tab >}}
{{< /tabs >}}

## 向 Linux 镜像与容器添加 CA 证书

如果你在需要内部或自定义证书的环境（如公司代理或安全服务）中运行容器化工作负载，必须确保容器信任这些证书。否则，容器内应用在访问 HTTPS 端点时可能出现请求失败或安全告警。

在构建阶段[将 CA 证书加入镜像](#add-certificates-to-images)，可确保从该镜像启动的任意容器都信任这些证书。这对需要无缝访问内部 API、数据库或其他服务的生产应用尤为重要。

若无法重建镜像，也可以直接[向运行中的容器添加证书](#add-certificates-to-containers)。但运行时添加的证书在容器销毁或重建后不会保留，因此更适用于临时修复或测试场景。

## 向镜像添加证书

> [!NOTE]
> The following commands are for an Ubuntu base image. If your build uses a
> different Linux distribution, use equivalent commands for package management
> (`apt-get`, `update-ca-certificates`, and so on).

要在构建镜像时加入 CA 证书，可在 Dockerfile 中添加如下指令：

```dockerfile
# 安装 CA 证书软件包
RUN apt-get update && apt-get install -y ca-certificates
# 将上下文中的 CA 证书拷贝到构建容器
COPY your_certificate.crt /usr/local/share/ca-certificates/
# 更新容器内的 CA 证书
RUN update-ca-certificates
```

### 向容器添加证书

> [!NOTE]
> The following commands are for an Ubuntu-based container. If your container
> uses a different Linux distribution, use equivalent commands for package
> management (`apt-get`, `update-ca-certificates`, and so on).

向一个正在运行的 Linux 容器添加 CA 证书：

1. 下载 MITM 代理的 CA 证书。
2. 如果证书不是 `.crt` 格式，将其转换为 `.crt`：

   ```console {title="Example command"}
   $ openssl x509 -in cacert.der -inform DER -out myca.crt
   ```

3. 将证书拷贝到正在运行的容器：

    ```console
    $ docker cp myca.crt <containerid>:/tmp
    ```

4. 进入容器：

    ```console
    $ docker exec -it <containerid> sh
    ```

5. 确认证书更新所需的软件包 `ca-certificates` 已安装：

    ```console
    # apt-get update && apt-get install -y ca-certificates
    ```

6. 将证书拷贝到 CA 证书目录：

    ```console
    # cp /tmp/myca.crt /usr/local/share/ca-certificates/root_cert.crt
    ```

7. 更新 CA 证书：

    ```console
    # update-ca-certificates
    ```

    ```plaintext {title="Example output"}
    Updating certificates in /etc/ssl/certs...
    rehash: warning: skipping ca-certificates.crt, it does not contain exactly one certificate or CRL
    1 added, 0 removed; done.
    ```

8. 验证容器可以通过 MITM 代理进行通信：

    ```console
    # curl https://example.com
    ```

    ```plaintext {title="Example output"}
    <!doctype html>
    <html>
    <head>
        <title>Example Domain</title>
    ...
    ```
