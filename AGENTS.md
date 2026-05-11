# AGENTS.md

AI agent 协作框架源模板。所有 agent（planner / builder / reviewer / teacher / maintainer / maintainer_flash / aide）启动时自动加载。

## 环境契约

具体环境约束（语言版本、虚拟环境、依赖管理、命令前缀、自检命令、禁止事项等）由 **planner 在项目启动时写进该项目的 `PLAN.md` 第 0 节"环境契约（不可协商）"**。

- builder 和 reviewer **必须**在启动时读该项目的 `PLAN.md` 第 0 节，严格遵循其中定义的命令调用规则
- 项目级命令（pytest / mypy / ruff / pip / cargo / pnpm 等）应在该项目目录下执行
- 如果 `<项目目录>/PLAN.md` 不存在或第 0 节为空，**所有 bash / 包安装命令必须暂停**——planner 必须先和用户澄清技术栈
- **硬件环境**：开发者本地硬件信息（OS / CPU / RAM / GPU）见 [`docs/developer-environment.md`](./docs/developer-environment.md)（首次使用：`cp docs/developer-environment.template.md docs/developer-environment.md` 并填入你的硬件信息）。planner 在起草 PLAN.md 第 0 节时必须先读此文件，确保 task 对 GPU 显存 / 内存的需求不超出可用硬件；如项目需要更高算力，PLAN 中应注明"需租用云 GPU"及推荐配置
> 本机 conda 环境清单、OS/CPU/GPU/CUDA/磁盘等硬件信息集中记录在各项目的
> `docs/conda-environments.md` 中。
> planner 在起草 PLAN.md 第 0 节时应先读该文件，避免假设错误的环境配置。

## 配置模式

### 模式 A：纯用户默认（大多数项目）

项目根**没有** `opencode.jsonc`。7 个 agent 直接从用户级配置（`~/.config/opencode/opencode.jsonc`）加载，零设置。

### 模式 B：项目覆盖（个别项目特殊需求）

项目根放 `opencode.jsonc`，只写要覆盖的 agent/字段。它**合并叠加**到用户级配置上——没写的 agent 保持不变。

```jsonc
// 示例：项目 A 给 builder 切换了模型
{
  "agent": {
    "builder": {
      "model": "openrouter/anthropic/claude-sonnet-4.6"
    }
  }
}
```

## 工作流

新项目的标准流程：

1. **起项目**：用户在宿主仓库中告诉 planner 项目需求 + 项目名（slug，推荐小写字母 + 连字符，如 `ts-link-checker`、`llm-gateway`）
2. **写计划**：Planner 在用户确认的项目目录下创建 `PLAN.md`（第 0 节为环境契约）
3. **实现 task**：Builder 切换到项目目录（`cd <项目目录>` 或命令加路径前缀），按 PLAN 顺序实现，每批 ≤ 3 个 → 跑测试 → 召唤 reviewer
4. **审查**：Reviewer 独立验证测试 / lint / 类型检查，输出 APPROVED / NEEDS-FIX / REJECTED；`REVIEW.md` 写到 `<项目目录>/REVIEW.md`
5. **交付**：项目交付后，builder 自动调用 teacher 子 agent 生成学习材料（写到`<项目目录>/learning-notes/`）

## 多项目共存

- 项目存放在用户确认的目录下，planner 会向用户询问路径
- 各项目独立，不交叉污染
- **不交叉污染**：项目 A 的 builder 不能读写项目 B 的文件；reviewer、teacher 同理
- **项目名用 slug**（小写字母 + 连字符），无空格、无大写、无中文
- **各项目的 manifest 独立**：不同项目的 `pyproject.toml` / `package.json` 互不影响
- **editable install 注意路径**：Python 项目应在项目目录内 `pip install -e .`（不在仓库根），避免不同项目的 src layout 冲突
- 所有项目共享同一套 agent 配置和 prompt

planner 启动时会按项目状态自动做检测-决定（新项目 / 继续 / 已交付），详见 [`agent-prompts/planner.md`](./agent-prompts/planner.md) 的工作流程。

