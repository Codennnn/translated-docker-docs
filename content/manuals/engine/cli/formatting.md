---
description: CLI 与日志输出格式化参考
keywords: format, formatting, output, templates, log
title: 格式化命令与日志输出
weight: 40
aliases:
  - /engine/admin/formatting/
  - /config/formatting/
---

Docker 支持使用 [Go 模板](https://golang.org/pkg/text/template/) 来控制部分命令和日志驱动的输出格式。

Docker 还提供了一组基础函数，用于在模板中处理各个元素。下面的示例均使用 `docker inspect`，但很多 CLI 命令都支持 `--format` 标志，并在参考文档中给出了自定义输出格式的示例。

> [!NOTE]
>
> 使用 `--format` 时请注意你的 shell 行为差异。
> 在 POSIX shell 中，可直接使用单引号：
>
> ```console
> $ docker inspect --format '{{join .Args " , "}}'
> ```
>
> 在 Windows（如 PowerShell）中，同样使用单引号，但需要对参数中的双引号进行转义：
>
> ```console
> $ docker inspect --format '{{join .Args \" , \"}}'
> ```

## join（连接）

`join` 将字符串列表连接为单个字符串，并在各元素之间插入分隔符。

```console
$ docker inspect --format '{{join .Args " , "}}' container
```

## table（表格）

`table` 指定输出中应包含哪些字段。

```console
$ docker image list --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

## json（JSON 编码）

`json` 将元素编码为 JSON 字符串。

```console
$ docker inspect --format '{{json .Mounts}}' container
```

## lower（转小写）

`lower` 将字符串转换为小写。

```console
$ docker inspect --format "{{lower .Name}}" container
```

## split（分割）

`split` 按给定分隔符将字符串切分为字符串列表。

```console
$ docker inspect --format '{{split .Image ":"}}' container
```

## title（首字母大写）

`title` 将字符串的首字符大写。

```console
$ docker inspect --format "{{title .Name}}" container
```

## upper（转大写）

`upper` 将字符串转换为大写。

```console
$ docker inspect --format "{{upper .Name}}" container
```

## pad（填充）

`pad` 为字符串添加空格填充。你可以分别指定前后填充的空格数量。

```console
$ docker image list --format '{{pad .Repository 5 10}}'
```

上例在镜像仓库名之前添加 5 个空格、之后添加 10 个空格。

## truncate（截断）

`truncate` 将字符串截断到指定长度；若原字符串更短，则保持不变。

```console
$ docker image list --format '{{truncate .Repository 15}}'
```

上例将镜像仓库名截断为最多 15 个字符。

## println（换行输出）

`println` 将每个值单独输出在一行。

```console
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{println .IPAddress}}{{end}}' container
```

## 提示

想了解可用的数据字段，可先以 JSON 形式输出整条记录：

```console
$ docker container ls --format='{{json .}}'
```
