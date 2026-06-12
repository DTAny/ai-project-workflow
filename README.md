# AI Project Skills — 从需求到实现的完整工具链

一套用于 AI 辅助软件开发的完整技能包，覆盖从 PRD 撰写到代码实现的全流程。

## Skills 列表

| Skill | 描述 |
|-------|------|
| `prd` | 通过多轮 Q&A 生成结构化产品需求文档 (PRD) |
| `detailed-design` | 从 PRD 生成模块级技术详细设计文档 |
| `task-generator` | 将详细设计拆分为最小可执行任务清单 |
| `vibe-coding-prompt` | 生成 AI 可执行的 Vibe Coding 实现 Prompt |
| `project-orchestrator` | 编排完整项目流程：PRD → 设计 → 任务 → 实现 → 部署 |

## 工作流

```
┌─────┐    ┌────────────────┐    ┌────────────────┐    ┌────────────────┐    ┌──────────────┐
│ PRD │───→│ Detailed       │───→│ Task           │───→│ Vibe Coding    │───→│ AI 代码实现  │
│     │    │ Design         │    │ Generator      │    │ Prompt         │    │              │
└─────┘    └────────────────┘    └────────────────┘    └────────────────┘    └──────────────┘
     ↑            ↑                     ↑                     ↑
WHAT to build   HOW to build      WHAT to implement     HOW to tell AI
```

或使用 `project-orchestrator` 一键编排全部六个阶段。

## 安装

### 安装全部 Skills

```bash
npx skills add <your-github-username>/ai-project-skills --all
```

### 选择性安装

```bash
# 只安装 PRD skill
npx skills add <your-github-username>/ai-project-skills --skill prd

# 安装多个
npx skills add <your-github-username>/ai-project-skills --skill prd --skill detailed-design --skill task-generator

# 全局安装（所有项目可用）
npx skills add <your-github-username>/ai-project-skills --all -g
```

### 指定 Agent

```bash
# 只安装到 Claude Code
npx skills add <your-github-username>/ai-project-skills --all -a claude-code
```

## 触发词

每个 skill 都内置了中英文触发词，安装后 AI Agent 会在用户说出相关短语时自动激活对应的 skill。详见各 `SKILL.md` 的 `description` 字段。

## Skills 详情

### PRD — 产品需求文档

从用户想法出发，通过多轮结构化 Q&A（`AskUserQuestion`）收集需求，生成包含功能需求、非功能需求、技术架构、开放问题清单的完整 PRD 文档。

**核心原则：不猜测 (DO NOT GUESS)** — 每个技术决策要么来自用户确认，要么标注为推荐。

### Detailed Design — 详细设计

基于 PRD 生成模块级技术详细设计，包含数据结构、服务接口、算法、错误处理、测试策略。每步停顿确认，技术决策可追溯。

### Task Generator — 任务拆分

将详细设计文档拆分为最小可执行任务清单，每个任务指定具体输出文件和验证标准，按依赖关系排序。

### Vibe Coding Prompt — AI 实现入口

融合 PRD + 详细设计 + 任务清单，生成自包含的 AI 实现 Prompt，包含 Agent 模式选择、阶段范围、质量门禁、模块规格和全局约束。

### Project Orchestrator — 项目流程编排

完整的六阶段流程编排器，主 Agent 只编排不写代码，每阶段定稿前停顿确认，全程集成部署环境规划与热部署能力。

## 许可

MIT
