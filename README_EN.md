# AI Project Workflow — From Idea to Code

> 🇨🇳 [中文文档](README.md)

> **A suite of 5 composable skills for AI coding agents that guides your project from raw idea all the way to production code.**
>
> Whether you use Claude Code, Cursor, Codex, or any other AI coding tool — install these skills, say one phrase, and the agent automatically activates the right skill to walk you through requirements → design → tasks → implementation → deployment.

---

## What Problem Does This Solve?

The most common AI-coding disasters:

- ❌ You casually describe an idea, the AI starts writing code immediately, and halfway through you realize everything is going in the wrong direction
- ❌ Requirements are vague, technical decisions are guesswork, module boundaries are fuzzy — the codebase turns into a tangled mess
- ❌ As the project grows complex, context gets too long, and the AI starts forgetting earlier design decisions
- ❌ You have an idea in your head but don't know how to communicate it to the AI step by step

These 5 skills are designed to fix exactly this — **they make the AI help you see the road clearly before you write a single line of code**.

---

## The Workflow: One Pipeline, End to End

```
Your Idea ──→ [1] PRD ──→ [2] Detailed Design ──→ [3] Task Generator ──→ [4] Vibe Coding Prompt ──→ AI Writes Code
               │              │                        │                         │
               │              │                        │                         │
          WHAT to build?  HOW to design?         WHO does what?          Agent, start building
```

Or take the express lane:

```
Your Idea ──→ [5] Project Orchestrator ──→ Fully automated end-to-end (including deployment)
```

---

## Quick Install

```bash
# Install all 5 skills
npx skills add DTAny/ai-project-workflow --all

# Or pick and choose
npx skills add DTAny/ai-project-workflow --skill prd --skill detailed-design --skill task-generator

# Install globally (available across all your projects)
npx skills add DTAny/ai-project-workflow --all -g
```

No configuration needed after installation. Just use any of the trigger phrases listed below and your agent will activate the right skill automatically.

---

## The 5 Skills

### 1. `prd` — Product Requirements Document

> **Turn a fuzzy idea into a structured requirements document.**

When you have nothing written down yet, this skill helps you nail down requirements through multi-round Q&A. It never guesses your intent — every decision is either confirmed by you or explicitly labeled as a recommendation.

**Output file:** `doc/prd.md` — includes functional requirements, non-functional requirements, technical architecture suggestions, and a list of open questions.

**Trigger it with:**

| English | 中文 |
|---------|------|
| "create a PRD" | "帮我写 PRD" |
| "write a requirements document" | "生成产品需求文档" |
| "draft a product spec" | "写一个需求文档" |
| "before we start coding" | "产品需求" / "规格说明" |

---

### 2. `detailed-design` — Detailed Technical Design

> **Transform the PRD into module-level technical specifications.**

Taking the PRD as input, it identifies technical decision gaps (the PRD says JWT but not which signing algorithm? It asks you to decide), then generates data structures, service interfaces, business logic flows, error handling strategies, and testing plans for every module.

**Prerequisite:** `doc/prd.md`

**Output file:** `doc/detailed-designed.md` — a modular technical design document where every design decision is traceable back to the PRD or your confirmation.

**Trigger it with:**

| English | 中文 |
|---------|------|
| "create a detailed design" | "帮我做详细设计" |
| "generate a technical design" | "生成技术方案" |
| "architecture design" / "design doc" | "架构设计" / "技术设计" |
| "create an implementation plan" | "基于 PRD 生成设计文档" |

---

### 3. `task-generator` — Task Breakdown

> **Break the technical design into minimum executable task checklists.**

Each task is scoped to a single file, with a concrete file path, implementation description, and validation criteria. Tasks are ordered by dependency — infrastructure comes first, dependent modules come after their dependencies.

**Prerequisite:** `doc/prd.md` + `doc/detailed-designed.md`

