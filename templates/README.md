# templates/ — 可选片段（按所选技术栈追加）

本目录存放**与具体语言/工具链相关**的可选文件片段。仓库根的 `.gitignore` / `.editorconfig` / `.gitattributes` 维持"语言无关最小集"；选定技术栈后，从这里取对应片段追加到根文件即可。

## 当前已收录

| 文件 | 用途 | 用法 |
|---|---|---|
| `AGENTS.md` | 宿主仓库的 AGENTS.md 模板（极简版，指向框架文档） | `cp templates/AGENTS.md ./AGENTS.md` |
| `.session-log.md` | 宿主仓库的 session 操作日志模板（由 maintainer / planner / builder 自动追加） | `cp templates/.session-log.md ./docs/.session-log.md`（需先 `mkdir -p docs`） |
| `PROGRESS.md` | builder 工作日志骨架（5 节：Done / Blocked / Plan-Issue / Need-Approval / Review-Round） | `cp templates/PROGRESS.md <项目目录>/PROGRESS.md` |
| `REVIEW.md` | reviewer 审查报告模板（验收核对 + Must-Fix + Nice-to-Have + 下一步指令） | 首次 review 时 `cp templates/REVIEW.md <项目目录>/REVIEW.md`；后续 reviewer 覆盖式写入 |
| `gitignore-python.txt` | Python 项目通用忽略片段（pycache / dist / venv / 类型检查器缓存等） | `cat templates/gitignore-python.txt >> .gitignore` |
| `gitignore-node.txt` | Node / TypeScript 项目通用忽略片段（node_modules / dist / tsbuildinfo 等） | `cat templates/gitignore-node.txt >> .gitignore` |
| `gitignore-rust.txt` | Rust / Cargo 项目通用忽略片段（target/ / rustc_info.json 等） | `cat templates/gitignore-rust.txt >> .gitignore` |
| `user-prompts/` | 用户 → agent 提示词模板（builder / planner / maintainer） | 直接阅读对应 `.md` 文件，复制模板填入具体需求 |
| `project-ideas/` | 项目 idea 模板与学习路线图（TEMPLATE.md / 00-ROADMAP.md） | 按 TEMPLATE.md 填写新项目 prompt，续号放入此目录 |

## 未来计划收录

按 `<FRAMEWORK-PATH>/docs/agent-rules.md` 沉淀经验，**每跑一个新语言/生态的 case study**，对应的 ignore / config 片段进这个目录；**每暴露新的工作流模板需求**，对应的 agent 工作产物模板也进这里：

- **工作产物模板（agent 契约）**：`PROGRESS.md` / `REVIEW.md` 已经收录；未来可视需要加 `PLAN.md` 骨架、`learning-notes/` 目录骨架等
- **工具链 / 配置片段**：

- `gitignore-go.txt` —— Go module
- `editorconfig-jvm.txt` —— Kotlin / Java / Scala 等

## 命名约定

- 片段类型前缀：`gitignore-*` / `editorconfig-*` / `gitattributes-*`
- 语言/生态后缀：纯小写，跟 PLAN 第 0 节"语言 / 运行时"字段对齐（`python` / `node` / `rust` / `go` / `jvm` 等）
- 文件后缀**统一用 `.txt`**（不是 `.gitignore`）—— 避免被仓库根的 `.gitignore` 自身规则误识别 / 影响 GitHub linguist
