# PROGRESS Log

> **模板说明**：首次创建时按此建好五个小节（内容为空也要保留标题）。
> 条目每行格式统一：`<时间戳> | <Task ID> | <分类> | <一句话>`。
> `<分类>` 取值：`done` / `bug` / `plan` / `dep` 等。
>
> 来源：`agent-prompts/builder.md` 第 55-79 行。

## Done
（每完成一个 task 追加一行）
YYYY-MM-DD HH:MM | T01 | done | 实现 <module>.<function> + 单测

## Blocked
（被 bug / 环境 / 外部依赖卡住的事项）
YYYY-MM-DD HH:MM | T03 | bug | 测试框架 fixture / mock / setup 报错，已尝试 X、Y、Z 都不行，求助

## Plan-Issue
（你判断 PLAN.md 本身有缺陷的事项——会触发计划层面的回退）
YYYY-MM-DD HH:MM | T05 | plan | 验收标准"返回 utf-8 字符串"与"签名 → bytes"矛盾

## Need-Approval
（请求用户批准的事项，例如新依赖、范围变更）
YYYY-MM-DD HH:MM | T02 | dep | 需引入新依赖 <package> <version>，原因：…

## Review-Round
（记录每轮 review 的结论与关键发现）
| 轮次 | Reviewer 批次 | 结论 | 关键发现 |
|---|---|---|---|
| 1 | T01-T03 | NEEDS-FIX | MF-01: … |