## 自动编号约定

agent 在生成需要编号的文件或目录（如代码库模块目录、`docs/` 下的文档等）时，遵循以下约定：

### 起号约定

| 场景 | 起始号 | 示例 |
|---|---|---|
| 目录/系列的第一个索引文件（总览、roadmap） | `0.0` | `0.0-ROADMAP.md`、`0.0-index.md` |
| 正文内容文件 | `1.0` | `1.0-基础篇.md`、`2.1-进阶.md` |
| 附录/参考类 | 从 `A.0` 或 `Z.0` | `A.0-术语表.md`、`Z.0-changelog.md` |

### 编号层级

采用 `X.Y-名称.md` 格式：
- `X` — 章/主题排序：正文从 `1` 起，索引从 `0` 起
- `Y` — 节内排序：从 `0` 或 `1` 起均可，目录内统一即可

各目录内部自成编号，不继承上级目录的编号层级。不强求全目录统一深度——内容简单用两级 `X.Y`，复杂可追加三级 `X.Y.Z`。

### 插入与重排

新增文件插入中间时，对**之后**的文件顺延编号。如 `1.0` 和 `2.0` 之间插入一篇 → 原 `2.0` 改为 `3.0`，原 `3.0` 改为 `4.0`，以此类推。

### 不需要编号的文件

- 纯模板文件（如 `TEMPLATE.md`）
- 单个独立文件无系列关系的（如 `agent-rules.md`）
- 图片、PDF 等附件
- `README.md`（目录内说明文件）

## Git Commit 约定

### 分支策略

| 分支 | 用途 | 谁可以操作 |
|---|---|---|
| `dev` | 日常开发，所有改动先合入此分支 | **所有 agent**（planner / builder / maintainer 均在 dev 上工作） |
| `main` | 稳定发布版本，仅接收 `dev` 的 merge + push 到远程 | **仅 maintainer**，且需用户完整下令 |

> **硬性规则（违反即严重错误）**：
> 1. **所有改动在 `dev` 上完成**——含 README 版本更新、文档、配置、prompts。`main` 只接受 `dev` 的 merge，**禁止任何 agent 直接 commit 到 `main`**。
> 2. **commit 与 push 的默认分支**：用户说「提交」/「commit」→ 默认在 `dev` 上 commit。用户说「push」→ 默认指 push `main`，完整流程为：确保 `dev` 工作树干净 → merge `dev` 到 `main` →（如果用户要求打 tag 则打上）→ push `main` + tags → 立即切回 `dev`。
> 3. **main 仅 maintainer 可操作**——planner / builder / maintainer_flash 不能操作 main。maintainer 也需用户完整下令（如「merge 到 main」/「push」）才动 main。
> 4. **其他 primary agent 操作仓库前必须确认当前分支为 `dev`**——不在 `dev` 则拒绝操作并提醒用户切回。
> 5. **仅 `main` 推送到 GitHub**——`dev` 仅本地开发，不推送远端。

### Commit Message 格式

使用 Conventional Commits 风格，保证历史一致、可追溯：

- **宿主仓库项目级 commit**（builder）：`<type>: T0X T0Y - <一句话概述>`
  - 示例：`feat: T02 T03 - core stats and preprocessing`
  - 示例：`fix: MF-01 - strip_frontmatter no longer .lstrip() body newlines`
  - 项目收尾：`chore: project complete - all T01-T07 delivered`
- **框架仓库 commit**（maintainer）：`<type>(scope): <描述>`
  - 示例：`fix(config): correct reasoningEffort camelCase`
  - 示例：`docs: add submodule usage instructions to AGENTS and README`

类型前缀：`feat:` / `fix:` / `chore:` / `refactor:` / `docs:`

### 红线

- Commit message 必须真实反映改动——不写空话
- 不写 AI 生成水印
- 框架仓库改动与宿主仓库项目改动分开 commit——绝不混在一个 commit 里

### 语言风格

