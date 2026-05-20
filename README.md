# OpenCode 多 Agent 协作开发框架

> Agent 模板与配置仓库。可通过**用户级配置**全局安装（一次配置，所有项目零设置）或 **git submodule** 嵌入到任意业务仓库，提供统一的 AI agent 协作开发能力。

---

## 这是什么

一个**纯框架仓库**——只包含 agent system prompts、配置文件和工程文档，不包含任何业务项目代码。支持两种使用方式：

- **用户级配置**：将配置安装到 `~/.config/opencode/`，所有项目自动共享，零配置启动
- **Git Submodule**：嵌入到业务仓库（宿主仓库），适合需要版本锁定的团队协作

核心流程是 `planner` → `builder` → `reviewer` 三角协作：

```
                  ┌──────────────┐
   你（用户） ──→  │   planner    │  写 PLAN.md（项目契约）
                  │  (primary)   │  ← 不写代码、不跑命令
                  └──────┬───────┘
                         │ Tab 切换
                         ↓
                  ┌──────────────┐
                  │   builder    │  按 PLAN 实现 task，写 PROGRESS.md
                  │  (primary)   │  → 每批结束召唤 reviewer
                  └──────┬───────┘
                         │ Task tool 调用
                         ↓
                  ┌──────────────┐
                  │   reviewer   │  独立验证，写 REVIEW.md
                  │  (subagent)  │  → APPROVED / NEEDS-FIX / REJECTED
                  └──────────────┘
```

辅助 agent（按需调用）：

- **`teacher`**（subagent）：项目交付后自动生成学习材料到 `<项目目录>/learning-notes/`
- **`maintainer`**（primary）：框架维护——修改配置与 prompts、一致性审计
- **`maintainer_flash`**（primary）：轻量只读版 maintainer——回答仓库结构/设计问题、代码导航、解释代码
- **`aide`**（subagent）：maintainer 的廉价执行器（V4 Flash + reasoning），承担可验证的审计/搜索/总结任务

| 文件 | 由谁写 | 谁来读 | 作用 |
|---|---|---|---|
| `<项目目录>/PLAN.md` | planner | builder, reviewer, teacher | 项目契约（需求 / 技术选型 / task / 验收标准） |
| `<项目目录>/PROGRESS.md` | builder | planner（回退）, teacher | 增量工作日志（done / blocked / plan-issue） |
| `<项目目录>/REVIEW.md` | reviewer | builder, teacher | 当批次的审查结论（每轮覆盖写入） |
| `<项目目录>/learning-notes/**` | teacher | 学习者 | 学习材料（按需生成） |

完整 agent prompt：`agent-prompts/{planner,builder,reviewer,teacher,maintainer,maintainer_flash,aide}.md`

---

## 适用场景

- 需要多 agent 分工协作、有明确验收标准的中小型项目
- 多个仓库希望共享同一套 agent 协作约定和配置
- 想理解"如何让多个 LLM agent 可控地协作"，而非"一键生成项目"

不适用：一次性脚本、已有成熟 CI / review 流程的大型项目。

---

## 框架仓库目录

```text
.                                     # 框架根（prompts + 文档 + 配置模板）
├── AGENTS.md                         # 仓库级开发约定，所有 agent 启动时自动加载
├── config/
│   ├── opencode.jsonc                # 配置源模板（DeepSeek 直连）
│   └── opencode.openrouter.jsonc.bak # 备份配置源模板（OpenRouter / Claude / GPT）
├── README.md                         # 本文件
├── LICENSE                           # MIT License
├── agent-prompts/
│   ├── planner.md / builder.md / reviewer.md / teacher.md / maintainer.md / maintainer_flash.md / aide.md
├── docs/
│   ├── agent-rules.md               # 工程法则（agent 必读的 6 条硬约束）
│   ├── quickstart.md                 # 快速开始详细指南（3 种方式 + 配置叠加示例）
│   ├── known-risks.md                # 已知风险与逃生通道
│   ├── developer-environment.md     # 硬件环境档案
│   ├── developer-environment.template.md  # 硬件环境模板
│   ├── host-repo-setup.md           # Submodule 宿主仓库初始化指南
│   ├── private/                     # 个人文件（不被 git 跟踪）
│   └── .session-log.md              # 框架 session 操作日志
├── templates/
│   ├── README.md                     # 模板目录说明
│   ├── .session-log.md              # 宿主仓库 session 日志模板
│   ├── AGENTS.md / PROGRESS.md / REVIEW.md   # 工作产物模板
│   ├── project-ideas/                # 项目 idea 池
│   ├── user-prompts/                 # 用户 → agent 提示词模板
│   └── gitignore-python.txt / gitignore-node.txt / gitignore-rust.txt
└── .editorconfig / .gitattributes / .gitignore
```

