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

| 分支 | 用途 | 谁可操作 |
|---|---|---|
| `dev` | 日常开发，所有改动先合入此分支 | 所有 agent（planner / builder / maintainer 均在 dev 上工作） |
| `main` | 稳定发布，仅接收 `dev` merge + push | 仅 maintainer（需用户完整下令） |

硬规则：所有改动在 `dev` 上 commit，禁止直接 commit 到 `main`；commit 默认 `dev`，push 默认 `main`（merge dev→main→push→切回 dev）；仅 main 推送 GitHub。

### Commit Message

Conventional Commits：builder 用 `<type>: T0X T0Y - <概述>`，maintainer 用 `<type>(scope): <描述>`。类型：`feat/fix/chore/refactor/docs`。禁止空话和 AI 水印。框架与项目改动分开 commit。

## Agent 职责边界

| Agent | 作用域 | 写权限 | 典型用途 |
|---|---|---|---|
| `planner` | 宿主仓库单项目 | `<项目目录>/PLAN.md` + `<项目目录>/docs/**` | 起草项目计划，修订 PLAN |
| `builder` | 宿主仓库单项目 | 项目源码 / 测试 / `PROGRESS.md` | 实现、测试、commit、召唤 reviewer |
| `reviewer` | 宿主仓库单项目 | 仅 `<项目目录>/REVIEW.md` | 独立验证测试 / lint / 类型检查；输出 APPROVED/NEEDS-FIX/REJECTED |
| `teacher` | 宿主仓库单项目 | `<项目目录>/learning-notes/**` | 项目交付后由 builder 调用，生成学习材料 |
| `aide` | 框架文档层 | 仅 `docs/**` + `templates/**` | maintainer 的廉价 subagent（V4 Flash + reasoning）。承担可验证的审计/搜索/总结任务，maintainer 审核其输出 |
| `consultant_1` | 宿主仓库全仓库（读全库，写仅 `docs/consults/`） | 仅 `docs/consults/**` | 独立 AI 顾问（Claude Opus 4.7），仅 maintainer 在用户指令下调用，输出顾问报告 |
| `consultant_2` | 宿主仓库全仓库（读全库，写仅 `docs/consults/`） | 仅 `docs/consults/**` | 独立 AI 顾问（GPT-5.5），仅 maintainer 在用户指令下调用，输出顾问报告 |
| `consultant_3` | 宿主仓库全仓库（读全库，写仅 `docs/consults/`） | 仅 `docs/consults/**` | 独立 AI 顾问（Gemini 3.1 Pro Preview），仅 maintainer 在用户指令下调用，输出顾问报告 |
| `consultant_4` | 宿主仓库全仓库（读全库，写仅 `docs/consults/`） | 仅 `docs/consults/**` | 独立 AI 顾问（DeepSeek V4 Pro），仅 maintainer 在用户指令下调用，输出顾问报告 |
| `maintainer` | **框架仓库全层** | 项目文件需用户明确授权。其他路径 allow | 框架维护、修改配置/prompts、一致性审计、回答仓库设计问题 |
| `maintainer_flash` | 全仓库（只读） | 无（只读，仅对话输出） | 轻量版 maintainer——回答仓库结构/设计问题、代码导航、解释代码。不能修改文件或跑命令 |

**maintainer 是唯一跨层可写（framework + 项目）agent**。planner / builder / reviewer / teacher 严格聚焦宿主仓库的单项目。`maintainer_flash` 是全仓库只读——适用于需要框架知识但不需要修改的场景（可替代原 explainer 的职责）。`aide` 是 maintainer 的专用执行器，仅操作文档层，只能由 maintainer 通过 Task 工具调用。四位 `consultant_N` 是独立顾问 subagent，使用不同模型提供多视角分析；**仅 maintainer 可在用户显式指令下通过 Task 工具调用，maintainer 不得主动调用**。

**适合找 maintainer 的场景**：

- 修改 framework 自身（prompts、配置、文档）
- 回答仓库结构、设计相关问题
- 处理"非标准"需求（不属于 plan / build / review / teach 流程）
- 调用独立顾问（consultant_1/2/3/4）进行多视角分析（需用户显式指令）

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

maintainer 修改上述任一文件后，**必须在同一次 session 内**将改动同步到另一侧的所有对应文件。同步的操作步骤见 `agent-prompts/maintainer.md` Rule 2。

### 规则 3：两侧差异仅限路径格式与脱敏信息

框架侧用相对路径 + 占位符，用户侧用本机绝对路径 + 实际值。其余内容分毫不差。详见 `agent-prompts/maintainer.md`。

### 规则 4：框架内部两份 opencode.jsonc 同步

`config/` 下两份配置除 model 字段外必须一致，改一份同步另一份。

## 更新框架

`git pull` 拉取框架新版本后，`agent-prompts/` 立即生效。但如果 `opencode.jsonc` 或 `AGENTS.md` 有结构变更，需 maintainer 按规则 2 同步到 `~/.config/opencode/`。

## 本地 MCP 工具

MCP 工具描述（tool 名称、参数、用法）通过 MCP 协议自动注入给 agent，本文件中不重复列出。MCP server 的 conda 环境依赖见用户级配置 `opencode.jsonc`。

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

## 已知风险

3 种已知风险与逃生通道详见 [`docs/known-risks.md`](./docs/known-risks.md)（同 session Tab 切换 / 终端 kill 不丢 session / 内置 vs 自定义 agent）。
