---
title: 在 Docker Compose 中安全管理机密
linkTitle: Compose 中的机密
weight: 60
description: 了解如何在 Docker Compose 中安全管理运行时与构建时机密。
keywords: secrets, compose, security, environment variables, docker secrets, secure Docker builds, sensitive data in containers
tags: [Secrets]
aliases:
- /compose/use-secrets/
---

“机密”（secret）是指不应通过网络明文传输、也不应以未加密形式保存在 Dockerfile 或应用源码中的任何敏感数据，例如密码、证书或 API 密钥。

{{% include "compose/secrets.md" %}}

环境变量通常对所有进程可见，访问审计也较困难；在调试错误时，它们还可能被意外写入日志。使用 secrets 能有效降低这些风险。

## 使用 secrets

容器内的 secrets 会以文件形式挂载在 `/run/secrets/<secret_name>`。

将 secret 注入容器可分两步完成：首先，在 Compose 文件中使用[顶层 `secrets` 元素](/reference/compose-file/secrets.md)定义机密；然后，在服务定义中通过[服务级 `secrets` 属性](/reference/compose-file/services.md#secrets)声明所需机密。Compose 会按服务维度授予对机密的访问权限。

与其他方法不同，这种方式可利用标准文件系统权限在服务容器内实现细粒度的访问控制。

## 示例

### 单服务注入机密

在以下示例中，`myapp` 服务被授予对 `my_secret` 的访问权限。在容器内，`/run/secrets/my_secret` 的内容来自文件 `./my_secret.txt`。

```yaml
services:
  myapp:
    image: myapp:latest
    secrets:
      - my_secret
secrets:
  my_secret:
    file: ./my_secret.txt
```

### 多服务共享机密与密码管理

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
上述进阶示例中：

- 每个服务下的 `secrets` 属性用于声明要注入到该容器的机密。
- 顶层 `secrets` 段定义了 `db_password` 与 `db_root_password`，并通过 `file` 指定其取值来源。
- 当容器启动部署时，Docker 会在 `/run/secrets/<secret_name>` 下创建对应的绑定挂载（bind mount）。

> [!NOTE]
>
> 这里演示的 `_FILE` 环境变量是一种约定，被部分镜像采用，包括 Docker 官方镜像如 [mysql](https://hub.docker.com/_/mysql) 与 [postgres](https://hub.docker.com/_/postgres)。

### 构建时机密（Build secrets）

在以下示例中，`npm_token` 作为构建时机密提供，其值来自环境变量 `NPM_TOKEN`。

```yaml
services:
  myapp:
    build:
      secrets:
        - npm_token
      context: .

secrets:
  npm_token:
    environment: NPM_TOKEN
```

## 参考

- [顶层 `secrets` 元素](/reference/compose-file/secrets.md)
- [服务级 `secrets` 属性](/reference/compose-file/services.md#secrets)
- [构建时机密](https://docs.docker.com/build/building/secrets/)