- 使用清晰、平实的书面语言，不过度口语化
- 代码注释和 docstring 可以使用适度的对话风格增强可读性，但不含网络梗
- 这是软要求——内容准确性和完整性优先，在此前提下注意措辞

## Agent 职责边界

| Agent | 作用域 | 写权限 | 典型用途 |
|---|---|---|---|
| `planner` | 宿主仓库单项目 | `<项目目录>/PLAN.md` + `<项目目录>/docs/**` | 起草项目计划，修订 PLAN |
| `builder` | 宿主仓库单项目 | 项目源码 / 测试 / `PROGRESS.md` | 实现、测试、commit、召唤 reviewer |
| `reviewer` | 宿主仓库单项目 | 仅 `<项目目录>/REVIEW.md` | 独立验证测试 / lint / 类型检查；输出 APPROVED/NEEDS-FIX/REJECTED |
| `teacher` | 宿主仓库单项目 | `<项目目录>/learning-notes/**` | 项目交付后由 builder 调用，生成学习材料 |
| `aide` | 框架文档层 | 仅 `docs/**` + `templates/**` | maintainer 的廉价 subagent（V4 Flash + reasoning）。承担可验证的审计/搜索/总结任务，maintainer 审核其输出 |
| `maintainer` | **框架仓库全层** | 项目文件需用户明确授权。其他路径 allow | 框架维护、修改配置/prompts、一致性审计、回答仓库设计问题 |
| `maintainer_flash` | 全仓库（只读） | 无（只读，仅对话输出） | 轻量版 maintainer——回答仓库结构/设计问题、代码导航、解释代码。不能修改文件或跑命令 |

**maintainer 是唯一跨层可写（framework + 项目）agent**。planner / builder / reviewer / teacher 严格聚焦宿主仓库的单项目。`maintainer_flash` 是全仓库只读——适用于需要框架知识但不需要修改的场景（可替代原 explainer 的职责）。`aide` 是 maintainer 的专用执行器，仅操作文档层。`aide` 只能由 maintainer 通过 Task 工具调用。

**适合找 maintainer 的场景**：

- 修改 framework 自身（prompts、配置、文档）
- 回答仓库结构、设计相关问题
- 处理"非标准"需求（不属于 plan / build / review / teach 流程）

**不应该找 maintainer 的场景**（用专门 agent）：

- 起草新项目 PLAN → `planner`
- 实现具体 task → `builder`
- 审查 task 完成情况 → `reviewer`
- 讲解项目设计思路 → `teacher`

## 同步约定（硬性规则）

以下三份文件在框架仓库与 `~/.config/opencode/` 中各有一份副本，互为模板/副本关系——框架侧是源模板，用户侧是实际生效的副本：

| 文件 | 框架侧路径 | 用户侧路径 |
|---|---|---|
| `AGENTS.md` | `ai-agent-framework/AGENTS.md` | `~/.config/opencode/AGENTS.md` |
| `opencode.jsonc` | `ai-agent-framework/config/opencode.jsonc` | `~/.config/opencode/opencode.jsonc` |
| `opencode.openrouter.jsonc.bak` | `ai-agent-framework/config/opencode.openrouter.jsonc.bak` | `~/.config/opencode/opencode.openrouter.jsonc.bak` |

### 规则 1：写入权限仅限 maintainer

除 maintainer 外，**任何 agent**（planner / builder / reviewer / teacher / maintainer_flash / aide）均不得 edit 或 write 以上任一文件。此规则在 `opencode.jsonc` 的 permission 层硬封，并由各 agent prompt 明文重申。

### 规则 2：改一侧立即同步另一侧

maintainer 修改上述任一文件后，**必须在同一次 session 内**将改动同步到另一侧的所有对应文件。禁止"先记下来稍后同步"。同步的操作步骤见 `agent-prompts/maintainer.md` Rule 2。

### 规则 3：两侧差异仅限以下两类

- **路径格式**：框架侧用相对路径（`{file:./agent-prompts/...}`），用户侧用本机绝对路径
- **脱敏信息**：框架侧开源，用占位符（`<YOUR-CONDA-ENV-PYTHON>` 等）；用户侧填入本机实际值

