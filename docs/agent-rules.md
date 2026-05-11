# Agent Rules

> Agent 启动时必须读取本文件。这些是从实战中提炼的硬约束——违反会浪费所有 agent 的时间。

---

### 1. 配置层硬封 > Prompt 层软约束

- `permission.edit` 路径白名单是配置层硬约束，agent 无法突破
- prompt 说"只能写 PLAN.md"是软约束，LLM 可能违反
- **能用配置层封死的，永远不要靠 prompt 自觉**
- 但 prompt 层依然必要：告诉 LLM **为什么**存在这个限制，边界场景下能做更聪明的判断

### 2. Plan-Issue vs Bug-Issue

| 类型 | 触发条件 | 处理方 |
|---|---|---|
| **Bug-Issue** | 实现细节出错（算法、API 用错） | builder 自己修，最多重试 3 次 |
| **Plan-Issue** | 任务定义不清 / 验收矛盾 / 技术选型不可行 | 切回 planner |

**关键判定**：如果 builder 反复修 must-fix 都过不了 reviewer，且 reviewer 反馈一致指向"PLAN 本身有问题"——就是 Plan-Issue，不要硬刚。

### 3. Reviewer 必须独立复跑，不信任 builder 的输出

- reviewer 必须自行跑测试 / 类型检查 / lint，**不能轻信** builder 给的测试结果
- builder 可能无意间用了宽断言（`>=`）、测试被框架静默跳过、环境不一致等
- 这是这套框架最重要的设计决定之一

### 4. 简单修复不需要走完整三角流程

| 改动规模 | 路径 |
|---|---|
| < 50 LOC，已知方案，无设计取舍 | builder/maintainer 直接修，跑测试，commit |
| 需要新增 task 或修改验收标准 | planner → builder → reviewer 完整三角 |

### 5. Task 颗粒度与验收标准

- 单 task < 30 分钟、< 100 LOC
- 验收标准必须**可机械验证**（具体命令 / 文件存在性 / 返回值断言）
- 拒绝主观描述（"代码质量良好"）和宽断言（`>=`）——能用 `==` 等值就用 `==`
- 任务依赖必须是 DAG，无循环

### 6. 环境 / 工具链验证前置

- 在写业务代码前，先用一条最简命令验证工具与运行时兼容
- 环境契约写在 `PLAN.md` 第 0 节——builder/reviewer 必须严格遵守"命令调用规则"
- 工具链兼容性问题早暴露比晚发现省数小时回滚