> **注意**：框架仓库不包含项目代码。业务项目代码位于宿主仓库中，存放路径由用户确认。

---

## 快速开始

### 前置条件

- 安装 [opencode](https://opencode.ai)
- 设置 `DEEPSEEK_API_KEY`：`export DEEPSEEK_API_KEY=sk-...`

### 方式一：用户级配置（推荐个人开发者）

```bash
git clone <本仓库地址> ~/ai-agent-framework
mkdir -p ~/.config/opencode
sed 's|{file:./agent-prompts/|{file:'"$HOME"'/ai-agent-framework/agent-prompts/|g' \
  ~/ai-agent-framework/config/opencode.jsonc > ~/.config/opencode/opencode.jsonc
sed 's|{file:./agent-prompts/|{file:'"$HOME"'/ai-agent-framework/agent-prompts/|g' \
  ~/ai-agent-framework/config/opencode.openrouter.jsonc.bak > ~/.config/opencode/opencode.openrouter.jsonc.bak
cp ~/ai-agent-framework/templates/AGENTS.md ~/.config/opencode/AGENTS.md
cd ~/my-project && opencode
```

### 方式二：Submodule 嵌入宿主仓库（推荐团队协作）

```bash
# 在你的业务仓库根目录
git submodule add <本框架仓库地址> framework/
git submodule update --init --recursive
```

宿主仓库根目录需放置 `AGENTS.md` 和 `opencode.jsonc`（prompt 路径加 `framework/` 前缀）。详见 [`docs/host-repo-setup.md`](./docs/host-repo-setup.md)。

### 方式三：直接克隆（框架维护）

```bash
git clone <本仓库地址> && cd ai-agent-framework && opencode
```

用于修改 agent prompts、配置文件等框架层资产。

> 📖 **详细版**：配置叠加规则（4 个场景）、框架更新机制、完整启动流程 → [`docs/quickstart.md`](./docs/quickstart.md)

---

## 本地 MCP 工具（可选）

配套仓库 [mcp-tools](https://github.com/lanxiukai/mcp-tools)（`tag >= v0.2.1`）提供基于本地模型的 MCP 工具，包括语音转文字（Qwen3-ASR）、文档 OCR 解析（GLM-OCR）、图片描述（QwenVision）。这些工具为可选补充，非必需品：

- **适用**：纯文本模型（如 DeepSeek V3）需要多模态能力，或离线 / 隐私敏感的本地处理场景
- **不必须**：多模态模型（GPT-4V、Claude Sonnet、Gemini）可直接使用原生视觉/音频能力处理同类任务

详见 [AGENTS.md § 本地 MCP 工具](./AGENTS.md#本地-mcp-工具)。

---

## 项目 idea 池

本框架提供了项目 idea 模板 [`templates/project-ideas/TEMPLATE.md`](./templates/project-ideas/TEMPLATE.md)，可按此模板创建自己的项目 prompt。模板覆盖：学习目标、功能说明、前置知识、环境搭建、验证标准——**填完即用**。

---

## 调整 Agent 行为

- 改 prompt 风格 / 工作流细节 → 编辑 `agent-prompts/*.md`
- 改 model / temperature / 工具权限 → 编辑 `opencode.jsonc`（用户级配置在 `~/.config/opencode/opencode.jsonc`）
- 改仓库级开发约定 → 编辑 `AGENTS.md`
- 框架更新后用户级配置如何生效 → 见 [`docs/quickstart.md#框架更新`](./docs/quickstart.md#框架更新)

> 懒人方案：召唤 `maintainer` agent 来做——它有全仓库权限。maintainer 会在改 prompts 前口述意图并等你确认。

---

## 设计原则

- **planner 不写代码、不跑命令**：只写 PLAN.md。代码层面的细节属于 builder
- **环境契约写在 PLAN.md 第 0 节**：框架不绑定任何语言，技术栈按项目决定
- **reviewer 独立验证**：必须自行重跑测试 / 类型检查 / lint——不信任 builder 的输出
- **三态结论**：APPROVED / NEEDS-FIX / REJECTED，不留灰色地带
- **配置层硬封 > prompt 层软约束**：能用 permission 封死的，不靠 prompt"劝"
- **教训反馈环**：每个项目的 bug → 提炼通用法则 → 写回 agent prompt 和 `docs/agent-rules.md`

> 工程法则（agent 必读）：[`docs/agent-rules.md`](./docs/agent-rules.md)

---

## 分支策略

| 分支 | 用途 |
|---|---|
| `dev` | 日常开发，所有改动先合入此分支 |
| `main` | 稳定发布版本，从 `dev` 合并已验证的改动 |

> 开发流程：在 `dev` 上迭代，测试通过后 merge 到 `main`，打 tag 标记版本。只有 `main` 分支推送到 GitHub——`dev` 仅本地开发，不推送远端。

---

## 已知风险

详见 [`docs/known-risks.md`](./docs/known-risks.md)。

---

## 同步约定

框架 `config/opencode.jsonc`、`.bak` 和根目录 `AGENTS.md` 与 `~/.config/opencode/` 下同名文件互为模板/副本——**改一侧必须立即同步另一侧**。仅 maintainer 有权修改。详见 [`AGENTS.md` 同步约定](./AGENTS.md#同步约定硬性规则)。

---

## 版本更新

| 版本 | 日期 | 主要变更 |
|---|---|---|
| v0.3.6 | 2026-05-20 | 新增 consultant_4 独立 AI 顾问 subagent（DeepSeek V4 Pro），四顾问体系完善；maintainer 编辑权限简化：移除 `projects/**` 硬 deny，项目文件写保护改为纯 prompt 层约束 |\n| v0.3.5 | 2026-05-15 | 新增三位独立 AI 顾问 subagent（consultant_1/2/3），分别使用 Claude Opus 4.7 / GPT-5.5 / Gemini 3.1 Pro Preview；maintainer 仅用户显式指令下可调用 |
| v0.3.4 | 2026-05-13 | ASR 超时 3min→30min，全 agent `transcribe_podcast` 权限；README mcp-tools URL 修正 + 版本依赖标注 >= v0.2.1 |
| v0.3.3 | 2026-05-12 | MCP config fix + maintainer 修漏（tag 流程强制 README 前置更新，config/README 合并 commit）|
| v0.3.2 | 2026-05-12 | Token 精简：AGENTS.md -39%（删 MCP 重复文档/压缩同步与风险/合并 Git 约定），builder.md 删内嵌 PROGRESS.md 模板，4 agent 网络故障应对提取到 docs/common-protocols.md；每 session 启动省 ~1,800 tokens |
| v0.3.1 | 2026-05-11 | README 新增「本地 MCP 工具」小节，AGENTS.md 补充 MCP 工具可选性说明 |
| v0.3.0 | 2026-05-11 | 同步约定重构（4 条硬规则 + 3 步操作流程）；框架开源脱敏（敏感路径占位符化、新增 private/ 个人文件管理）；AGENTS.md 全面同步（分支纪律、agent 边界、版本管理、同步约定） |

---

## License

MIT — 见 [`LICENSE`](./LICENSE)。

`agent-prompts/` 下的 agent system prompts、`templates/` 下的脚手架文件均受 MIT 保护。可自由使用、修改、分发——保留版权声明即可。
