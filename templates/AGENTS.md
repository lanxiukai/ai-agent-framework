# AGENTS.md

> 本仓库使用 AI agent 协作框架。具体配置方式和路径说明见下文。
>
> 如果没有项目特殊需求，且已配置好用户级 `~/.config/opencode/`，**可以删除此文件**——agent 仍正常工作。

## 使用方式

<!-- 根据你的实际使用方式，删掉不适用的段落 -->

### 方式 A：用户级配置（推荐个人开发者）

框架安装在 `~/.config/opencode/`，所有项目自动共享 agent 配置。本项目的 `opencode.jsonc`（如存在）会叠加到用户级配置上——只写要覆盖的 agent/字段。

```jsonc
// 示例：项目级覆盖 builder 模型
{
  "agent": {
    "builder": { "model": "openrouter/anthropic/claude-sonnet-4.6" }
  }
}
```

**框架文档路径：** 将 `<FRAMEWORK_PATH>` 替换为框架 clone 目录的绝对路径（如 `/home/you/ai-agent-framework`）。

### 方式 B：Submodule 嵌入（推荐团队协作）

框架作为 `framework/` submodule 嵌入。项目根 `opencode.jsonc` 必须存在并指向 submodule 内的 prompt 文件。详见框架 README。

**框架文档路径：** 直接用 `framework/` 前缀，如 `framework/AGENTS.md`。

---

## 框架文档

<!-- 将 <FRAMEWORK_PATH> 替换为：用户级 → 框架绝对路径，submodule → framework -->

- 仓库级约定：`<FRAMEWORK_PATH>/AGENTS.md`
- 工程法则：`<FRAMEWORK_PATH>/docs/agent-rules.md`
- Agent prompts：`<FRAMEWORK_PATH>/agent-prompts/`
- 硬件环境：`<FRAMEWORK_PATH>/docs/developer-environment.md`（首次使用需从 `<FRAMEWORK_PATH>/docs/developer-environment.template.md` 复制并填入本机信息）

## 工作流

所有项目工作通过 `maintainer` 统一入口：

1. 用户告诉 maintainer 项目需求 → maintainer 调 planner 起草 PLAN.md
2. maintainer 调 builder 逐批实现 task（每批 ≤ 3 个）
3. maintainer 调 reviewer 独立验证每批产出
4. maintainer 调 teacher 生成学习材料（可选）

## 环境契约

技术栈约束由 planner 协商后写进 `<项目目录>/PLAN.md` 第 0 节。

## 多项目共存

- 项目存放在用户确认的目录下，planner 会向用户询问路径
- 各项目独立，不交叉污染
- Submodule 场景下所有项目共享同一套 submodule 的 agent 配置

## 项目约定（按需添加）

<!-- 以下写本项目特有的要求，如：代码风格、测试阈值、禁止使用的依赖等 -->
