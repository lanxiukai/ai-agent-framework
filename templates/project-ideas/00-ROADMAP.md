# AI Agent 开发——项目建议（从易到难）

> 本目录收录 5 个 AI Agent 开发项目 prompt，按难度从 MCP 协议 → Agent Runtime → 多 Agent TUI 递进。
> 建议按编号顺序 ①→⑤ 逐项完成——每个项目都构建在前一个的基础上。

## 分层架构

```
┌─────────────────────────────────────┐
│  Layer 3: Agent 应用                │  ← 多 agent 协作 / TUI
├─────────────────────────────────────┤
│  Layer 2: Agent 框架 / 协议         │  ← agent loop / skill 系统
├─────────────────────────────────────┤
│  Layer 1: Agent 工具 / 通信         │  ← MCP 协议 / tool 桥接
└─────────────────────────────────────┘
```

## 推荐学习路径

```
① mcp-bridge ──→ ② mcp-client ──→ ③ skill-loader
                                       │
     ┌─────────────────────────────────┘
     ↓
④ agent-core ──→ ⑤ openclaw-lite
```

> **前置条件**：理解 LLM API 调用流程和 prompt 工程基础。如果还不太熟，先花 1-2 小时读一遍 OpenAI Chat Completions API 文档和 Anthropic 的 prompt 设计指南。

---

## 项目列表

| 项目号 | Slug | 难度 | 说明 |
|--------|------|------|------|
| ① | `mcp-bridge` | ⭐⭐ | MCP 协议 server 实现，将常用 API 封装为标准 MCP tool |
| ② | `mcp-client` | ⭐⭐ | MCP 多 server 发现、连接、编排与统一 tool 命名空间 |
| ③ | `skill-loader` | ⭐⭐ | 基于文件系统的 Skill 发现、匹配、加载器（prompt 组合与上下文预算） |
| ④ | `agent-core` | ⭐⭐⭐ | 从零实现最小 Agent Runtime 核心循环（零框架依赖） |
| ⑤ | `openclaw-lite` | ⭐⭐⭐ | 多 Agent Session 管理器（TUI + SQLite 持久化） |

---

## ① mcp-bridge —— MCP 协议工具桥接（⭐⭐ 中等，Rust 或 TypeScript）

实现一个 MCP（Model Context Protocol）server，将常用 API（GitHub、文件系统、数据库、HTTP）封装为标准 MCP tool。

- **能学到**：MCP 协议（JSON-RPC over stdio/SSE）、tool schema 设计、agent-tool 交互边界
- **实用价值**：通过 MCP 接入 agent，解耦 opencode 内置工具
- **核心功能**：≥4 个 tool（file_read/write、http_get、shell_exec、github_issue_create）、JSON Schema tool 描述、标准化错误响应
- **预估**：6-8 个 task，Rust（serde_json + tokio）或 TypeScript

## ② mcp-client —— MCP 多 Server 发现与编排（⭐⭐ 中等，Rust 或 TypeScript）

① 教的是"怎么做一个 MCP tool provider"（server 端）。这个项目教你**另一半**——怎么做一个 MCP client 来发现、连接、消费多个 MCP server 的 tool，并将它们统一暴露给 agent。

- **能学到**：MCP client 协议（initialize → list_tools → call_tool 生命周期）、多 server 路由与 tool 命名冲突解决、SSE vs stdio transport 选择、tool 调用超时与重试
- **实用价值**：生产环境中 MCP server 是分散的（GitHub MCP、文件系统 MCP、数据库 MCP 各跑各的），client 负责把它们组合成一个统一的 tool 命名空间。①+② 学完才算真懂 MCP
- **核心功能**：连接 ≥3 个 MCP server（可以是 ① 做的 + 社区 server）、tool 发现与去重、统一 tool 列表暴露给 LLM、调用日志与耗时统计
- **预估**：5-7 个 task，Rust（tokio + serde_json）或 TypeScript

## ③ skill-loader —— 基于文件系统的 Skill 注册与加载器（⭐⭐ 中等，Python 或 Rust）

Skills（opencode 的 `/skill`、Claude 的 custom instructions）本质上是**按需加载的专用 prompt + 工具集**。这个项目做一个从文件系统发现、匹配、加载 skill 的系统——核心挑战不是"读文件"，而是 prompt 组合优先级、上下文窗口预算管理、工具路由。

