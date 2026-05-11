# 已知风险与逃生通道

> 本文档原为 README "已知风险与逃生通道"章节的详细版。AGENTS.md 中的同节是 agent 启动时加载的版本，此处为人类可读的详细说明。

---

## 1. OpenRouter → Anthropic 路由 502 错误

> 📉 所有 agent 已改用 DeepSeek 直连——此风险大幅降低。仅在切回 `opencode.openrouter.jsonc.bak` 时相关。

**现象**：planner 调用 `write` 工具时收到 `502 Network connection lost`。应对（轻→重）：重试 1-3 次 → 用 opencode 内置 Build agent 协助写盘 → 对话输出 + 人工保存。

## 2. TUI 状态栏 reasoning effort 显示可能不准

opencode 已知 bug（[#17588](https://github.com/anomalyco/opencode/issues/17588)）。**不影响实际请求**——reasoning 配置生效，只是状态栏显示有问题。

## 3. 内置 Build / Plan agent ≠ 自定义 builder / planner

opencode TUI 同时存在两套 agent。内置的不受本仓库约束——可以在工作流死锁时作为逃生通道。

## 4. 终端被 kill 不会丢 session

opencode session 持久化到 `~/.local/share/opencode/`。`opencode -c` 重新连接最近一次 session。

## 5. 同 session Tab 切换可能不刷新 system prompt

**应对**：每个 agent 一个 session。切换 agent 时退出重新 `opencode` 进入，确保 system prompt 正确加载。
