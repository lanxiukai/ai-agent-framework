# Review Report

> **模板说明**：reviewer 覆盖式写入，每批次一份。必须严格遵守下方结构。
> 来源：`agent-prompts/reviewer.md` 第 41-95 行。

> 批次：T0X、T0Y、T0Z
> 审阅时间：<YYYY-MM-DD HH:MM>
> 审阅者：reviewer
> 总体结论：**APPROVED** | **NEEDS-FIX** | **REJECTED**

## 1. 验收标准核对
| Task | 标准 | 状态 | 证据 |
|---|---|---|---|
| T01 | `pytest tests/test_foo.py -v` 通过 | ✅ PASS | 见下方测试输出 |
| T01 | 空输入返回 `[]` | ❌ FAIL | `src/foo.py:42` 抛 TypeError 而非返回空列表 |
| T02 | … | … | … |

## 2. 客观验证结果
```
$ <PLAN 第 0 节"自检命令" —— 证明你在正确的环境里>
…实际输出（含解释器/工具链版本、路径标识等）…

$ <PLAN 第 0 节"测试命令">
…测试输出摘要…

$ <PLAN 第 0 节"类型检查 / lint 命令">
…lint / 类型检查输出摘要…
```

## 3. Must-Fix（必须修复才能进入下一批）
按严重度从高到低排列。每条必须包含：**位置**、**问题**、**影响**、**建议改法**。

### MF-01: <短标题>
> 以下示例假设 Python 项目；按 PLAN 第 0/3 节相应替换路径和语法。
- 位置：`src/foo.py:42-58`
- 问题：函数未处理空列表输入，会抛 TypeError
- 影响：违反 T01 验收标准；调用方无法区分"空结果"与"错误"
- 建议改法：在函数开头加 `if not items: return []`，并在 `tests/test_foo.py` 增加对应用例

### MF-02: …

## 4. Nice-to-Have（建议但不阻塞）
### NH-01: <短标题>
- 位置：…
- 建议：…
- 理由：…

## 5. 安全 / 性能 / 可维护性观察
（如无重大发现写"无"）

## 6. 给 maintainer 的下一步指令
1. NEEDS-FIX 时：请将 must-fix 列表转达 builder，由 builder 修复后重新提交
2. APPROVED 时：builder 可进入下一批 task
3. REJECTED 时：需要 planner 介入或用户决策，**不要**让 builder 继续强行推进
