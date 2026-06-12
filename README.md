# AI Project Workflow · 项目实现工具链

> 🇬🇧 [Read in English](README_EN.md)

> **一套面向 AI 编程 Agent 的技能包，将项目从想法到代码的全流程拆解为 5 个可组合的 Skill。**
>
> 无论你是用 Claude Code、Cursor、Codex 还是其他 AI 编程工具，安装后只需说一句话，Agent 就会自动激活对应的 Skill，带你走完需求 → 设计 → 任务 → 实现 → 部署的每一步。

---

## 这个项目解决了什么问题？

用 AI 做项目时最常见的翻车现场：

- ❌ 你随口说了一个想法，AI 就开始写代码，写到一半发现方向全歪了
- ❌ 需求不清晰、技术决策靠猜、模块边界模糊，代码越堆越乱
- ❌ 项目一复杂，上下文太长，AI 开始忘掉之前的设计决策
- ❌ 你脑子里有想法但不知道怎么一步步告诉 AI

这 5 个 Skill 就是为解决这些问题设计的 —— **让 AI 在你写第一行代码之前，先帮你把路看清楚**。

---

## 工作流：一条 Pipeline 走到底

```
你的想法 ──→ [1] PRD ──→ [2] Detailed Design ──→ [3] Task Generator ──→ [4] Vibe Coding Prompt ──→ AI 写代码
              │              │                        │                         │
              │              │                        │                         │
         要做什么？     怎么设计？              谁来做什么？              Agent 请开始执行
```

或者一步到位：

```
你的想法 ──→ [5] Project Orchestrator ──→ 全自动编排以上所有阶段（含部署）
```

---

## 快速安装

```bash
# 安装全部 5 个 Skill
npx skills add DTAny/ai-project-workflow --all

# 或者挑着装
npx skills add DTAny/ai-project-workflow --skill prd --skill detailed-design --skill task-generator

# 全局安装（让所有项目都能用）
npx skills add DTAny/ai-project-workflow --all -g
```

安装后无需任何配置，直接用下面表格里的短语触发即可。

---

## 5 个 Skill 详解

### 1. `prd` — 产品需求文档

> **把模糊的想法变成结构化的需求文档。**

在你还没有任何文档时，这个 Skill 会通过多轮问答帮你梳理需求。它绝不猜测你的意图 —— 每个决策要么由你确认，要么标记为"推荐"。

**产出文件：** `doc/prd.md` — 包含功能需求、非功能需求、技术架构建议和待决问题清单。

**你可以这样触发它：**

| 中文 | English |
|------|---------|
| "帮我写 PRD" | "create a PRD" |
| "生成产品需求文档" | "write a requirements document" |
| "写一个需求文档" | "draft a product spec" |
| "产品需求" / "规格说明" | "before we start coding" |

---

### 2. `detailed-design` — 详细技术设计

> **把需求文档变成模块级技术方案。**

以 PRD 为输入，识别技术决策缺口（PRD 写了 JWT 但没写加密算法？让你选），然后生成每个模块的数据结构、服务接口、业务流程、错误处理和测试策略。

**前置依赖：** 需要 `doc/prd.md`

**产出文件：** `doc/detailed-designed.md` — 模块化技术设计文档，每个设计决策可追溯到 PRD 或你的确认。

**你可以这样触发它：**

| 中文 | English |
|------|---------|
| "帮我做详细设计" | "create a detailed design" |
| "生成技术方案" | "generate a technical design" |
| "架构设计" / "技术设计" | "architecture design" / "design doc" |
| "基于 PRD 生成设计文档" | "create an implementation plan" |

---

### 3. `task-generator` — 任务拆分

> **把技术设计拆成最小可执行任务清单。**

每个任务精确到一个文件，包含具体的文件路径、实现内容和验证标准。任务按依赖排序 —— 基础设施先行，依赖模块做完再做上层。

**前置依赖：** 需要 `doc/prd.md` + `doc/detailed-designed.md`

**产出文件：** `doc/tasks/module-*.md`（每个模块一个任务文件）+ `doc/tasks/progress.md`（总进度看板）

**你可以这样触发它：**

