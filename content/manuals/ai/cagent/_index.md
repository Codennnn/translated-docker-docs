---
title: cagent
description: cagent 让你构建、编排并共享协同工作的 AI 代理（agents）。
weight: 60
params:
  sidebar:
    group: Open source
keywords: [ai, agent, cagent]
---

{{< summary-bar feature_name="cagent" >}}

[cagent](https://github.com/docker/cagent) 让你可以构建、编排并共享 AI 代理。
你可以用它来定义协同工作的 AI 代理团队。

cagent 基于“_root agent_（根代理）”的概念运作：它充当团队负责人，
并把任务分派给你定义的子代理。
每个代理：
- 使用你选择的模型，并采用你设置的参数。
- 可访问[内置工具](#built-in-tools)与在
  [Docker MCP 网关](/manuals/ai/mcp-catalog-and-toolkit/mcp-gateway.md)中配置的 MCP 服务器。
- 在各自独立的上下文中工作，彼此不共享知识。

根代理是你的主要交互入口。每个代理都有自己的上下文，
它们不会共享知识。

## 关键特性

- 多租户架构：支持客户端隔离与会话管理。
- 通过 MCP（Model Context Protocol）集成的丰富工具生态。
- 分层式代理系统，支持智能任务分派。
- 多种接口形态：CLI、TUI、API 服务器与 MCP 服务器。
- 通过集成 Docker 仓库进行代理分发。
- 安全优先的设计：合理的客户端作用域与资源隔离。
- 事件驱动的流式传输，支持实时交互。
- 多模型支持（OpenAI、Anthropic、Gemini、DMR、Docker AI Gateway）。

## 快速上手 cagent

1. 下载适用于你操作系统的[最新发行版](https://github.com/docker/cagent/releases)。

   > [!NOTE]
   > 你可能需要为该二进制文件赋予可执行权限。
   > 在 macOS 与 Linux 上，运行：

     ```console
     chmod +x /path/to/downloads/cagent-linux-<arm/amd>64
     ```

   > [!NOTE]
   > 你也可以从源码构建 cagent。参见[仓库说明](https://github.com/docker/cagent?tab=readme-ov-file#build-from-source)。

1. 可选：按需重命名二进制文件，并更新 PATH 以包含 cagent 可执行文件。

1. 设置以下环境变量：

   ```bash
   # 如果使用 Docker AI Gateway，请设置此环境变量，或使用
   # `--models-gateway <url_to_docker_ai_gateway>` CLI 标志

   export CAGENT_MODELS_GATEWAY=<url_to_docker_ai_gateway>

   # 或者，为远程推理服务设置密钥。
   # 如果使用 Docker AI Gateway，则不需要这些。

   export OPENAI_API_KEY=<your_api_key_here>    # 用于 OpenAI 模型
   export ANTHROPIC_API_KEY=<your_api_key_here> # 用于 Anthropic 模型
   export GOOGLE_API_KEY=<your_api_key_here>    # 用于 Gemini 模型
   ```

1. 通过将以下示例保存为 `assistant.yaml` 创建一个代理：

   ```yaml {title="assistant.yaml"}
   agents:
     root:
       model: openai/gpt-5-mini
       description: 一个乐于助人的 AI 助手
       instruction: |
         你是一名知识渊博的助手，帮助用户完成各类任务。
         回答需有帮助、准确且简明。
   ```

1. 使用你的代理开始对话：

   ```bash
   cagent run assistant.yaml
   ```

## 创建一个“代理团队”（agentic team）

你可以使用 `cagent new` 命令，通过 AI 提示（prompting）生成一组代理：

```console
$ cagent new

For any feedback, visit: https://docker.qualtrics.com/jfe/form/SV_cNsCIg92nQemlfw

Welcome to cagent! (Ctrl+C to exit)

What should your agent/agent team do? (describe its purpose):

> I need a cross-functional feature team. The team owns a specific product
  feature end-to-end. Include the key responsibilities of each of the roles
  involved (engineers, designer, product manager, QA). Keep the description
  short, clear, and focused on how this team delivers value to users and the business.
```

或者，你也可以手动编写配置文件。例如：

```yaml {title="agentic-team.yaml"}
agents:
  root:
    model: claude
    description: "负责分派任务并管理工作流的主协调代理"
    instruction: |
      You are the root coordinator agent. Your job is to:
      1. 理解用户请求并将其拆解为可管理的任务。
      2. 将合适的任务分派给辅助代理。
      3. 协调结果，确保任务顺利完成。
      4. 向用户提供最终回应。
      当你收到请求时，分析需要完成的事项，并决定：
      - 如果任务简单，则自行处理；
      - 如果需要特定帮助，则交给辅助代理；
      - 将复杂请求拆分为多个子任务。
    sub_agents: ["helper"]

  helper:
    model: claude
    description: "在根代理的指示下完成各类任务的辅助代理"
    instruction: |
      You are a helpful assistant agent. Your role is to:
      1. 完成根代理分配的具体任务；
      2. 提供详尽且准确的结果；
      3. 若任务不清楚，请主动澄清；
      4. 将结果反馈给根代理。

      请在任何被分配的任务中尽可能充分且有帮助。

models:
  claude:
    provider: anthropic
    model: claude-sonnet-4-0
    max_tokens: 64000
```

[See the reference documentation](https://github.com/docker/cagent?tab=readme-ov-file#-configuration-reference).

## 内置工具（Built-in tools）

cagent 提供一组内置工具，用于增强代理能力。
使用这些工具无需配置任何外部 MCP 工具。

```yaml
agents:
  root:
    # ... other config
    toolsets:
      - type: todo
      - type: transfer_task
```

### Think 工具

Think 工具允许代理按步骤推理问题：

```yaml
agents:
  root:
    # ... other config
    toolsets:
      - type: think
```

### Todo 工具

Todo 工具有助于代理管理任务清单：

```yaml
agents:
  root:
    # ... other config
    toolsets:
      - type: todo
```

### Memory 工具

Memory 工具提供持久化存储：

```yaml
agents:
  root:
    # ... other config
    toolsets:
      - type: memory
        path: "./agent_memory.db"
```

### 任务转移工具（Task transfer）

任务转移工具是一个内部工具，允许某代理将任务分派给子代理。若要阻止代理分派任务，请确保其配置中未定义子代理。

### 通过 Docker MCP 网关使用工具

如果你使用 [Docker MCP 网关](/manuals/ai/mcp-catalog-and-toolkit/mcp-gateway.md)，
可以配置你的代理与该网关交互，并使用其中配置的 MCP 服务器。参见 [docker mcp gateway run](/reference/cli/docker/mcp/gateway/gateway_run.md)。

例如，启用代理通过 MCP 网关使用 Duckduckgo：

```yaml
toolsets:
  - type: mcp
    command: docker
    args: ["mcp", "gateway", "run", "--servers=duckduckgo"]
```

## CLI 交互命令

在与代理的 CLI 会话中，你可以使用以下命令：

| Command  | Description                              |
|----------|------------------------------------------|
| /exit    | 退出程序                                  |
| /reset   | 清空会话历史                               |
| /eval    | 保存当前会话以供评估                       |
| /compact | 压缩当前会话                               |

## 分享你的代理

你可以将代理配置打包并通过 Docker Hub 进行共享。
开始之前，请确保你已有一个[Docker 仓库](/manuals/docker-hub/repos/create.md)。

推送代理：

```bash
cagent push ./<agent-file>.yaml <namespace>/<reponame>
```

拉取代理到当前目录：

```bash
cagent pull <namespace>/<reponame>
```

代理的配置文件名为 `<namespace>_<reponame>.yaml`。通过 `cagent run <filename>` 运行。

## 相关页面

- 更多关于 cagent 的信息，见
[GitHub 仓库](https://github.com/docker/cagent)。
- [Docker MCP 网关](/manuals/ai/mcp-catalog-and-toolkit/mcp-gateway.md)