**除以上两类差异外，两侧文件的 agent 列表、permission 规则、model 配置、MCP 配置结构、所有文档段落，分毫不差。**

### 规则 4：框架内部两份 opencode.jsonc 同步

`opencode.jsonc` 与 `opencode.openrouter.jsonc.bak` 的 model 字段可以不同，其余（agent 列表 / permission / 角色边界）必须一致。改一份必须同时改另一份。

## 更新框架

`git pull` 拉取框架新版本后，`agent-prompts/` 立即生效。但如果 `opencode.jsonc` 或 `AGENTS.md` 有结构变更，需 maintainer 按规则 2（框架侧 → 用户侧）手动同步到 `~/.config/opencode/`。

## 本地 MCP 工具

以下 MCP (Model Context Protocol) 工具在用户级配置中启用，所有 agent 均可通过标准 tool call 调用。每个 MCP server 可调用多个 tool，tool 的描述和参数由 MCP 协议自动传递给 agent。

> **这些 MCP 工具为可选补充，非必需品**。多模态模型（如 GPT-4V、Claude Sonnet、Gemini）拥有原生视觉/音频理解能力，可直接处理同类任务。本地工具适用于纯文本模型、离线环境，或需要本地计算保障隐私/零成本的场景。

### gh_grep — GitHub 代码搜索

| 属性 | 值 |
|---|---|
| 类型 | 远程 MCP (`https://mcp.grep.app`) |
| Tool | `gh_grep_searchGitHub` |
| 用途 | 搜索公开 GitHub 仓库中的真实代码示例，辅助 agent 学习不熟悉的 API / 框架用法 |

### qwen3_asr — 语音转文字 (ASR)

| 属性 | 值 |
|---|---|
| 命令 | `qwen-asr` conda 环境 → `asr/asr_mcp_server.py` |
| Tools | `transcribe_audio`, `asr_status` |
| REST 端口 | `8000` |
| 模型 | Qwen3-ASR-1.7B，支持 52 种语言 |
| 音频格式 | WAV, MP3, FLAC, OGG 等 |

**自动唤醒**：首次调用 `transcribe_audio` 时，MCP server 会检测 ASR REST 服务是否在线（`localhost:8000/health`）。离线则自动执行 `asr/qwen3_asr_start.sh` 后台启动，轮询等待就绪（最长 60s）。

**用法示例**：
```
transcribe_audio("/home/user/recording.wav")
transcribe_audio("/home/user/interview.mp3", language="zh")
asr_status()                       -- 返回服务状态和 GPU 信息
```

### glm_ocr — 文档 OCR 解析

| 属性 | 值 |
|---|---|
| 命令 | `glm-ocr` conda 环境 → `ocr/glm_ocr_mcp_server.py` |
| Tools | `ocr_glm`, `ocr_glm_status` |
| REST 端口 | `8002` |
| 模型 | GLM-OCR (0.9B VLM)，显存占用约 2.5 GB |
| 文档格式 | PNG, JPG, BMP, TIFF, WEBP, PDF |
| 输出 | Markdown（含 LaTeX 公式 + 表格），支持中英文、手写体 |

**自动唤醒**：首次调用 `ocr_glm` 时，MCP server 会检测 OCR REST 服务是否在线（`localhost:8002/health`）。离线则自动执行 `ocr/glm_ocr_start.sh` 后台启动，轮询等待就绪（最长 90s；首次需加载模型约 3s，后续重启约 2s）。

**空闲超时**：REST 服务 30s 无请求后自动停止释放 GPU。下次调用时自动重启，对 agent 透明——agent 不需要关心 OCR 服务的启停，只需调用 `ocr_glm()` 即可。

**用法示例**：
```
ocr_glm("/home/user/document.pdf")                       -- 返回 Markdown
ocr_glm("/home/user/photo.png", output_format="json")     -- 返回完整 JSON（含逐页详情）
ocr_glm_status()                                          -- 返回服务状态和 GPU 信息
```

### qwen_vision — 视觉理解 / 图像描述

