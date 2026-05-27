# 快速开始（详细指南）

> 本文档为 README 快速开始章节的详细版。先看 [README.md](../README.md#快速开始) 了解概览，需要细节时回到这里。

---

## 前置条件

- 安装 [opencode](https://opencode.ai)
- 设置 `DEEPSEEK_API_KEY`：

  ```bash
  export DEEPSEEK_API_KEY=sk-...
  ```

- 如需切回 OpenRouter / Claude / GPT：
  - 用户级：`cd ~/.config/opencode && cp opencode.openrouter.jsonc.bak opencode.jsonc`
  - 项目级/submodule：在项目根或 `framework/` 下 `cp opencode.openrouter.jsonc.bak opencode.jsonc`
  - 同时 export `OPENROUTER_API_KEY`

---

## 方式一：用户级配置（推荐个人开发者）

OpenCode 支持从 `~/.config/opencode/` 加载全局配置——一次安装，所有项目零配置启动。

```bash
# 1. Clone 框架到固定路径
git clone <本仓库地址> ~/ai-agent-framework

# 2. 将 opencode.jsonc 复制到用户级目录
#    并替换所有 prompt 路径为框架的绝对路径
mkdir -p ~/.config/opencode
sed 's|{file:./agent-prompts/|{file:'"$HOME"'/ai-agent-framework/agent-prompts/|g' \
  ~/ai-agent-framework/config/opencode.jsonc > ~/.config/opencode/opencode.jsonc

# 同理生成备份配置
sed 's|{file:./agent-prompts/|{file:'"$HOME"'/ai-agent-framework/agent-prompts/|g' \
  ~/ai-agent-framework/config/opencode.openrouter.jsonc.bak > ~/.config/opencode/opencode.openrouter.jsonc.bak

# 3. 复制用户级 AGENTS.md（可选但推荐）
cp ~/ai-agent-framework/templates/AGENTS.md ~/.config/opencode/AGENTS.md

# 4. 启动——任意项目目录
cd ~/my-project
opencode
```

### 配置叠加规则

项目级 `opencode.jsonc` **合并叠加**到用户级配置上——不是替换。你写的节会覆盖同名节，没写的保持原样。

以下 4 个场景覆盖常见的项目级修改需求：

#### ① 修改某个 agent 的模型或权限（其余继承）

```jsonc
{
  "agent": {
    "builder": {
      "model": "openrouter/anthropic/claude-sonnet-4.6",
      "maxTokens": 131072
    },
    "reviewer": {
      "temperature": 0.3
    }
  }
}
```

没写的 agent（planner/teacher/maintainer/maintainer_flash/aide）和没写的字段（builder 的 prompt/permission）全部从用户级继承。

#### ② 禁用某个 agent

```jsonc
{
  "agent": {
    "teacher": { "disable": true },
    "aide":    { "disable": true }
  }
}
```

#### ③ 新增自定义 agent（追加到用户级 agent 列表之后）

```jsonc
{
  "agent": {
    "my-deployer": {
      "mode": "subagent",
      "model": "deepseek/deepseek-v4-pro",
      "prompt": "{file:./deployer-prompt.md}",
      "permission": {
        "bash": "allow",
        "edit": { "*": "allow" }
      }
    }
  }
}
```

#### ④ 组合使用（改模型 + 禁用 + 新增）

```jsonc
{
  "agent": {
    "build": { "disable": true },
    "plan":  { "disable": true },

    "builder": {
      "model": "openrouter/anthropic/claude-sonnet-4.6"
    },
    "teacher": { "disable": true },

    "my-helper": {
      "mode": "subagent",
      "model": "deepseek/deepseek-v4-flash",
      "prompt": "{file:./helper-prompt.md}"
    }
  }
}
```

> 合并行为与 `Object.assign(userConfig, projectConfig)` 一致——项目级写的键覆盖用户级同名键，不写的保留。

---

## 方式二：Submodule 嵌入宿主仓库（推荐团队协作）

将框架作为 git submodule 引入，锁定版本——所有协作者使用相同的 agent 行为。

```bash
# 在你的业务仓库根目录
git submodule add <本框架仓库地址> framework/
git submodule update --init --recursive
```

opencode 启动时会读取宿主仓库根目录的 `AGENTS.md` 和 `opencode.jsonc`（指向 `framework/agent-prompts/` 下的 prompt 文件）。你的项目代码在宿主仓库中开发，存放路径由 planner 与用户协商确认。

> **注意**：子目录（包括 submodule）内的 `AGENTS.md` 和 `opencode.jsonc` **不会被 opencode 自动加载**。宿主仓库必须在根目录自己放置这两个文件。AGENTS.md 模板：[`../templates/AGENTS.md`](../templates/AGENTS.md)

### 宿主仓库 opencode.jsonc 配置

宿主仓库的 `opencode.jsonc` 需要将每个 agent 的 `prompt` 字段指向 submodule 内的 prompt 文件。参照框架仓库 `config/opencode.jsonc` 编写，只需修改两处：

**1. 所有 `prompt` 路径加上 `framework/` 前缀：**

```jsonc
{
  "agent": {
    "planner":       { "prompt": "framework/agent-prompts/planner.md" },
    "builder":       { "prompt": "framework/agent-prompts/builder.md" },
    "reviewer":      { "prompt": "framework/agent-prompts/reviewer.md" },
    "teacher":       { "prompt": "framework/agent-prompts/teacher.md" },
    "maintainer":    { "prompt": "framework/agent-prompts/maintainer.md" },
    "maintainer_flash": { "prompt": "framework/agent-prompts/maintainer_flash.md" },
    "aide":          { "prompt": "framework/agent-prompts/aide.md" }
    // model / temperature / tools / permission 按需调整
  }
}
```

**2. 仅保留 `opencode.jsonc`：** 不需要复制 `opencode.openrouter.jsonc.bak`——备份配置由框架仓库维护，宿主仓库只需一份活跃配置。

> 更详细的 submodule 宿主仓库初始化指南：[`host-repo-setup.md`](./host-repo-setup.md)

---

## 方式三：直接克隆（框架维护）

```bash
git clone <本仓库地址>
cd ai-agent-framework
opencode
```

此方式用于修改 agent prompts、配置文件等框架层资产。不要在此仓库中开发业务项目。

---

## 框架更新

| 使用方式 | 更新命令 | 生效范围 | 注意事项 |
|---|---|---|---|
| 用户级配置 | `cd ~/ai-agent-framework && git pull` | **所有项目**立即生效 | prompt 使用绝对路径直接读框架文件，`git pull` + 重启 opencode 即生效；`opencode.jsonc` 和 `AGENTS.md` 在 `~/.config/opencode/` 下是独立副本（非 symlink），框架仓库中这些文件有结构变更时需 maintainer 按同步约定手动同步 |
| Submodule | `cd framework/ && git pull origin main && cd .. && git add framework/ && git commit -m "chore: update framework"` | **当前宿主仓库** | 每个宿主仓库独立更新；可 pin 不同版本 |

---

## 启动流程

opencode 启动后，Tab 切到 `maintainer`，告诉它你想做什么——maintainer 会调度 planner 写出 `PLAN.md`，然后调度 builder 逐批实现 task。每批完成后 maintainer 会调 reviewer 独立验证，确保质量后再推进下一批。
