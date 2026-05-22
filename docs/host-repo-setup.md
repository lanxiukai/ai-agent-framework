# Submodule 宿主仓库初始化指南

> 使用 git submodule 方式从零搭建宿主仓库。5 步完成——总耗时 < 5 分钟。
>
> **如果你使用的是用户级配置（推荐个人开发者）：** 请参考 [README 方式一](../README.md#方式一用户级配置推荐个人开发者)，无需阅读本指南。用户级配置一次安装即可，不需要 submodule，也不需要在每个项目里放 `opencode.jsonc` 和 `AGENTS.md`。

---

## 前置条件

- 已安装 [opencode](https://opencode.ai)
- 已设置 `DEEPSEEK_API_KEY`（或 `OPENROUTER_API_KEY`，如果用备份配置）
- 本框架仓库地址（如 `git@github.com:lanxiukai/ai-agent-framework.git`）

---

## 步骤 1：创建仓库 + 引入框架

```bash
# 创建宿主仓库
mkdir my-project-repo && cd my-project-repo
git init
git checkout -b main

# 引入框架 submodule
git submodule add <框架仓库地址> framework/
git submodule update --init --recursive
```

---

## 步骤 2：创建宿主 AGENTS.md

opencode 只读取工作目录根部的 `AGENTS.md`——不会递归扫描子目录。你必须自己在根目录放一份。

```bash
cp framework/templates/AGENTS.md ./AGENTS.md
```

这份模板只有 22 行，唯一职责是指向框架文档并串联工作流。**不需要**把框架的完整 `AGENTS.md` 复制过来——agent 启动时会自动按指针读 `framework/AGENTS.md`。

---

## 步骤 3：创建宿主 opencode.jsonc

参照 `framework/config/opencode.jsonc` 编写，只需修改一处——所有 agent 的 `prompt` 路径加上 `framework/` 前缀：

```jsonc
{
  "agent": {
    "planner": {
      "prompt": "framework/agent-prompts/planner.md",
      "model": "deepseek/deepseek-v4-pro",
      "mode": "primary"
    },
    "builder": {
      "prompt": "framework/agent-prompts/builder.md",
      "model": "deepseek/deepseek-v4-pro",
      "mode": "primary"
    },
    "reviewer": {
      "prompt": "framework/agent-prompts/reviewer.md",
      "model": "deepseek/deepseek-v4-pro",
      "mode": "subagent"
    },
    "teacher": {
      "prompt": "framework/agent-prompts/teacher.md",
      "model": "deepseek/deepseek-v4-pro",
      "mode": "subagent"
    },
    "maintainer": {
      "prompt": "framework/agent-prompts/maintainer.md",
      "model": "deepseek/deepseek-v4-pro",
      "mode": "primary"
    },
    "maintainer_flash": {
      "prompt": "framework/agent-prompts/maintainer_flash.md",
      "model": "deepseek/deepseek-v4-flash",
      "mode": "primary"
    },
    "aide": {
      "prompt": "framework/agent-prompts/aide.md",
      "model": "<你的廉价模型>",
      "mode": "subagent"
    }
  }
}
```

> **注意**：`model` / `temperature` / `permission` 可以按宿主仓库的需要调整，框架不强制。`framework/config/opencode.openrouter.jsonc.bak` 不需要复制——它只是框架仓库的备份配置，与宿主仓库无关。

---

## 步骤 4：创建目录 + 分支

```bash
# 创建项目目录（业务代码放这里）
mkdir -p projects/

# 创建 dev 分支（日常开发用）
git checkout -b dev

# 回到 main，首次 commit
git checkout main
git add -A
git commit -m "chore: init host repo with agent framework submodule"
```

> 分支策略：`dev` 做日常开发，验证通过后 merge 到 `main` 并打 tag。**只有 `main` 推送到 GitHub**。

---

## 步骤 5：启动开发

```bash
opencode
```

启动后：
1. Tab 切到 `planner`
2. 告诉它你的项目需求 + 项目名（slug，如 `ts-link-checker`）
3. Planner 会在用户确认的目录下创建 `PLAN.md`
4. PLAN 写好后**退出 session，重新 `opencode` 选 builder**
5. Builder 按 task 顺序实现，每批 ≤ 3 个 → 跑测试 → 召唤 reviewer → 继续

---

## 初始仓库结构

```
my-project-repo/
├── .git/
├── .gitmodules              # submodule 配置
├── AGENTS.md                # 宿主仓库约定（22 行极简版）
├── opencode.jsonc           # agent 配置（prompt 路径指向 framework/）
├── framework/               # ← git submodule（本框架）
│   ├── agent-prompts/
│   ├── config/
│   │   ├── opencode.jsonc
│   │   └── opencode.openrouter.jsonc.bak
│   ├── AGENTS.md
│   └── ...
└── projects/                # 你的业务项目放这里
    └── <slug>/
        ├── PLAN.md
        ├── PROGRESS.md
        └── REVIEW.md
```

---

## 后续维护

### 贡献回框架

在宿主仓库中开发时，如果你发现或总结了以下内容，**直接修改 framework/ submodule 中的对应文件**，而非留在宿主仓库：

| 内容类型 | 示例 | 写到 framework/ 的哪里 |
|---|---|---|
| 通用工程法则（从项目中学到的） | "先写测试再改代码"、"配置层硬封 > prompt 软约束" | `<FRAMEWORK-PATH>/docs/agent-rules.md`（提炼为法则） |
| agent prompt 改进建议 | 某类 task 反复被 reviewer 打回，prompt 需要更明确 | `agent-prompts/<agent>.md`（在 submodule 中编辑） |
| 新的模板需求 | 发现需要一个新的 PROGRESS 模板变体 | `templates/` |

**判断标准**：这条知识换个宿主仓库还有用吗？

- 有用 → 放到 `framework/` submodule 里，所有宿主仓库受益
- 只和当前项目代码有关（如"mdwc 的 strip_frontmatter 有 bug"）→ 留在宿主仓库

> **脱敏提醒**：框架仓库开源。如果你在 submodule 中修改了 `config/opencode.jsonc`、`config/opencode.openrouter.jsonc.bak` 或 `AGENTS.md`，提交前确认这些文件**不含**本机绝对路径（`/home/xxx/...`）、conda 环境路径、git remote URL、私人密钥等敏感信息。prompt 路径用相对格式（`{file:./agent-prompts/...}`），MCP `command` 路径用占位符（`<YOUR-CONDA-ENV-PYTHON>`）。

修改 submodule 后，在 framework/ 目录内正常 git 操作：

```bash
cd framework/
git checkout -b dev    # 在 submodule 的 dev 分支上改
# ... 改文件 ...
git add -A
git commit -m "docs: add opencode session recovery tip from mdwc project"
git push origin dev    # 推到框架仓库（提 PR 或直接合）
cd ..
git add framework/     # 宿主仓库记录新的 submodule commit
git commit -m "chore: bump framework submodule"
```

这也是**教训反馈环**的具体操作——每个宿主仓库的项目实战 → 提炼通用法则 → 写回框架，让所有仓库一起变聪明。

### 框架升级

当框架仓库发布新版本时，在宿主仓库中升级到最新：

```bash
# 1. 在 submodule 内拉最新代码
cd framework/
git pull origin main

# 2. 回到宿主仓库，记录新的 submodule commit
cd ..
git add framework/
git commit -m "chore: update agent framework to vX.Y.Z"
```

> **注意**：升级用的是 `git pull`，**不是** `git submodule update`。
> - `git pull` → 拉到远程最新，跟上框架演进
> - `git submodule update` → checkout 回宿主仓库记录的旧 commit（恢复，不是升级）

`git submodule update --init --recursive` 只在一种场景用——**首次 `git clone` 别人的仓库后**，子目录是空的，用它来拉取 submodule 内容：

```bash
git clone <宿主仓库地址>
cd <repo>
git submodule update --init --recursive   # 填充 framework/ 目录
```

### 新增第二个项目

```bash
opencode
# Tab → planner
# 说："我想做 XXX，项目名叫 second-project"
# Planner 自动在 projects/second-project/ 下创建 PLAN.md
```

---

## 相关文档

| 文档 | 内容 |
|---|---|
| [`AGENTS.md`](../AGENTS.md) | 仓库级开发约定（所有 agent 启动时自动加载） |
| [`agent-prompts/`](../agent-prompts/) | 各 agent 的 system prompt |
| [`../docs/agent-rules.md`](../docs/agent-rules.md) | 工程法则 |
| [`templates/project-ideas/TEMPLATE.md`](../templates/project-ideas/TEMPLATE.md) | 项目 idea 模板 |
| [`templates/AGENTS.md`](../templates/AGENTS.md) | 宿主仓库 AGENTS.md 模板 |