**Output files:** `doc/tasks/module-*.md` (one task file per module) + `doc/tasks/progress.md` (master progress dashboard)

**Trigger it with:**

| English | 中文 |
|---------|------|
| "break down modules into tasks" | "任务拆分" / "给我拆任务" |
| "create task lists" | "生成任务清单" |
| "generate a task board" | "拆分开发任务" |
| "prepare implementation tickets" | "把设计文档拆成任务" |

---

### 4. `vibe-coding-prompt` — AI Implementation Blueprint

> **Fuse all design documents into a single, self-contained prompt that an AI agent can execute directly.**

This prompt is the "blueprint" for another AI agent (or yourself). It specifies: how many agents to use, which agent handles which modules, interface contracts between modules, quality gates, security policies, and a complete configuration manifest.

**Prerequisite:** PRD + Detailed Design + Task Breakdown (all three)

**Output file:** `doc/prompt.md` — a self-contained executable implementation instruction set

**Trigger it with:**

| English | 中文 |
|---------|------|
| "generate a coding prompt" | "生成 AI 开发 Prompt" |
| "create an implementation prompt" | "写一个实现 Prompt" |
| "turn design docs into executable instructions" | "生成开发指令" |

---

### 5. `project-orchestrator` — Full Project Orchestrator

> **One phrase to launch it all — from zero to deployed, fully orchestrated.**

This skill sequentially guides you through 6 phases: **Requirements Confirmation → PRD → Detailed Design → Task Breakdown → Code Implementation → Deployment & Testing**. The main Agent orchestrates but never writes code itself; every phase ends with a confirmation checkpoint; after each phase you're prompted to start a fresh session to avoid context bloat.

**Trigger it with:**

| English | 中文 |
|---------|------|
| "start a new project" | "开始新项目" |
| "full project workflow" | "帮我从零开始做一个项目" |
| "project orchestration" | "完整的开发流程" |
| "new project with full workflow" | "我要开始一个新项目" |

---

## Which Path Should You Take?

| Your Situation | Recommended Action |
|----------------|-------------------|
| Fresh idea, nothing written down | Use `project-orchestrator` — say "start a new project" |
| Already have a PRD, need design | Say "create a detailed design" |
| Have PRD + design, ready to break it down | Say "break down modules into tasks" |
| All three docs ready, time to execute | Say "generate a coding prompt" |
| Just want a requirements doc | Use `prd` — say "create a PRD" |

---

## Supported AI Coding Tools

These skills follow the [Agent Skills specification](https://agentskills.io) and work with:

- **Claude Code** (recommended — supports all advanced features)
- Cursor / Codex / OpenCode / Windsurf
- 35+ other compatible AI coding agents

Target specific tools with the `-a` flag:

```bash
npx skills add DTAny/ai-project-workflow --all -a claude-code -a opencode
```

---

## Core Design Philosophy

Every skill follows the same iron rule: **DO NOT GUESS**.

- Every technical decision is recorded with its source: "explicitly stated by you" or "recommended by AI"
- When a gap is found, the skill stops and asks — never silently fills it with a default
- Every step has a confirmation checkpoint — it only moves forward after you say yes

---

## Troubleshooting

### Trigger phrases not working after install?

If a skill appears in the list but the agent never activates it, check your `SKILL.md` line endings — they must be **LF** (`\n`), not **CRLF** (`\r\n`).

On Windows, `git clone` and `npx skills add` can auto-convert LF to CRLF when `core.autocrlf=true` is set. **Claude Code's YAML frontmatter parser silently fails on CRLF**, causing the `description` field to be dropped — the skill name appears but with no description, so trigger phrases can never match.

This repository uses `.gitattributes` to force LF for all skill files, so this should not happen under normal circumstances. If you clone manually or install from another source, verify your line endings.

```bash
# Check for CRLF
file skills/*/SKILL.md | grep CRLF

# Fix it
sed -i 's/\r$//' skills/*/SKILL.md
```

---

## License

MIT
