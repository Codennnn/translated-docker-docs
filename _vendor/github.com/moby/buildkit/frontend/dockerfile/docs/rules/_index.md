---
title: 构建检查
description: |
  BuildKit 内置支持基于一组预定义规则来分析构建配置，
  用于强制执行 Dockerfile 和构建最佳实践。
keywords: buildkit, 检查, dockerfile, 前端, 规则
---

BuildKit 内置支持基于一组预定义规则来分析构建配置，用于强制执行 Dockerfile 和构建最佳实践。遵循这些规则有助于避免错误并确保 Dockerfile 的良好可读性。

检查以构建调用的方式运行，但不会生成构建输出，而是执行一系列检查来验证构建是否违反任何规则。要运行检查，请使用 `--check` 标志：

```console
$ docker build --check .
```

要了解有关如何使用构建检查的更多信息，请参阅[检查构建配置](https://docs.docker.com/build/checks/)。

<table>
  <thead>
    <tr>
      <th>名称</th>
      <th>描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="./stage-name-casing/">StageNameCasing</a></td>
      <td>阶段名称应该使用小写字母</td>
    </tr>
    <tr>
      <td><a href="./from-as-casing/">FromAsCasing</a></td>
      <td>'as' 关键字应该与 'from' 关键字的大小写保持一致</td>
    </tr>
    <tr>
      <td><a href="./no-empty-continuation/">NoEmptyContinuation</a></td>
      <td>空续行将在未来版本中变为错误</td>
    </tr>
    <tr>
      <td><a href="./consistent-instruction-casing/">ConsistentInstructionCasing</a></td>
      <td>Dockerfile 中的所有命令应该使用相同的大小写（全大写或全小写）</td>
    </tr>
    <tr>
      <td><a href="./duplicate-stage-name/">DuplicateStageName</a></td>
      <td>阶段名称应该是唯一的</td>
    </tr>
    <tr>
      <td><a href="./reserved-stage-name/">ReservedStageName</a></td>
      <td>保留字不应该用作阶段名称</td>
    </tr>
    <tr>
      <td><a href="./json-args-recommended/">JSONArgsRecommended</a></td>
      <td>建议对 ENTRYPOINT/CMD 使用 JSON 参数格式，以防止与操作系统信号相关的意外行为</td>
    </tr>
    <tr>
      <td><a href="./maintainer-deprecated/">MaintainerDeprecated</a></td>
      <td>MAINTAINER 指令已弃用，请改用标签来定义镜像作者</td>
    </tr>
    <tr>
      <td><a href="./undefined-arg-in-from/">UndefinedArgInFrom</a></td>
      <td>FROM 命令必须使用已声明的 ARG</td>
    </tr>
    <tr>
      <td><a href="./workdir-relative-path/">WorkdirRelativePath</a></td>
      <td>如果在构建中没有声明绝对工作目录，相对工作目录可能会在基础镜像更改时产生意外结果</td>
    </tr>
    <tr>
      <td><a href="./undefined-var/">UndefinedVar</a></td>
      <td>变量应该在使用前定义</td>
    </tr>
    <tr>
      <td><a href="./multiple-instructions-disallowed/">MultipleInstructionsDisallowed</a></td>
      <td>在同一阶段中不应该使用相同类型的多个指令</td>
    </tr>
    <tr>
      <td><a href="./legacy-key-value-format/">LegacyKeyValueFormat</a></td>
      <td>不应使用带空格分隔符的旧键值格式</td>
    </tr>
    <tr>
      <td><a href="./redundant-target-platform/">RedundantTargetPlatform</a></td>
      <td>在 FROM 中将平台设置为预定义的 $TARGETPLATFORM 是多余的，因为这是默认行为</td>
    </tr>
    <tr>
      <td><a href="./secrets-used-in-arg-or-env/">SecretsUsedInArgOrEnv</a></td>
      <td>敏感数据不应该在 ARG 或 ENV 命令中使用</td>
    </tr>
    <tr>
      <td><a href="./invalid-default-arg-in-from/">InvalidDefaultArgInFrom</a></td>
      <td>全局 ARG 的默认值导致空或无效的基础镜像名称</td>
    </tr>
    <tr>
      <td><a href="./from-platform-flag-const-disallowed/">FromPlatformFlagConstDisallowed</a></td>
      <td>FROM --platform 标志不应该使用常量值</td>
    </tr>
    <tr>
      <td><a href="./copy-ignored-file/">CopyIgnoredFile（实验性）</a></td>
      <td>尝试复制被 .dockerignore 排除的文件</td>
    </tr>
    <tr>
      <td><a href="./invalid-definition-description/">InvalidDefinitionDescription（实验性）</a></td>
      <td>构建阶段或参数的注释应该遵循格式：`# <参数/阶段名称> <描述>`。如果不打算作为描述注释，请在指令和注释之间添加空行或注释。</td>
    </tr>
    <tr>
      <td><a href="./expose-proto-casing/">ExposeProtoCasing</a></td>
      <td>EXPOSE 指令中的协议应该使用小写字母</td>
    </tr>
    <tr>
      <td><a href="./expose-invalid-format/">ExposeInvalidFormat</a></td>
      <td>EXPOSE 指令中不应该使用 IP 地址和主机端口映射。这将在未来版本中变为错误</td>
    </tr>
  </tbody>
</table>
