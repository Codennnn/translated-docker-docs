---
title: 使用 Docker Secrets 管理敏感数据
description: 如何在 Docker 服务中安全地存储、获取与使用敏感数据
keywords: swarm, secrets, credentials, sensitive strings, sensitive data, security,
  encryption, encryption at rest
tags: [Secrets]
---

## 关于 Secrets

在 Docker Swarm 服务的语境中，secret（机密）是一段数据，例如密码、SSH 私钥、SSL 证书，或其他不应在网络上明文传输、也不应以明文形式存放在 Dockerfile 或应用源代码中的数据。你可以使用 Docker Secrets 对这些数据进行集中管理，并仅将其安全地传递给需要访问的容器。在 Docker Swarm 中，secret 在传输与静态存储时均会被加密。只有被显式授予访问权限的服务，且仅在其任务运行期间，才能访问相应的 secret。

当容器在运行时需要某些敏感数据，但你又不希望将其保存到镜像或代码仓库时，可以使用 secrets。典型场景包括：

- 用户名与密码
- TLS 证书与密钥
- SSH 密钥
- 其他重要信息，例如数据库名称或内部服务器地址
- 一般字符串或二进制内容（最大 500 KB）

> [!NOTE]
>
> Docker Secrets 仅适用于 Swarm 服务，不适用于独立容器。要使用该功能，可以考虑将容器改为以“服务”的方式运行。对于有状态的容器，通常可以在不修改容器代码的情况下，以副本数为 1 的形式运行。

另一个常见用法是通过 secrets 在容器与凭据之间引入一层抽象。设想你的应用有开发、测试、生产三个环境：每个环境可以在各自的 Swarm 中保存不同的凭据，但都使用同一个 secret 名称。这样你的容器只需要知道 secret 的名字即可在三个环境中正常工作。

你也可以使用 secrets 来管理非敏感数据（例如配置文件）。不过，Docker 提供了更合适的机制：[configs](configs.md) 用于存放非敏感数据。Configs 会直接挂载到容器文件系统中，而不是通过内存盘（RAM disk）。

### Windows 支持

Docker 在 Windows 容器中同样支持 secrets。实现上存在差异的地方会在下文示例中标注。请注意以下差异：

- Microsoft Windows 没有内置的 RAM 磁盘驱动，因此在运行中的 Windows 容器内，secrets 会以明文形式存储在容器的根磁盘上。不过，当容器停止时，这些 secrets 会被显式删除。此外，Windows 不支持使用 `docker commit` 或类似命令将正在运行的容器持久化为镜像。

