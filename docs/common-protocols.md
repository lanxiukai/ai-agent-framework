# 通用协议

> Agent 启动时必须读取本文件。以下协议是所有 agent 共享的行为规范。

## 网络 / Provider 故障应对

LLM provider 链路有时会出现 `502 / provider_unavailable / Tool execution aborted` 这类瞬时故障，常见于 `write` / `edit` 工具调用切换的瞬间。

1. **首次失败**：直接重试一次（70-80% 概率成功）。不要重新分析或重新规划——内容已在 context 里。
2. **连续失败 ≥ 3 次**：不再硬重试。将待写入内容用代码块（` ```<lang> ... ``` `）输出到对话，让用户人工保存；或建议用户切到 opencode 内置 `Build` agent（如该 agent 使用不同 provider）协助落盘——内容仍是你的产出，Build 只做"执行器"。
3. **绝不降级**：不因网络故障简化内容、跳过文件或降低质量标准。
