# 用户 → planner 常用提示词模板

> 本模板从多个实战项目的起手 prompt 中提炼。**用法**：写新项目 prompt 时复制对应模板，填入你的具体信息。
> planner 的职责：读需求 → 写 PLAN.md（含环境契约、任务拆解、验收标准），不写代码。
>
> **以下模板以 Python 项目为例；若你的项目用其他语言（Rust / TypeScript / Go 等），参考 `templates/project-ideas/` 下的起手 prompt 替换工具链、依赖管理和 CI 命令。**

---

## 模板 A：从零启动新项目

```
我想做一个 <语言> <类型> 叫 <项目名>（<简短描述>）。

核心功能：
- <功能 1>
- <功能 2>
- <功能 3>

支持选项：
- <选项 1>
- <选项 2>

<特殊处理逻辑>（如 CJK 策略、多语言支持等）

环境约束（请如实写进 PLAN 第 0 节）：
- 语言版本：...
- 虚拟环境：...（路径绝对化）
- 命令调用规则：所有 python / pip / pytest 一律用绝对路径
- 禁止事项：裸 python、conda activate、pip install --user、sudo
- 自检命令：...（含预期输出格式）

依赖管理 / 打包（重要：只用一种 manifest）：
- 只用 pyproject.toml（PEP 621），不创建 requirements.txt
- 依赖分组：[project.dependencies] 运行时 + [project.optional-dependencies] dev 组
- [project.scripts] 注册 <cli-name> = "<模块>.cli:main"
- 安装方式：<pip 绝对路径> install -e ".[dev]"

测试 / 类型检查 / Lint：
- 测试命令：...
- 类型检查命令：...
- Lint 命令：...

业务边界（明确不做）：
- 不做：<列举>
- 做：<列举>

请按 PLAN.md 模板写出完整 plan：
- 任务粒度细到 6-8 个 task，每个 < 30 分钟可独立完成
- 每个 task 的验收标准必须可机械验证（具体命令 / 断言 / 文件存在性）
- 风险章节至少 2 条
- 状态：如无异议置为 approved，否则置为 drafting 列出疑问
```

### 实战示例

见 `templates/project-ideas/` 下的起手 prompt 存档。
每个项目跑完后建议将其起手 prompt 保存到该目录，格式参照其中的 TEMPLATE 模板。

---

## 模板 B：修订已有 PLAN（Plan-Issue 回退后）

```
<描述 Plan-Issue 矛盾点>。用户决策：选方案 X（<简述>）。

请按你的"计划修订时保留旧版"规则修订 PLAN.md：
1. <修订点 1>
2. <修订点 2>
3. <修订点 3>
4. 把废弃的旧验收标准块整段移到末尾的 ## Changelog 区，注明日期与原因
5. 修完后状态保持 approved，告诉我可以切回 builder 继续
```

### 实战示例

```
矛盾点：T04 验收写 4 个 fixture，实际目录有 5 个 → reviewer REJECTED。
用户选方案 2（分子目录）：基础 4 fixture 移到 tests/fixtures/baseline/。

请修订 PLAN.md：
1. 第 5 节目录结构：新增 baseline/ 子目录
2. T04/T06 验收标准路径更新
3. T06 拆为两条精确断言（baseline==4 + fixtures==5）
4. 旧验收标准移到 Changelog 区，标注 DEPRECATED + 原因
```

---

## 关键原则

| 原则 | 说明 |
|---|---|
| **环境契约要绝对化** | 解释器路径、pip 路径、自检命令全都写绝对路径，禁止裸命令 |
| **CJK / 模糊算法要写死** | 不要写"一字一词"就完事——写 Unicode 区间表格 + 分词伪代码 |
| **业务边界写"不做什么"** | 比"做什么"更能防止 builder 过度发挥 |
| **验收标准要可机械验证** | 命令可跑、exit code 可查、文件可 `ls`——拒绝主观描述 |
| **风险章要真思考** | 不要凑数写"可能出 bug"这种废话——每条风险配一条具体应对 |