- 在 Windows 上，建议在宿主机中启用包含 Docker 根目录卷的 [BitLocker](https://technet.microsoft.com/en-us/library/cc732774(v=ws.11).aspx)，以确保运行中容器的 secrets 在静态存储时被加密。

- 由于 Windows 不支持对非目录文件进行绑定挂载，自定义目标路径的 secret 文件不会被直接绑定挂载到 Windows 容器中。相反，容器的所有 secrets 会统一挂载到容器内的 `C:\ProgramData\Docker\internal\secrets`（这是实现细节，应用不应依赖）。随后通过符号链接指向容器内期望的目标路径。默认目标路径为 `C:\ProgramData\Docker\secrets`。

- 在创建使用 Windows 容器的服务时，secrets 不支持设置 UID、GID 与文件权限（mode）。目前在容器内，只有管理员与具备 `system` 权限的用户可以访问 secrets。

## Docker 如何管理 Secrets

当你向 Swarm 添加一个 secret 时，Docker 会通过双向 TLS 连接将其发送给管理节点。该 secret 会被写入加密的 Raft 日志。整个 Raft 日志会在所有管理节点间复制，因此 secrets 享有与其他 Swarm 管理数据相同的高可用保障。

当你为新建或正在运行的服务授予某个 secret 的访问权限时，解密后的 secret 会以内存文件系统的形式挂载到容器中。默认挂载位置为 Linux 容器的 `/run/secrets/<secret_name>`，或 Windows 容器的 `C:\ProgramData\Docker\secrets`。你也可以指定自定义位置。

你可以随时更新某个服务，以授予其访问更多 secrets，或撤销其对某个 secret 的访问权限。

仅当节点是 Swarm 管理节点，或其正在运行的服务任务已被授予访问权限时，该节点才可访问（加密的）secrets。当容器任务停止运行时，挂载给它的解密后的 secrets 会从内存文件系统中卸载，并从该节点内存中清除。

如果节点在运行拥有 secret 访问权限的任务容器时与 Swarm 失联，该任务容器仍可访问其已挂载的 secrets，但在节点重新连回 Swarm 之前无法接收更新。

你可以随时添加或检查单个 secret，或列出全部 secrets。但无法删除正在被运行中服务使用的 secret。可参考「[轮换 secret](secrets.md#example-rotate-a-secret)」在不影响正在运行的服务的情况下移除旧 secret。

为方便更新或回滚 secrets，可在 secret 名称中加入版本号或日期。同时，你也可以控制 secret 在容器内的挂载路径，这有助于平滑切换。

## 进一步了解 `docker secret` 命令

你可以通过以下链接查看具体命令，也可以继续阅读「[使用 secrets 的示例](secrets.md#simple-example-get-started-with-secrets)」。

- [`docker secret create`](/reference/cli/docker/secret/create.md)
- [`docker secret inspect`](/reference/cli/docker/secret/inspect.md)
- [`docker secret ls`](/reference/cli/docker/secret/ls.md)
- [`docker secret rm`](/reference/cli/docker/secret/rm.md)
- [`--secret`](/reference/cli/docker/service/create.md#secret) flag for `docker service create`
- [`--secret-add` and `--secret-rm`](/reference/cli/docker/service/update.md#secret-add) flags for `docker service update`

## 示例

本节通过三个由浅入深的示例，演示如何使用 Docker Secrets。示例中使用的镜像已适配 Secrets，便于上手。若要类似地改造你自己的镜像，请参见「[为镜像添加对 Docker Secrets 的支持](#build-support-for-docker-secrets-into-your-images)」。

> [!NOTE]
>
> 为简化演示，以下示例均在单引擎（single-Engine）的 Swarm 上运行，且不进行服务扩容。示例使用 Linux 容器，但 Windows 容器同样支持 Secrets。参见「[Windows 支持](#windows-support)」。

### 在 Compose 文件中定义并使用 secrets

`docker-compose` 与 `docker stack` 均支持在 Compose 文件中定义 secrets。详见「[Compose 文件参考](/reference/compose-file/legacy-versions.md)」。

### 简单示例：快速上手 secrets

该示例用少量命令演示 secrets 的基本用法。更贴近实战的示例请继续阅读「[进阶示例：在 Nginx 服务中使用 secrets](#intermediate-example-use-secrets-with-a-nginx-service)」。

1.  向 Docker 添加一个 secret。`docker secret create` 的最后一个参数用于指定读取 secret 的文件路径；当其为 `-` 时，表示从标准输入读取。

    ```console
    $ printf "This is a secret" | docker secret create my_secret_data -
    ```

2.  创建一个 `redis` 服务并授予其访问该 secret 的权限。默认情况下，容器可通过 `/run/secrets/<secret_name>` 访问该 secret；你也可以通过 `target` 选项自定义容器内的目标文件名。

    ```console
    $ docker service  create --name redis --secret my_secret_data redis:alpine
    ```

3.  使用 `docker service ps` 验证任务是否正常运行。若一切正常，输出类似：

    ```console
    $ docker service ps redis

    ID            NAME     IMAGE         NODE              DESIRED STATE  CURRENT STATE          ERROR  PORTS
    bkna6bpn8r1a  redis.1  redis:alpine  ip-172-31-46-109  Running        Running 8 seconds ago  
    ```

    若任务失败并不断重启，你会看到类似输出：

    ```console
    $ docker service ps redis

    NAME                      IMAGE         NODE  DESIRED STATE  CURRENT STATE          ERROR                      PORTS
    redis.1.siftice35gla      redis:alpine  moby  Running        Running 4 seconds ago                             
     \_ redis.1.whum5b7gu13e  redis:alpine  moby  Shutdown       Failed 20 seconds ago      "task: non-zero exit (1)"  
     \_ redis.1.2s6yorvd9zow  redis:alpine  moby  Shutdown       Failed 56 seconds ago      "task: non-zero exit (1)"  
     \_ redis.1.ulfzrcyaf6pg  redis:alpine  moby  Shutdown       Failed about a minute ago  "task: non-zero exit (1)"  
     \_ redis.1.wrny5v4xyps6  redis:alpine  moby  Shutdown       Failed 2 minutes ago       "task: non-zero exit (1)"
    ```

4.  通过 `docker ps` 获取 `redis` 服务任务容器的 ID，然后用 `docker container exec` 进入容器读取 secret 文件内容。该文件默认所有用户可读，且文件名与 secret 名一致。第一条命令演示如何查找容器 ID；第二与第三条命令使用 shell 展开自动完成。

    ```console
    $ docker ps --filter name=redis -q

    5cb1c2348a59

    $ docker container exec $(docker ps --filter name=redis -q) ls -l /run/secrets

    total 4
    -r--r--r--    1 root     root            17 Dec 13 22:48 my_secret_data

    $ docker container exec $(docker ps --filter name=redis -q) cat /run/secrets/my_secret_data

    This is a secret
    ```

5.  验证当你对容器执行 commit 时，secret 不会被提交进镜像。

    ```console
    $ docker commit $(docker ps --filter name=redis -q) committed_redis

    $ docker run --rm -it committed_redis cat /run/secrets/my_secret_data

    cat: can't open '/run/secrets/my_secret_data': No such file or directory
    ```

6.  尝试删除该 secret。由于 `redis` 服务仍在运行且拥有访问权限，因此删除会失败。

    ```console
    $ docker secret ls

    ID                          NAME                CREATED             UPDATED
    wwwrxza8sxy025bas86593fqs   my_secret_data      4 hours ago         4 hours ago


    $ docker secret rm my_secret_data

    Error response from daemon: rpc error: code = 3 desc = secret
    'my_secret_data' is in use by the following service: redis
    ```

7.  通过更新服务，撤销运行中 `redis` 服务对该 secret 的访问权限。

    ```console
    $ docker service update --secret-rm my_secret_data redis
    ```

8.  重复步骤 3 与 4，验证服务已无法访问该 secret。注意容器 ID 会不同，因为 `service update` 会重新部署该服务。

    ```console
    $ docker container exec -it $(docker ps --filter name=redis -q) cat /run/secrets/my_secret_data

    cat: can't open '/run/secrets/my_secret_data': No such file or directory
    ```

9.  停止并删除服务，同时从 Docker 中删除该 secret。

    ```console
    $ docker service rm redis

    $ docker secret rm my_secret_data
    ```

### 简单示例：在 Windows 服务中使用 secrets

这是一个在 Windows 10 上，基于 Docker for Windows 运行 Windows 容器、并以 Microsoft IIS 服务演示 secrets 的简单示例。示例较为“朴素”，把网页内容存放在 secret 中。

该示例假设你已安装 PowerShell。

1.  Save the following into a new file `index.html`.

    ```html
    <html lang="en">
      <head><title>Hello Docker</title></head>
      <body>
        <p>Hello Docker! You have deployed a HTML page.</p>
      </body>
    </html>
    ```

2.  若尚未进行，先初始化或加入 Swarm。

    ```console
    > docker swarm init
    ```

3.  Save the `index.html` file as a swarm secret named `homepage`.

    ```console
    > docker secret create homepage index.html
    ```

4.  创建一个 IIS 服务，并授予其访问 `homepage` secret 的权限。

    ```console
    > docker service create `
        --name my-iis `
        --publish published=8000,target=8000 `
        --secret src=homepage,target="\inetpub\wwwroot\index.html" `
        microsoft/iis:nanoserver
    ```

    > [!NOTE]
    >
    > 从技术上讲，本示例并不需要使用 secrets；[configs](configs.md) 更为合适。本示例仅用于演示。

5.  Access the IIS service at `http://localhost:8000/`. It should serve
    the HTML content from the first step.

6.  删除服务与 secret。

    ```console
    > docker service rm my-iis
    > docker secret rm homepage
    > docker image remove secret-test
    ```

### 进阶示例：在 Nginx 服务中使用 secrets

本示例分为两部分。[第一部分](#generate-the-site-certificate) 仅用于生成站点证书，并不直接涉及 Docker Secrets；但它是为[第二部分](#configure-the-nginx-container)做准备——在第二部分中，我们会将站点证书与 Nginx 配置以 secrets 的形式存储并使用。

#### 生成站点证书

为站点生成根 CA 与 TLS 证书及密钥。生产环境中，你可以使用 `Let’s Encrypt` 等服务生成 TLS 证书与密钥；本示例使用命令行工具。此步骤略显繁琐，但仅是为了后续能有内容可作为 Docker secret 存储。若你希望跳过这些子步骤，可[使用 Let’s Encrypt](https://letsencrypt.org/getting-started/) 生成站点密钥与证书（文件名分别为 `site.key` 与 `site.crt`），然后直接跳到「[配置 Nginx 容器](#configure-the-nginx-container)」。

1.  Generate a root key.

    ```console
    $ openssl genrsa -out "root-ca.key" 4096
    ```

2.  Generate a CSR using the root key.

    ```console
    $ openssl req \
              -new -key "root-ca.key" \
              -out "root-ca.csr" -sha256 \
              -subj '/C=US/ST=CA/L=San Francisco/O=Docker/CN=Swarm Secret Example CA'
    ```

3.  配置根 CA。新建文件 `root-ca.cnf` 并粘贴以下内容。该配置限制根 CA 仅签发叶子证书，而不能签发中间 CA：

    ```ini
    [root_ca]
    basicConstraints = critical,CA:TRUE,pathlen:1
    keyUsage = critical, nonRepudiation, cRLSign, keyCertSign
    subjectKeyIdentifier=hash
    ```

4.  签发证书。

    ```console
    $ openssl x509 -req  -days 3650  -in "root-ca.csr" \
                   -signkey "root-ca.key" -sha256 -out "root-ca.crt" \
                   -extfile "root-ca.cnf" -extensions \
                   root_ca
    ```

5.  Generate the site key.

    ```console
    $ openssl genrsa -out "site.key" 4096
    ```

6.  生成站点证书并使用站点密钥对其签名。

    ```console
    $ openssl req -new -key "site.key" -out "site.csr" -sha256 \
              -subj '/C=US/ST=CA/L=San Francisco/O=Docker/CN=localhost'
    ```

7.  配置站点证书。新建文件 `site.cnf` 并粘贴以下内容。该配置限制站点证书只能用于服务器认证，不能用于签发其他证书：

    ```ini
    [server]
    authorityKeyIdentifier=keyid,issuer
    basicConstraints = critical,CA:FALSE
    extendedKeyUsage=serverAuth
    keyUsage = critical, digitalSignature, keyEncipherment
    subjectAltName = DNS:localhost, IP:127.0.0.1
    subjectKeyIdentifier=hash
    ```

8.  签发站点证书。

    ```console
    $ openssl x509 -req -days 750 -in "site.csr" -sha256 \
        -CA "root-ca.crt" -CAkey "root-ca.key"  -CAcreateserial \
        -out "site.crt" -extfile "site.cnf" -extensions server
    ```

9.  Nginx 服务不需要 `site.csr` 与 `site.cnf` 文件；但若你想重新生成站点证书，它们仍然有用。请妥善保护 `root-ca.key` 文件。

#### 配置 Nginx 容器

1.  准备一个最基本的 Nginx 配置，用于通过 HTTPS 提供静态文件。TLS 证书与密钥将作为 Docker secrets 存储，方便后续轮换。

    In the current directory, create a new file called `site.conf` with the
    following contents:

    ```nginx
    server {
        listen                443 ssl;
        server_name           localhost;
        ssl_certificate       /run/secrets/site.crt;
        ssl_certificate_key   /run/secrets/site.key;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
    ```

2.  创建三个 secrets，分别对应密钥、证书与 `site.conf`。任意小于 500 KB 的文件都可以作为 secret 存储。这使得密钥、证书与配置可以与使用它们的服务解耦。以下命令的最后一个参数为主机文件系统中读取 secret 的文件路径；示例中 secret 名与文件名相同。

    ```console
    $ docker secret create site.key site.key

    $ docker secret create site.crt site.crt

    $ docker secret create site.conf site.conf
    ```

    ```console
    $ docker secret ls

    ID                          NAME                  CREATED             UPDATED
    2hvoi9mnnaof7olr3z5g3g7fp   site.key       58 seconds ago      58 seconds ago
    aya1dh363719pkiuoldpter4b   site.crt       24 seconds ago      24 seconds ago
    zoa5df26f7vpcoz42qf2csth8   site.conf      11 seconds ago      11 seconds ago
    ```

3.  创建一个运行 Nginx 的服务，并授予其访问上述三个 secrets 的权限。`docker service create` 命令结尾处会把 `site.conf` 的真实位置创建成指向 `/etc/nginx/conf.d/` 的符号链接，Nginx 会在该目录查找额外配置。此步骤在 Nginx 启动前完成，因此即便变更配置也无需重建镜像。

    > [!NOTE]
    >
    > Normally you would create a Dockerfile which copies the `site.conf`
    > into place, build the image, and run a container using your custom image.
    > This example does not require a custom image. It puts the `site.conf`
    > into place and runs the container all in one step.

    默认情况下，secrets 会位于容器内的 `/run/secrets/` 目录。如果你希望在其他路径使用 secret，可能需要额外操作。下面示例通过创建符号链接，让 Nginx 能从期望路径读取 `site.conf`：

    ```console
    $ docker service create \
         --name nginx \
         --secret site.key \
         --secret site.crt \
         --secret site.conf \
         --publish published=3000,target=443 \
         nginx:latest \
         sh -c "ln -s /run/secrets/site.conf /etc/nginx/conf.d/site.conf && exec nginx -g 'daemon off;'"
    ```

    除了创建符号链接外，你也可以通过 `target` 选项为 secret 指定自定义位置。下面示例展示了如何无需符号链接，就将 `site.conf` 暴露在容器内的 `/etc/nginx/conf.d/site.conf`：

    ```console
    $ docker service create \
         --name nginx \
         --secret site.key \
         --secret site.crt \
         --secret source=site.conf,target=/etc/nginx/conf.d/site.conf \
         --publish published=3000,target=443 \
         nginx:latest \
         sh -c "exec nginx -g 'daemon off;'"
    ```

    `site.key` 与 `site.crt` 使用了简写语法，未设置自定义 `target`，因此它们会以各自名称挂载在 `/run/secrets/` 下。此时，容器中将存在以下三个文件：

    - `/run/secrets/site.key`
    - `/run/secrets/site.crt`
    - `/etc/nginx/conf.d/site.conf`

4.  验证 Nginx 服务已成功运行。

    ```console
    $ docker service ls

    ID            NAME   MODE        REPLICAS  IMAGE
    zeskcec62q24  nginx  replicated  1/1       nginx:latest

    $ docker service ps nginx

    NAME                  IMAGE         NODE  DESIRED STATE  CURRENT STATE          ERROR  PORTS
    nginx.1.9ls3yo9ugcls  nginx:latest  moby  Running        Running 3 minutes ago
    ```

5.  验证服务可用性：确保可以访问 Nginx，且使用了正确的 TLS 证书。

    ```console
    $ curl --cacert root-ca.crt https://localhost:3000

    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support. refer to
    <a href="https://nginx.org">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="https://www.nginx.com">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

    ```console
    $ openssl s_client -connect localhost:3000 -CAfile root-ca.crt

    CONNECTED(00000003)
    depth=1 /C=US/ST=CA/L=San Francisco/O=Docker/CN=Swarm Secret Example CA
    verify return:1
    depth=0 /C=US/ST=CA/L=San Francisco/O=Docker/CN=localhost
    verify return:1
    ---
    Certificate chain
     0 s:/C=US/ST=CA/L=San Francisco/O=Docker/CN=localhost
       i:/C=US/ST=CA/L=San Francisco/O=Docker/CN=Swarm Secret Example CA
    ---
    Server certificate
    -----BEGIN CERTIFICATE-----
    …
    -----END CERTIFICATE-----
    subject=/C=US/ST=CA/L=San Francisco/O=Docker/CN=localhost
    issuer=/C=US/ST=CA/L=San Francisco/O=Docker/CN=Swarm Secret Example CA
    ---
    No client certificate CA names sent
    ---
    SSL handshake has read 1663 bytes and written 712 bytes
    ---
    New, TLSv1/SSLv3, Cipher is AES256-SHA
    Server public key is 4096 bit
    Secure Renegotiation IS supported
    Compression: NONE
    Expansion: NONE
    SSL-Session:
        Protocol  : TLSv1
        Cipher    : AES256-SHA
        Session-ID: A1A8BF35549C5715648A12FD7B7E3D861539316B03440187D9DA6C2E48822853
        Session-ID-ctx:
        Master-Key: F39D1B12274BA16D3A906F390A61438221E381952E9E1E05D3DD784F0135FB81353DA38C6D5C021CB926E844DFC49FC4
        Key-Arg   : None
        Start Time: 1481685096
        Timeout   : 300 (sec)
        Verify return code: 0 (ok)
    ```

6.  清理：删除 `nginx` 服务与创建的 secrets。

    ```console
    $ docker service rm nginx

    $ docker secret rm site.crt site.key site.conf
    ```

### 高阶示例：在 WordPress 服务中使用 secrets

本示例中，你将创建一个单节点 MySQL 服务并设置自定义 root 密码，将凭据以 secrets 形式存储，然后创建一个单节点的 WordPress 服务，并通过这些凭据连接到 MySQL。[下一个示例](#example-rotate-a-secret) 会在此基础上演示如何轮换 MySQL 密码并更新服务，确保 WordPress 仍可连接。

该示例展示了如何使用 Docker Secrets，避免将敏感凭据写入镜像或直接通过命令行传递。

> [!NOTE]
>
> 为简化演示，本示例使用单引擎 Swarm，且 MySQL 使用单节点。MySQL 并不能仅通过增加副本来实现水平扩展，搭建 MySQL 集群也超出了本文范围。
>
> 另外，修改 MySQL 的 root 密码并非简单地改一个磁盘文件；你需要使用查询语句或 `mysqladmin` 来修改。

1.  生成一个随机的字母数字组合作为 MySQL 密码，并通过 `docker secret create` 将其保存为名为 `mysql_password` 的 Docker secret。若要更改密码长度，调整 `openssl` 命令最后一个参数即可。这只是创建随机密码的一种方式，你也可以使用其他方法生成。

    > [!NOTE]
    >
    > 创建 secret 后无法直接更新；你只能删除并重新创建，且不能删除被服务使用中的 secret。不过，你可以通过 `docker service update` 为运行中的服务授予或撤销对 secret 的访问权限。若需要“更新”能力，建议在 secret 名称中加入版本号，以便添加新版本、更新服务并移除旧版本。

    最后一个参数为 `-`，表示从标准输入读取内容。

    ```console
    $ openssl rand -base64 20 | docker secret create mysql_password -

    l1vinzevzhj4goakjap5ya409
    ```

    返回值不是密码本身，而是 secret 的 ID。本文后续将省略该 ID 的显示。

    为 MySQL 的 `root` 用户生成第二个 secret。该 secret 不会与稍后创建的 WordPress 服务共享，只用于初始化 `mysql` 服务。

    ```console
    $ openssl rand -base64 20 | docker secret create mysql_root_password -
    ```

    使用 `docker secret ls` 列出 Docker 管理的 secrets：

    ```console
    $ docker secret ls

    ID                          NAME                  CREATED             UPDATED
    l1vinzevzhj4goakjap5ya409   mysql_password        41 seconds ago      41 seconds ago
    yvsczlx9votfw3l0nz5rlidig   mysql_root_password   12 seconds ago      12 seconds ago
    ```

    这些 secrets 会被存储在 Swarm 的加密 Raft 日志中。

2.  创建一个用户自定义 overlay 网络，供 MySQL 与 WordPress 服务之间通信使用。无需向外部主机或容器暴露 MySQL 服务。

    ```console
    $ docker network create -d overlay mysql_private
    ```

3.  创建 MySQL 服务。该服务具有以下特征：

    - 由于副本数设置为 `1`，仅运行一个 MySQL 任务。对 MySQL 的负载均衡实现不在本示例范围内，且并非仅靠增加副本即可完成。
    - 仅允许 `mysql_private` 网络中的其他容器访问。
    - 使用卷 `mydata` 存储 MySQL 数据，使其在 `mysql` 服务重启后仍可保留。
    - 这些 secrets 会以 `tmpfs` 内存文件系统的形式挂载在 `/run/secrets/mysql_password` 与 `/run/secrets/mysql_root_password`。它们不会以环境变量方式暴露；即便执行 `docker commit` 也不会被提交进镜像。`mysql_password` 是非特权的 WordPress 容器用于连接 MySQL 的 secret。
    - 设置环境变量 `MYSQL_PASSWORD_FILE` 与 `MYSQL_ROOT_PASSWORD_FILE`，分别指向 `/run/secrets/mysql_password` 与 `/run/secrets/mysql_root_password`。`mysql` 镜像在首次初始化系统数据库时会从这些文件读取密码字符串；之后，密码会存储在 MySQL 的系统数据库中。
    - 设置环境变量 `MYSQL_USER` 与 `MYSQL_DATABASE`。容器启动时会创建名为 `wordpress` 的数据库；`wordpress` 用户仅对该数据库拥有完整权限，不能创建/删除数据库，也不能修改 MySQL 配置。

      ```console
      $ docker service create \
           --name mysql \
           --replicas 1 \
           --network mysql_private \
           --mount type=volume,source=mydata,destination=/var/lib/mysql \
           --secret source=mysql_root_password,target=mysql_root_password \
           --secret source=mysql_password,target=mysql_password \
           -e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
           -e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
           -e MYSQL_USER="wordpress" \
           -e MYSQL_DATABASE="wordpress" \
           mysql:latest
      ```

4.  使用 `docker service ls` 验证 `mysql` 容器已运行。

    ```console
    $ docker service ls

    ID            NAME   MODE        REPLICAS  IMAGE
    wvnh0siktqr3  mysql  replicated  1/1       mysql:latest
    ```

5.  完成 MySQL 的准备后，创建一个连接到 MySQL 的 WordPress 服务。该服务具有以下特征：

    - 由于副本数设置为 `1`，仅运行一个 WordPress 任务。受限于会话数据存储在容器文件系统，WordPress 的负载均衡不在本示例范围内，留作读者实践。
    - 将 WordPress 暴露在宿主机的 30000 端口，便于外部访问；如果宿主机 80 端口空闲，也可以改为对外暴露 80 端口。
    - 连接到 `mysql_private` 网络以便与 `mysql` 容器通信；同时在所有 Swarm 节点上将容器的 80 端口发布到 30000 端口。
    - 拥有对 `mysql_password` secret 的访问权限，但在容器内指定了不同的目标文件名；WordPress 容器使用的挂载路径为 `/run/secrets/wp_db_password`。
    - 设置环境变量 `WORDPRESS_DB_PASSWORD_FILE` 为该 secret 的挂载路径。WordPress 服务会从该文件读取 MySQL 密码，并写入 `wp-config.php` 配置文件。
    - 使用用户名 `wordpress` 与 `/run/secrets/wp_db_password` 中的密码连接到 MySQL；若不存在，将创建名为 `wordpress` 的数据库。
    - 将主题与插件等数据存储在名为 `wpdata` 的卷中，以确保服务重启后仍可保留。

    ```console
    $ docker service create \
         --name wordpress \
         --replicas 1 \
         --network mysql_private \
         --publish published=30000,target=80 \
         --mount type=volume,source=wpdata,destination=/var/www/html \
         --secret source=mysql_password,target=wp_db_password \
         -e WORDPRESS_DB_USER="wordpress" \
         -e WORDPRESS_DB_PASSWORD_FILE="/run/secrets/wp_db_password" \
         -e WORDPRESS_DB_HOST="mysql:3306" \
         -e WORDPRESS_DB_NAME="wordpress" \
         wordpress:latest
    ```

6.  使用 `docker service ls` 与 `docker service ps` 验证服务运行状态。

    ```console
    $ docker service ls

    ID            NAME       MODE        REPLICAS  IMAGE
    wvnh0siktqr3  mysql      replicated  1/1       mysql:latest
    nzt5xzae4n62  wordpress  replicated  1/1       wordpress:latest
    ```

    ```console
    $ docker service ps wordpress

    ID            NAME         IMAGE             NODE  DESIRED STATE  CURRENT STATE           ERROR  PORTS
    aukx6hgs9gwc  wordpress.1  wordpress:latest  moby  Running        Running 52 seconds ago   
    ```

    此时理论上可以撤销 WordPress 对 `mysql_password` secret 的访问，因为 WordPress 已将其写入 `wp-config.php`。不过暂时不要这么做，我们稍后还会用到它来演示密码轮换。

7.  在任意 Swarm 节点访问 `http://localhost:30000/` 并通过 Web 向导完成 WordPress 初始化。上述设置会保存在 MySQL 的 `wordpress` 数据库中。WordPress 会为你的站点用户自动生成一个密码，这与 WordPress 访问 MySQL 的密码完全不同。请妥善保存该密码（例如使用密码管理器），在[轮换 secret](#example-rotate-a-secret) 后需要用它登录 WordPress。

    现在可以尝试发表一两篇文章，并安装一个 WordPress 插件或主题，以验证 WordPress 正常工作，且其状态能在服务重启后被保留。

8.  如果你打算继续下一示例（演示如何轮换 MySQL root 密码），现在不要清理任何服务或 secrets。

### 示例：轮换 secret

本示例基于上一示例：你将创建一个新的 MySQL 密码并保存为新的 secret，更新 `mysql` 与 `wordpress` 服务以使用它，然后移除旧 secret。

> [!NOTE]
>
> 更改 MySQL 数据库密码需要执行额外的查询或命令，而非修改一个环境变量或文件。镜像仅会在数据库不存在时设置密码；默认情况下，密码存储在 MySQL 自身的数据库中。因此，轮换密码或其他 secrets 往往需要 Docker 之外的配合步骤。

1.  Create the new password and store it as a secret named `mysql_password_v2`.

    ```console
    $ openssl rand -base64 20 | docker secret create mysql_password_v2 -
    ```

2.  更新 MySQL 服务，使其同时访问旧与新两个 secrets。注意你无法直接更新或重命名 secret，但可以撤销并以新的目标文件名重新授予访问。

    ```console
    $ docker service update \
         --secret-rm mysql_password mysql

    $ docker service update \
         --secret-add source=mysql_password,target=old_mysql_password \
         --secret-add source=mysql_password_v2,target=mysql_password \
         mysql
    ```

    更新服务会触发重启；第二次启动后，MySQL 服务可以同时在 `/run/secrets/old_mysql_password` 与 `/run/secrets/mysql_password` 下访问旧/新密码。

    尽管 MySQL 服务此时可访问两个 secrets，但 WordPress 用户的 MySQL 密码尚未变更。

    > [!NOTE]
    >
    > This example does not rotate the MySQL `root` password.

3.  接着，使用 `mysqladmin` CLI 修改 `wordpress` 用户的 MySQL 密码。该命令会从 `/run/secrets` 中读取旧/新密码，不会在命令行暴露或写入 shell 历史。

    请快速完成并继续下一步，因为此时 WordPress 会暂时无法连接 MySQL。

    First, find the ID of the `mysql` container task.

    ```console
    $ docker ps --filter name=mysql -q

    c7705cf6176f
    ```

    Substitute the ID in the command below, or use the second variant which
    uses shell expansion to do it all in a single step.

    ```console
    $ docker container exec <CONTAINER_ID> \
        bash -c 'mysqladmin --user=wordpress --password="$(< /run/secrets/old_mysql_password)" password "$(< /run/secrets/mysql_password)"'
    ```

    Or:

    ```console
    $ docker container exec $(docker ps --filter name=mysql -q) \
        bash -c 'mysqladmin --user=wordpress --password="$(< /run/secrets/old_mysql_password)" password "$(< /run/secrets/mysql_password)"'
    ```

4.  更新 `wordpress` 服务以使用新密码，同时保持目标路径 `/run/secrets/wp_db_password` 不变。该操作会触发滚动重启，并生效新 secret。

    ```console
    $ docker service update \
         --secret-rm mysql_password \
         --secret-add source=mysql_password_v2,target=wp_db_password \
         wordpress    
    ```

5.  再次在任意 Swarm 节点访问 http://localhost:30000/ 验证 WordPress 可用。使用你在上一示例中通过向导创建的 WordPress 用户名与密码登录。

    Verify that the blog post you wrote still exists, and if you changed any
    configuration values, verify that they are still changed.

6.  从 MySQL 服务撤销旧 secret 的访问权限，并将其从 Docker 中删除。

    ```console
    $ docker service update \
         --secret-rm mysql_password \
         mysql

    $ docker secret rm mysql_password
    ```


7.  运行以下命令，删除 WordPress 与 MySQL 服务、移除 `mydata` 与 `wpdata` 卷，并清理相关 secrets：

    ```console
    $ docker service rm wordpress mysql

    $ docker volume rm mydata wpdata

    $ docker secret rm mysql_password_v2 mysql_root_password
    ```

## 为镜像添加对 Docker Secrets 的支持

如果你要构建一个以服务形式部署的容器，且需要以环境变量形式提供凭据等敏感数据，建议改造镜像以利用 Docker Secrets。做法之一是：确保创建容器时传入镜像的每个参数，都可以改为从文件读取。

许多来自 [Docker Library](https://github.com/docker-library/) 的官方镜像（例如上文使用的 [wordpress](https://github.com/docker-library/wordpress/)）已按此方式适配。

启动 WordPress 容器时，通常通过环境变量传入必要参数。该镜像已更新：凡是重要参数（如 `WORDPRESS_DB_PASSWORD`），都提供了从文件读取的变体（如 `WORDPRESS_DB_PASSWORD_FILE`）。这既保证了向后兼容性，也允许容器从 Docker 管理的 secret 中读取信息，而非直接通过环境变量传入。

> [!NOTE]
>
> Docker Secrets 不会直接注入环境变量。这样设计是为了避免环境变量在容器之间被意外泄露（例如使用 `--link` 时）。

## 在 Compose 中使用 Secrets

```yaml

services:
   db:
     image: mysql:latest
     volumes:
       - db_data:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_root_password
       - db_password

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_password


secrets:
   db_password:
     file: db_password.txt
   db_root_password:
     file: db_root_password.txt

volumes:
    db_data:
```

下面的示例使用 Compose 文件与两个 secrets，创建一个简单的 WordPress 站点。

顶层的 `secrets` 元素定义了两个 secrets：`db_password` 与 `db_root_password`。

部署时，Docker 会创建这两个 secrets，并将 Compose 文件中指定的文件内容写入其中。

`db` 服务使用这两个 secrets，`wordpress` 服务使用其中一个。

部署后，Docker 会在服务容器内挂载 `/run/secrets/<secret_name>` 文件。这些文件不会持久化到磁盘，而是在内存中管理。

每个服务通过环境变量指定在容器内读取 secret 的路径。

关于 secrets 的短/长语法，详见《[Compose 规范](/reference/compose-file/secrets.md)》。
