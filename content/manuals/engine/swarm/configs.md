---
title: 使用 Docker Configs 存储配置信息
description: 如何将配置信息与运行时解耦存储
keywords: swarm, configuration, configs
---

## 关于 configs

Docker 的 swarm 服务配置（configs）允许你将非敏感信息（如配置文件）存储在服务镜像与运行中容器之外。这样可以让镜像尽可能通用，而无需把配置文件 bind-mount 到容器中，或通过环境变量注入。

configs 与 [secrets](secrets.md) 的工作方式类似，但有以下不同：configs 在静态存储时不会被加密，并且是直接挂载到容器的文件系统（不通过内存盘）。你可以在任意时间向服务添加或移除 configs，服务之间也可以共享同一个 config。你甚至可以将 configs 与环境变量或标签结合使用，以获得更大的灵活性。config 的值可以是通用字符串或二进制内容（大小上限 500 KB）。

> [!NOTE]
>
> Docker configs 仅对 swarm 服务可用，不适用于独立容器。如需使用该能力，可考虑将你的容器改为以副本数为 1 的服务运行。

configs 同时支持 Linux 与 Windows 服务。

### Windows 支持

Docker 在 Windows 容器中也支持 configs，但实现上存在差异（下文示例会注明）。请注意以下几点：

- 对于自定义目标路径（target）的配置文件，Windows 不会直接以 bind-mount 的方式挂载到容器内，因为 Windows 不支持非目录文件的 bind-mount。相反，容器的所有 configs 都会挂载到容器内的 `C:\ProgramData\Docker\internal\configs`（实现细节，不应被应用依赖），再通过符号链接指向 config 的目标路径。默认目标路径为 `C:\ProgramData\Docker\configs`。
- 当创建使用 Windows 容器的服务时，`UID`、`GID` 与 `mode` 这些选项对 configs 不生效。当前 configs 仅对容器内的管理员与具备 `system` 访问权限的用户可见。
- 在 Windows 上，使用 `--credential-spec` 并以 `config://<config-name>` 的格式创建或更新服务。这样可在容器启动前将 gMSA 凭据文件直接下发到节点，而不会在工作节点磁盘上写入 gMSA 凭据。详见[将服务部署到 swarm](services.md#gmsa-for-swarm)。

## Docker 如何管理 configs

当你向 swarm 添加一个 config 时，Docker 会通过双向 TLS 连接将其发送到 swarm 管理节点。该 config 会存入经过加密的 Raft 日志中；整个 Raft 日志会在其他管理节点之间复制，从而为 configs 提供与其他 swarm 管理数据一致的高可用保障。

当你为新建或正在运行的服务授予某个 config 的访问权限时，该 config 会以文件的形式挂载到容器中。在 Linux 容器内，挂载点默认为 `/<config-name>`；在 Windows 容器内，所有 configs 都会挂载到 `C:\ProgramData\Docker\configs`，并通过符号链接指向目标位置，默认目标为 `C:\<config-name>`。

你可以为 config 设置属主（`uid` 与 `gid`，可用数字 ID 或用户名/组名），也可以指定文件权限（`mode`）。这些设置对 Windows 容器会被忽略。

- 若未设置，config 的属主为运行容器命令的用户（通常为 `root`），属组为该用户的默认组（通常也为 `root`）。
- 若未设置，config 的默认权限为所有人可读（`0444`）。如果容器中设置了 `umask`，则最终权限会受该 `umask` 影响。

你可以随时更新服务以授予其访问更多 configs，或撤销其对某个 config 的访问。

只有当节点是管理节点，或正在运行已被授予 config 访问权限的服务任务时，节点才会获得对这些 configs 的访问权。当任务容器停止运行后，共享给它的 configs 会从该容器的内存文件系统卸载，并从该节点的内存中清除。

如果某个节点在运行拥有 config 访问权限的任务容器时与 swarm 失联，该任务容器仍可访问其已有的 configs，但在节点重新连回 swarm 之前无法接收更新。

你可以随时添加或检查单个 config，或列出全部 configs。但无法删除正在被某个运行中服务使用的 config。如何在不中断运行中服务的情况下删除 config，参见[轮换配置](#example-rotate-a-config)。

为了更方便地更新与回滚 configs，建议在 config 名称中加入版本号或日期。由于可以自定义 config 在容器内的挂载点，这种命名方式会更易于管理。

要更新某个栈（stack），请修改 Compose 文件并重新执行 `docker stack deploy -c <new-compose-file> <stack-name>`。若该文件中引用了新的 config，对应的服务会开始使用它们。注意配置是不可变的，不能直接修改已有服务使用的 config 文件；需要创建一个新的 config 来替换。

你也可以使用 `docker stack rm` 停止应用并下线该栈。该命令会移除任何由同一栈名通过 `docker stack deploy` 创建的 config，也就是说会移除所有 configs，包括未被服务引用的，以及在执行 `docker service update --config-rm` 后仍保留的 configs。

## 进一步了解 `docker config` 命令

你可以通过以下链接了解具体命令，或直接跳转到[在服务中使用 configs 的示例](#advanced-example-use-configs-with-a-nginx-service)。

- [`docker config create`](/reference/cli/docker/config/create.md)
- [`docker config inspect`](/reference/cli/docker/config/inspect.md)
- [`docker config ls`](/reference/cli/docker/config/ls.md)
- [`docker config rm`](/reference/cli/docker/config/rm.md)

## 示例

本节通过循序渐进的示例演示如何使用 Docker configs。

> [!NOTE]
>
> 为简化说明，这些示例使用单 Engine 的 swarm 与未扩容的服务。示例基于 Linux 容器，但 Windows 容器同样支持 configs。

### 在 Compose 文件中定义与使用 configs

`docker stack` 命令支持在 Compose 文件中定义 configs。然而，`docker compose` 暂不支持 `configs` 键。细节参见[Compose 文件参考](/reference/compose-file/legacy-versions.md)。

### 简单示例：快速上手 configs

这个简单示例仅用少量命令演示 configs 的基本用法。若需实际场景示例，请继续阅读[高级示例：在 Nginx 服务中使用 configs](#advanced-example-use-configs-with-a-nginx-service)。

1.  向 Docker 添加一个 config。`docker config create` 命令会读取标准输入，因为最后一个参数（代表要读取的文件）设置为 `-`。

    ```console
    $ echo "This is a config" | docker config create my-config -
    ```

2.  创建一个 `redis` 服务并授予其对该 config 的访问权限。默认情况下，容器可在 `/my-config` 读取该 config；你也可以通过 `target` 选项自定义容器内的文件名。

    ```console
    $ docker service create --name redis --config my-config redis:alpine
    ```

3.  使用 `docker service ps` 验证任务是否正常运行。若一切正常，输出类似：

    ```console
    $ docker service ps redis

    ID            NAME     IMAGE         NODE              DESIRED STATE  CURRENT STATE          ERROR  PORTS
    bkna6bpn8r1a  redis.1  redis:alpine  ip-172-31-46-109  Running        Running 8 seconds ago
    ```

4.  使用 `docker ps` 获取 `redis` 服务任务容器的 ID，以便用 `docker container exec` 连接到容器并读取 config 文件内容。该文件默认所有人可读，且文件名与 config 名称一致。第一条命令演示如何查找容器 ID，第二三条命令使用 shell 补全实现自动化：

    ```console
    $ docker ps --filter name=redis -q

    5cb1c2348a59

    $ docker container exec $(docker ps --filter name=redis -q) ls -l /my-config

    -r--r--r--    1 root     root            12 Jun  5 20:49 my-config

    $ docker container exec $(docker ps --filter name=redis -q) cat /my-config

    This is a config
    ```

5.  尝试删除该 config。由于 `redis` 服务正在运行且仍拥有对该 config 的访问权限，删除会失败：

    ```console

    $ docker config ls

    ID                          NAME                CREATED             UPDATED
    fzwcfuqjkvo5foqu7ts7ls578   hello               31 minutes ago      31 minutes ago


    $ docker config rm my-config

    Error response from daemon: rpc error: code = 3 desc = config 'my-config' is
    in use by the following service: redis
    ```

6.  通过更新服务移除其对该 config 的访问权限：

    ```console
    $ docker service update --config-rm my-config redis
    ```

7.  重复第 3、4 步，确认服务已无法访问该 config。注意容器 ID 会不同，因为 `service update` 会重新部署服务：

    ```console
    $ docker container exec -it $(docker ps --filter name=redis -q) cat /my-config

    cat: can't open '/my-config': No such file or directory
    ```

8.  停止并移除该服务，并从 Docker 中移除该 config：

    ```console
    $ docker service rm redis

    $ docker config rm my-config
    ```

### 简单示例：在 Windows 服务中使用 configs

下面是一个非常简单的示例，展示如何在 Docker for Windows（运行 Windows 容器，宿主为 Windows 10）上为 Microsoft IIS 服务使用 configs。该示例为了演示，将网页内容直接存放在 config 中。

本示例假设你已安装 PowerShell。

1.  新建文件 `index.html`，写入以下内容：

    ```html
    <html lang="en">
      <head><title>Hello Docker</title></head>
      <body>
        <p>Hello Docker! You have deployed a HTML page.</p>
      </body>
    </html>
    ```

2.  若尚未执行，先初始化或加入 swarm：

    ```powershell
    docker swarm init
    ```

3.  将 `index.html` 作为名为 `homepage` 的 swarm config 保存：

    ```powershell
    docker config create homepage index.html
    ```

4.  创建一个 IIS 服务并授予其对 `homepage` config 的访问权限：

    ```powershell
    docker service create
        --name my-iis
        --publish published=8000,target=8000
        --config src=homepage,target="\inetpub\wwwroot\index.html"
        microsoft/iis:nanoserver
    ```

5.  访问 `http://localhost:8000/`，应能看到第一步中的 HTML 内容。

6.  清理：移除服务与 config。

    ```powershell
    docker service rm my-iis

    docker config rm homepage
    ```

### 示例：使用模板化 config

如果需要让配置内容由模板引擎生成，可使用 `--template-driver` 参数并指定引擎名称。模板会在容器创建时渲染。

1.  新建 `index.html.tmpl` 文件：

    ```html
    <html lang="en">
      <head><title>Hello Docker</title></head>
      <body>
        <p>Hello {{ env "HELLO" }}! I'm service {{ .Service.Name }}.</p>
      </body>
    </html>
    ```

2.  将 `index.html.tmpl` 作为名为 `homepage` 的 swarm config 保存，并通过 `--template-driver` 指定模板引擎为 `golang`：

    ```console
    $ docker config create --template-driver golang homepage index.html.tmpl
    ```

3.  创建一个运行 Nginx 的服务，并让其能够访问环境变量 `HELLO` 与该 config：

    ```console
    $ docker service create \
         --name hello-template \
         --env HELLO="Docker" \
         --config source=homepage,target=/usr/share/nginx/html/index.html \
         --publish published=3000,target=80 \
         nginx:alpine
    ```

4.  验证服务已就绪：能够访问 Nginx，并返回正确的输出。

    ```console
    $ curl http://0.0.0.0:3000

    <html lang="en">
      <head><title>Hello Docker</title></head>
      <body>
        <p>Hello Docker! I'm service hello-template.</p>
      </body>
    </html>
    ```

### 高级示例：在 Nginx 服务中使用 configs {#advanced-example-use-configs-with-a-nginx-service}

该示例分为两部分。[第一部分](#generate-the-site-certificate) 用于生成站点证书，与 Docker configs 无直接关系，但为[第二部分](#configure-the-nginx-container) 做准备：在第二部分中，你会把站点证书作为一组 secrets 存储，并将 Nginx 配置作为一个 config 存储。示例还展示了如何为 config 设置选项（如容器内目标路径与文件权限 `mode`）。

#### 生成站点证书 {#generate-the-site-certificate}

生成站点所需的根 CA、TLS 证书与密钥。在生产环境中，你可能希望使用 `Let’s Encrypt` 之类的服务来生成 TLS 证书与密钥；本示例则使用命令行工具。该步骤稍显繁琐，但仅用于准备要存入 Docker secret 的材料。若想跳过这些子步骤，可以[使用 Let’s Encrypt](https://letsencrypt.org/getting-started/) 生成站点密钥与证书，将文件命名为 `site.key` 与 `site.crt`，然后跳至[配置 Nginx 容器](#configure-the-nginx-container)。

1.  生成根密钥：

    ```console
    $ openssl genrsa -out "root-ca.key" 4096
    ```

2.  使用根密钥生成 CSR：

    ```console
    $ openssl req \
              -new -key "root-ca.key" \
              -out "root-ca.csr" -sha256 \
              -subj '/C=US/ST=CA/L=San Francisco/O=Docker/CN=Swarm Secret Example CA'
    ```

3.  配置根 CA。新建 `root-ca.cnf`，写入以下内容，用于限制根 CA 只能签发叶子证书，不可签发中间 CA：

    ```ini
    [root_ca]
    basicConstraints = critical,CA:TRUE,pathlen:1
    keyUsage = critical, nonRepudiation, cRLSign, keyCertSign
    subjectKeyIdentifier=hash
    ```

4.  签发根证书：

    ```console
    $ openssl x509 -req -days 3650 -in "root-ca.csr" \
                   -signkey "root-ca.key" -sha256 -out "root-ca.crt" \
                   -extfile "root-ca.cnf" -extensions \
                   root_ca
    ```

5.  生成站点密钥：

    ```console
    $ openssl genrsa -out "site.key" 4096
    ```

6.  生成站点证书并使用站点密钥签名：

    ```console
    $ openssl req -new -key "site.key" -out "site.csr" -sha256 \
              -subj '/C=US/ST=CA/L=San Francisco/O=Docker/CN=localhost'
    ```

7.  配置站点证书。新建 `site.cnf`，写入以下内容，限制该证书只能用于服务器认证，且不能用于签发证书：

    ```ini
    [server]
    authorityKeyIdentifier=keyid,issuer
    basicConstraints = critical,CA:FALSE
    extendedKeyUsage=serverAuth
    keyUsage = critical, digitalSignature, keyEncipherment
    subjectAltName = DNS:localhost, IP:127.0.0.1
    subjectKeyIdentifier=hash
    ```

8.  签发站点证书：

    ```console
    $ openssl x509 -req -days 750 -in "site.csr" -sha256 \
        -CA "root-ca.crt" -CAkey "root-ca.key" -CAcreateserial \
        -out "site.crt" -extfile "site.cnf" -extensions server
    ```

9.  `site.csr` 与 `site.cnf` 并非 Nginx 服务运行所必需，但若需生成新证书则仍需要它们。请妥善保护 `root-ca.key`。

#### 配置 Nginx 容器 {#configure-the-nginx-container}

1.  准备一个最小化的 Nginx 配置，通过 HTTPS 提供静态文件。TLS 证书与密钥存储为 Docker secrets，便于后续轮换：

    新建 `site.conf`，写入以下内容：

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

2.  创建两个 secrets，分别代表密钥与证书。任意小于 500 KB 的文件都可以作为 secret 存储。这使得你可以将密钥与证书与其使用的服务解耦。在本示例中，secret 名称与文件名相同：

    ```console
    $ docker secret create site.key site.key

    $ docker secret create site.crt site.crt
    ```

3.  将 `site.conf` 保存为一个 Docker config。第一个参数为 config 名称，第二个参数为读取的文件：

    ```console
    $ docker config create site.conf site.conf
    ```

    列出 configs：

    ```console
    $ docker config ls

    ID                          NAME                CREATED             UPDATED
    4ory233120ccg7biwvy11gl5z   site.conf           4 seconds ago       4 seconds ago
    ```

4.  创建一个运行 Nginx 的服务，并授予其访问上述两个 secrets 与该 config 的权限。将权限模式设为 `0440`，使文件仅对属主与属组可读，其他用户不可读：

    ```console
    $ docker service create \
         --name nginx \
         --secret site.key \
         --secret site.crt \
         --config source=site.conf,target=/etc/nginx/conf.d/site.conf,mode=0440 \
         --publish published=3000,target=443 \
         nginx:latest \
         sh -c "exec nginx -g 'daemon off;'"
    ```

    在运行中的容器内，现有以下三个文件：

    - `/run/secrets/site.key`
    - `/run/secrets/site.crt`
    - `/etc/nginx/conf.d/site.conf`

5.  验证 Nginx 服务正在运行：

    ```console
    $ docker service ls

    ID            NAME   MODE        REPLICAS  IMAGE
    zeskcec62q24  nginx  replicated  1/1       nginx:latest

    $ docker service ps nginx

    NAME                  IMAGE         NODE  DESIRED STATE  CURRENT STATE          ERROR  PORTS
    nginx.1.9ls3yo9ugcls  nginx:latest  moby  Running        Running 3 minutes ago
    ```

6.  验证服务可用：能够访问 Nginx，并使用了正确的 TLS 证书。

    ```console
    $ curl --cacert root-ca.crt https://0.0.0.0:3000

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

    <p>For online documentation and support, refer to
    <a href="https://nginx.org">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="https://www.nginx.com">www.nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

    ```console
    $ openssl s_client -connect 0.0.0.0:3000 -CAfile root-ca.crt

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

7.  若不继续下一示例，请清理本示例产生的资源：移除 `nginx` 服务及保存的 secrets 与 config。

    ```console
    $ docker service rm nginx

    $ docker secret rm site.crt site.key

    $ docker config rm site.conf
    ```

现在你已经将 Nginx 服务的配置与镜像解耦。你可以使用完全相同的镜像运行多个站点，但使用不同的配置，而无需构建任何自定义镜像。

### 示例：轮换配置 {#example-rotate-a-config}

轮换 config 的做法是：先将更新后的配置以一个不同的名称保存为新 config；然后重新部署服务，移除旧 config，并在容器内相同的挂载点添加新 config。该示例基于上一节，演示如何轮换 `site.conf`。

1.  在本地编辑 `site.conf`，在 `index` 行末尾添加 `index.php`，并保存：

    ```nginx
    server {
        listen                443 ssl;
        server_name           localhost;
        ssl_certificate       /run/secrets/site.crt;
        ssl_certificate_key   /run/secrets/site.key;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm index.php;
        }
    }
    ```

2.  使用新的 `site.conf` 创建一个 Docker config，命名为 `site-v2.conf`：

    ```bah
    $ docker config create site-v2.conf site.conf
    ```

3.  更新 `nginx` 服务，使其使用新 config 替代旧 config：

    ```console
    $ docker service update \
      --config-rm site.conf \
      --config-add source=site-v2.conf,target=/etc/nginx/conf.d/site.conf,mode=0440 \
      nginx
    ```

4.  使用 `docker service ps nginx` 确认 `nginx` 服务已完全重新部署。完成后，即可删除旧的 `site.conf`：

    ```console
    $ docker config rm site.conf
    ```

5.  清理：可移除 `nginx` 服务，以及对应的 secrets 与 configs：

    ```console
    $ docker service rm nginx

    $ docker secret rm site.crt site.key

    $ docker config rm site-v2.conf
    ```

至此，你已在无需重建镜像的情况下完成了对 `nginx` 服务配置的更新。