| 中文 | English |
|------|---------|
| "任务拆分" / "给我拆任务" | "break down modules into tasks" |
| "生成任务清单" | "create task lists" |
| "拆分开发任务" | "generate a task board" |
| "把设计文档拆成任务" | "prepare implementation tickets" |

---

### 4. `vibe-coding-prompt` — AI 实现指令

> **融合所有设计文档，生成一份 AI 可以直接执行的实现 Prompt。**

这份 Prompt 是给另一个 AI Agent（或你自己）的"施工图纸"，包含：用几个 Agent、每个 Agent 做什么模块、模块间的接口约束、质量门禁、安全规范、配置清单。

**前置依赖：** 需要 PRD + Detailed Design + Task Breakdown 三件套

**产出文件：** `doc/prompt.md` — 自包含的可执行实现指令

**你可以这样触发它：**

| 中文 | English |
|------|---------|
| "生成 AI 开发 Prompt" | "generate a coding prompt" |
| "写一个实现 Prompt" | "create an implementation prompt" |
| "生成开发指令" | "turn design docs into executable instructions" |
| "把设计文档转成可执行的开发指令" | — |

---

### 5. `project-orchestrator` — 全程编排器

> **一句话启动，从零到部署全程自动编排。**

这个 Skill 会依次引导你走完 6 个阶段：**需求确认 → PRD 生成 → 详细设计 → 任务拆分 → 代码实现 → 部署测试**。主 Agent 只负责编排，绝不亲自写代码；每个阶段结束都会停顿等你确认；每个阶段完成后会提示你开新会话避免上下文过长。

**你可以这样触发它：**

| 中文 | English |
|------|---------|
| "开始新项目" / "走一遍完整的项目实现流程" | "start a new project" |
| "帮我从零开始做一个项目" | "full project workflow" |
| "完整的开发流程" | "project orchestration" |
| "我要开始一个新项目" | "new project with full workflow" |

---

## 怎么选？走完整 Pipeline 还是单步使用？

| 你的情况 | 推荐方式 |
|----------|---------|
| 有一个新想法但什么都没写 | 用 `project-orchestrator`，一句"开始新项目"搞定一切 |
| 已经有 PRD，只要设计 | 直接说"帮我做详细设计" |
| 已有 PRD + 设计，需要拆任务 | 直接说"任务拆分" |
| 三件套全齐，就差执行 | 直接说"生成 AI 开发 Prompt" |
| 只想写个需求文档给别人看 | 用 `prd` | "帮我写 PRD" |

---

## 支持的 AI 编程工具

这些 Skill 遵循 [Agent Skills 规范](https://agentskills.io)，目前在以下工具中可用：

- **Claude Code**（推荐 — 支持全部高级特性）
- Cursor / Codex / OpenCode / Windsurf
- 以及 35+ 其他兼容 Agent 的工具

安装时可以通过 `-a` 参数指定目标工具：

```bash
npx skills add DTAny/ai-project-workflow --all -a claude-code -a opencode
```

---

## 核心设计哲学

每个 Skill 都遵循同一条铁律：**不猜测 (DO NOT GUESS)**。

- 你的每个技术决策都会被记录来源：是"你明确说的"还是"AI 推荐的"
- 遇到缺口会主动停下来问你，不会悄悄用默认值填上
- 每一步都有停顿确认 (checkpoint)，你点头了才继续

---

## 疑难解答

### 安装后触发短语不生效？

如果 skill 已安装但 Agent 无法通过触发短语激活它，检查 `SKILL.md` 文件行尾符是否为 **LF**（`\n`）而非 **CRLF**（`\r\n`）。

Windows 上的 `git clone` 和 `npx skills add` 会因 `core.autocrlf=true` 设置自动将 LF 转为 CRLF，而 **Claude Code 的 YAML frontmatter 解析器无法处理 CRLF**，会导致 `description` 字段被静默丢弃——skill 名称出现在列表中但没有描述，也就无法匹配触发短语。

此仓库已通过 `.gitattributes` 强制所有 skill 文件使用 LF，正常情况下不会出现此问题。如果你手动 clone 或从其他来源安装，请确认文件行尾符。

```bash
# 检查行尾符
file skills/*/SKILL.md | grep CRLF

# 手动转换
sed -i 's/\r$//' skills/*/SKILL.md
```

---

## 许可

MIT
