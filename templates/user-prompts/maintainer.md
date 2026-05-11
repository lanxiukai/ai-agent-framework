# 用户 → maintainer 常用提示词模板

> 本模板从多个实战项目的维护操作中提炼。**用法**：切到 maintainer agent 后，复制对应模板填入具体路径和需求。
> maintainer 的职责：跨项目改文件、修 framework、回答仓库设计问题、处理非典型需求。maintainer 是唯一能读写 framework 层（agent-prompts/ / opencode.jsonc / AGENTS.md）的 agent。**注意**：maintainer 默认不写入 projects/ 下文件，需用户明确说出项目路径或"改项目文件"才动。

---

## 模板 A：写入初始提示词归档

```
将该提示词写入 templates/project-ideas/<项目名>.md
```

### 实战示例

```
将该提示词写入 templates/project-ideas/llm-gateway.md
```

---

## 模板 B：生成项目级 AGENTS.md

```
请帮我生成 `<项目目录>/AGENTS.md`。要求：

1. 这是项目级 AGENTS.md，opencode 会在 agent read/write 该目录下文件时自动加载
2. 长度精简（< 100 行），只放"项目硬约束 + 历史背景 + 关键决策"，不要复读 PLAN 或仓库根 AGENTS.md
3. 内容来源：read <项目目录>/PLAN.md（第 0 节环境契约 / 第 2 节业务边界 / 第 4 节关键规则 / 第 7 节风险 / Changelog）+ PROGRESS.md + REVIEW.md + 仓库 <FRAMEWORK-PATH>/docs/agent-rules.md 的 Bug N
4. 至少覆盖：项目身份 / 不可破坏硬约束清单 / 命令调用规则简化版 / 历史关键事件
5. 写完后给我 diff 摘要 + 建议 commit message，等我说"提交"再 git commit
```

### 实战示例

```
请帮我生成 `<项目目录>/AGENTS.md`。要求：
1. 项目级，自动加载
2. < 100 行，不重复 PLAN 或根 AGENTS.md
3. 来源：PLAN 第 0/2/4/7 节 + Changelog + PROGRESS + REVIEW
4. 覆盖：硬约束清单 / 命令规则 / 历史关键事件
5. 给 diff + commit message，等确认再提交
```

---

## 关键原则

| 原则 | 说明 |
|---|---|
| **maintainer 是"meta"角色** | 用它做跨项目、跨 framework 的事，不要用 builder/planner 代劳 |
| **改 prompts 先确认** | maintainer 的 prompt 要求 "修改 agent-prompts/*.md 必须先经用户确认"——遵守 |
| **先侦察再规划** | maintainer 启动阶段要求 "先 read/glob 扫一遍仓库全貌" |
| **写完后给 diff + commit message** | maintainer "不擅自 commit"——等你确认再执行 git 操作 |
| **改完核验** | 动了源码或测试 → 跑项目测试/lint；动了配置 → 跑语法解析；动了 prompts → 让人眼 review |
