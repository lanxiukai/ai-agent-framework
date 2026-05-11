# 用户 → builder 常用提示词模板

> 本模板从 mdwc / ts-link-checker / json-schema-validator 三个历史验证项目（仓库更名前）的 builder 指令中提炼。**用法**：切到 builder agent 后，复制对应模板填入任务编号和具体指令。
> builder 的职责：按 PLAN.md 写代码、跑测试、写 PROGRESS.md、每批结束召唤 reviewer。
>
> **以下模板以 Python 项目为例；若你的项目用其他语言，将 pytest/mypy/ruff 替换为对应工具的 CI 命令（如 cargo test/clippy、tsc/eslint/vitest 等）。**

---

## 模板 A：启动新批次（标准）

```
PLAN.md 已 approved。按你的"启动阶段"流程开始工作，本轮处理 T<X>-T<Y>（默认每批 ≤ 3 个 task）。开工前先把环境自检的输出贴出来给我看。
```

### 实战示例

```
PLAN.md 已 approved。按你的"启动阶段"流程开始工作，本轮处理 T01-T03。
开工前先把环境自检的输出贴出来给我看。
```

---

## 模板 B：继续下一批（上一批已 APPROVED）

```
继续下一批。按启动流程自检后进入 T<X>-T<Y>。
```

---

## 模板 C：PLAN 修订后继续（Plan-Issue 已解决）

```
PLAN.md 已由 planner 修订完成，状态仍为 approved。Changelog 区已记录 N 条 DEPRECATED。

请按 PLAN 修订后的内容完成本批次（T<X>-T<Y>）的剩余工作：

1. read 最新 PLAN.md（必做，特别是第 <具体节号>）
2. git mv <旧路径> → <新路径>（<哪些不动>）
3. 更新 <文件>：<具体改动>
4. 更新 <测试文件>：<具体改动>
5. 跑完整 CI 四连（pytest + mypy + ruff check + ruff format --check），全绿才进 commit
6. git add -A && git commit -m "<commit message>"
7. 召唤 @reviewer 复审本批次（带上之前的 baseline commit + 当前 HEAD）
```

### 实战示例（mdwc fixture rebase）

```
PLAN.md 已由 planner 修订完成。Changelog 区已记录两条 DEPRECATED。

请完成本批次（T04-T06）的剩余工作：

1. read 最新 PLAN.md（必做，特别是第 5 节目录结构 + T04/T06 验收标准）
2. git mv tests/fixtures/{empty,ascii,cjk,mixed}.md → tests/fixtures/baseline/
3. 更新 tests/conftest.py：新增 baseline_dir fixture
4. 更新 tests/test_discover.py：路径改 baseline/ + 等值断言（==）
5. 更新 tests/test_cli.py：路径改 baseline/ + 拆两条精确断言（baseline==4, fixtures==5）
6. 跑完整 CI 四连，全绿才进 commit
7. 召唤 @reviewer 复审
```

---

## 模板 D：最终交付（最后一个 task）

```
继续 T<X> 完成交付。重点：

1. README.md 处理：<具体方式>
   - 必须保留原有 <框架介绍/其他内容>
   - 区块至少含：安装命令、N 个典型调用示例、输出样例片段
   - 所有示例命令必须能原样复制粘贴跑通（自测）

2. 跑最终一次完整 CI 四连作为交付前验证

3. 把 PLAN.md 顶部状态从 approved 改为 done，最近更新 = <日期>

4. 把 DoD 全部勾选 [x]（含 README 示例自检）

5. PROGRESS.md 追加 T<X> done + 项目 closed

6. git add -A && git commit -m "chore: project complete"

7. 给 summary：完成 task 数、改动文件总数、最终测试统计、reviewer 结论
```

### 实战示例

```
继续 T07 完成交付。重点：

1. README 写 mdwc 区块（放在 ## License 之前）：
   - 保留原有框架介绍（grep "三角协作" 自检）
   - 至少 5 个可复制粘贴示例 + 输出片段
2. 跑最终 CI 四连
3. PLAN.md 状态改 done，DoD 全勾 [x]
4. PROGRESS.md 追加 T07 done
5. git commit
6. 给 summary
```

> 上面这段是 mdwc 项目的实际用法——替换为你的项目名、README 区块要求和示例数。

---

## 关键原则

| 原则 | 说明 |
|---|---|
| **必须让 builder 跑环境自检** | 每次新会话第一条 bash 必须是 PLAN 第 0 节的自检命令——builder prompt 已经要求了，但你提醒也不多余 |
| **明确 task 编号范围** | 不要只说"继续"，说"T04-T06"——builder 知道去看哪些验收标准 |
| **Plan-Issue 修复时给精确步骤** | git mv 路径、fixture 名称、断言改成 == 还是 >=——越精确重建成本越低 |
| **要求 "全绿才进 commit"** | 不要给 builder 留"绿不了就先 commit 后面再修"的余地 |
| **commit message 也指定风格** | 用 `fix:` / `feat:` / `chore:` 前缀，避免 builder 写 "general improvements" |