> 这个项目放在 agent-core 之前：先理解"如何组合 prompt 和工具"，再动手实现 agent runtime，思考层次才对。

- **能学到**：prompt 片段组合与冲突解决、上下文窗口预算（每个 skill 消耗 N token，总预算 X，加载策略）、基于标签/关键词的 skill 匹配、YAML frontmatter 元数据设计
- **实用价值**：agent 框架设计的核心能力——按需加载能省 50%+ 的上下文 token
- **核心功能**：`skills/` 目录下 `.md` 文件自动注册、`skill.list()` / `skill.match()` / `skill.load()` API、上下文窗口阈值告警、命中率统计
- **预估**：6-7 个 task，Python（pyyaml + tiktoken）或 Rust

## ④ agent-core —— 从零实现最小 Agent Runtime（⭐⭐⭐ 困难，Rust / Go / Python）

实现一个单 agent runtime 核心循环：system prompt 加载 → LLM 调用 → tool call 解析 → tool 执行 → 结果反馈 → 下一轮。零框架依赖。

> Python 实现仅建议用于学习（理解 agent loop 原理），生产环境推荐 Rust/Go。

- **能学到**：agent loop 基本原理、tool calling 协议内部、streaming 中断与恢复、context window 管理
- **实用价值**：理解"opencode 内部怎么工作"的最佳方式
- **核心功能**：OpenAI function calling 格式、≥3 个 tool、多轮对话、简单 TUI
- **预估**：7-10 个 task，Rust（tokio + async-stream + ratatui）/ Go / Python（asyncio + rich）

## ⑤ openclaw-lite —— 多 Agent Session 管理器（⭐⭐⭐ 困难，Rust + TUI）

构建一个简化版 opencode/openclaw：管理多个 agent session（每个有自己的 system prompt + 对话历史 + tools），支持 session 切换、fork、导出。

- **能学到**：TUI 框架（ratatui）、session 持久化（SQLite）、终端事件循环、多 agent 上下文隔离
- **实用价值**：可能不替代 opencode，但你会深刻理解 agent TUI 的内部设计
- **核心功能**：3 个内置 agent 模板（coder/reviewer/planner）、session 列表 + 切换、markdown 渲染、tool call 可视化
- **预估**：8-12 个 task

---

## 核心建议

1. **按顺序走，不要跳**。① mcp-bridge 教 tool 暴露，② mcp-client 教 tool 消费，③ skill-loader 教 prompt 组合，④ agent-core 教 agent loop——每一环都是下一环的前置。

2. **MCP 要学两端**。① mcp-bridge（server）教你"怎么暴露 tool"，② mcp-client 教你"怎么消费 tool"。只学一端等于只学了一半协议。

3. **skill-loader 是 agent-core 的认知前置**。先理解 prompt 组合和上下文预算，再实现 agent loop——否则你会写出一个"能跑但不省钱"的 agent。

4. **语言选择**：①-③ 可用 Python 或 TypeScript，④-⑤ 建议 Rust/Go——agent runtime 和 TUI 对性能和并发的要求更高。

---

## 与其他仓库的关系

| 仓库 | 聚焦 | 与本仓库的关系 |
|------|------|----------------|
| **本仓库**（`ai-agent-framework`） | Agent 框架 / MCP 协议 / Agent Runtime | — |
| [AI Infra 仓库](https://github.com/lanxiukai/ai-infra-practical-projects) | LLM API 网关 / prompt 工程 / RAG / vLLM 部署 | 前置。先理解 LLM 怎么用，再学 agent 怎么造 |
| [AI Model 仓库](https://github.com/lanxiukai/ai-model-practical-projects) | 大模型预训练 / 微调（LoRA / SFT / DreamBooth） | 进阶方向。理解模型内部后再做 agent 会更扎实 |

> 建议学习顺序：AI Infra（理解 LLM 怎么用）→ AI Model（理解模型怎么训）→ AI Agent（理解 agent 怎么造）

---

> 想新增项目 idea？**必须**使用 [`TEMPLATE.md`](./TEMPLATE.md) 模板，按序号+slug命名放入此目录。新增后同步更新本文件的推荐路径和分层架构图。