| 属性 | 值 |
|---|---|
| 命令 | `glm-ocr` conda 环境 → `vl/vision_mcp_server.py` |
| Tools | `describe_image`, `vision_status` |
| REST 端口 | `8080` |
| 模型 | Qwen3.6-35B-A3B（MoE VLM，Q4_K_XL） |
| 图像格式 | PNG, JPG/JPEG, GIF, BMP, WEBP |
| 输出 | JSON `{description, model, tokens_used}`，英文自然语言 |

**自动唤醒**：首次调用 `describe_image` 时，MCP server 会检测 Vision REST 服务是否在线（`localhost:8080/health`）。离线则自动执行 `vl/llama_start.sh` 后台启动，轮询等待就绪（最长时间 120s）。

**空闲超时**：不支持（llama-server 为独立进程，不设自动退出；需手动 `vl/llama_start.sh stop`）。

**用法示例**：
```
describe_image("/home/user/photo.jpg")                     -- 返回图像描述
describe_image("/home/user/photo.jpg", detail="high")      -- 高细节描述
vision_status()                                            -- 返回服务状态和 GPU 信息
```

### 本地 MCP 依赖的 conda 环境

| MCP | conda 环境 | 关键依赖 |
|---|---|---|
| `qwen3_asr` | `qwen-asr` | PyTorch, Transformers, FastAPI, MCP |
| `glm_ocr` | `glm-ocr` | PyTorch, Transformers >= 5.3.0, FastAPI, MCP |
| `qwen_vision` | `glm-ocr` | httpx, FastMCP, MCP |

> MCP server 进程由 OpenCode 自动管理（启动/停止），agent 无需手动操作。REST 后端服务的启停由 MCP server 内部的自动唤醒逻辑处理。

## 设计原则

这些原则是所有 agent 行为的基石——违反任何一条视为严重错误：

- **planner 不写代码、不跑命令**：只写 PLAN.md。代码层面的细节属于 builder
- **环境契约写在 PLAN.md 第 0 节**：仓库不绑定任何语言，技术栈按项目决定
- **reviewer 独立验证**：必须自行重跑测试 / 类型检查 / lint——不信任 builder 的输出
- **三态结论**：APPROVED / NEEDS-FIX / REJECTED，不留灰色地带
- **Plan-Issue 回退机制**：PLAN 本身有缺陷时立刻停下，报告用户，不强行推进
- **配置层硬封 > prompt 层软约束**：能用 permission 封死的，不靠 prompt"劝"
- **教训反馈环**：每个项目的 bug → 提炼通用法则 → 写回 agent prompt 和 `docs/agent-rules.md`。判断标准：这条知识换个宿主仓库还有用吗？有用 → 直接改 framework 仓库的 prompt / 文档；只和当前项目代码有关 → 留在宿主仓库

> 工程法则（agent 必读）：[`docs/agent-rules.md`](./docs/agent-rules.md)

## 仓库维护策略

以下策略适用于所有使用本框架的仓库：

### Tag 与版本管理

- **Tag 上限**：任何仓库的本地和远程 tag 只保留最新 10 个。打出新 tag 后，删除第 11 个及更早的旧 tag（远程删除前向用户确认）。
- **README 版本表裁剪**：`README.md` 的"版本更新"表格只显示最新 10 行记录，与 tag 数量对齐。
- **执行者**：此策略由 `maintainer` agent 在执行 tag 操作时自动落实（见 `agent-prompts/maintainer.md` Rule 6）。

## 已知风险与逃生通道

### 1. 同 session Tab 切换可能不刷新 system prompt

**应对**：每个 agent 一个 session。切换 agent 时退出重新 `opencode` 进入，确保 system prompt 正确加载。

### 2. 终端被 kill 不会丢 session

opencode session 持久化到 `~/.local/share/opencode/`。`opencode -c` 重新连接最近一次 session。

### 3. 内置 Build / Plan agent ≠ 自定义 builder / planner

opencode TUI 同时存在两套 agent。内置的不受本仓库约束——可以在工作流死锁时作为逃生通道。
