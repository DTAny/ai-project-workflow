---
name: vibe-coding-prompt
description: Generate a self-contained Vibe Coding Prompt from PRD, detailed design, and task breakdown. The entry point for an AI orchestrator to implement the entire project. Trigger: 生成AI开发prompt, 写一个实现prompt, 生成开发指令, 把设计文档转成可执行的开发指令, generate coding prompt, create implementation prompt.
---

# Vibe Coding Prompt Generation Skill

## Core Principle — DO NOT GUESS

**This is the most important rule of this skill. Every decision you make about
how the AI should implement the project must be traceable.**

The PRD and detailed design tell you WHAT and HOW. The Vibe Coding prompt tells
an AI agent HOW TO EXECUTE. Between the design docs and the prompt is a gap:
- Single agent or multi-agent?
- Which phases to include?
- What's the testing rigor?
- How to handle PRD open questions?

**You must not fill these gaps silently.** Every implementation strategy decision
must be either confirmed by the user or clearly labeled as a recommendation.

**Examples of guessing (DON'T):**
- Design has 12 modules → you assume multi-agent without asking
- PRD has 6 phases → you arbitrarily pick phases 1-3
- PRD says "offline grace period (suggest: 7 days)" → you hardcode 7 days without asking
- Design mentions "≥80% coverage" → you skip asking because it seems clear

**What to do instead:**
- "The design covers 12 modules — I recommend multi-agent mode (1 orchestrator + 12 module agents). Does that approach work for you?"
- "The PRD lists 6 phases. Should the prompt cover all of them, or just a subset?"
- "For the PRD's open questions (trial duration, etc.), should we use the suggested defaults or would you like to decide each now?"

## Prerequisites

The user must have, at minimum:
1. **PRD** (`doc/prd.md`) — requirements document
2. **Detailed Design** (`doc/detailed-designed.md` or equivalent) — module-level technical design
3. **Task Breakdown** (`doc/tasks/` directory) — per-module task lists

If any of these are missing:
- Point to the missing prerequisite: "I need [document] before I can generate a coding prompt."
- If PRD is missing → suggest the PRD skill
- If detailed design is missing → suggest the detailed-design skill
- If tasks are missing → suggest the task-generator skill

## Skill Chain & Related Skills

This skill is the **final step** in a 4-skill pipeline. Each predecessor produces an
artifact that this skill consumes:

```
┌─────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  prd    │───→│ detailed-design  │───→│ task-generator   │───→│ vibe-coding-     │
│         │    │                  │    │                  │    │ prompt (THIS)    │
│ Input:  │    │ Input: doc/prd.md│    │ Input: PRD +     │    │ Input: PRD +     │
│  user   │    │ Output: doc/     │    │   detailed design│    │   design + tasks │
│  ideas  │    │   detailed-      │    │ Output: doc/     │    │ Output: doc/     │
│ Output: │    │   designed.md    │    │   tasks/*.md     │    │   prompt.md      │
│  doc/   │    │                  │    │                  │    │                  │
│  prd.md │    │                  │    │                  │    │                  │
└─────────┘    └──────────────────┘    └──────────────────┘    └──────────────────┘
     ↑              ↑                      ↑                      ↑
     │              │                      │                      │
  WHAT to build  HOW to build         WHAT to implement      HOW to tell AI
                                       (task-level)          to implement it
```

| Role | Skill | Produces | Consumed by |
|------|-------|----------|-------------|
| Requirements | `prd` | `doc/prd.md` | detailed-design, task-generator, THIS |
| Design | `detailed-design` | `doc/detailed-designed.md` | task-generator, THIS |
| Tasking | `task-generator` | `doc/tasks/module-*.md` | THIS |
| **Execution** | **`vibe-coding-prompt` (THIS)** | **`doc/prompt.md`** | AI orchestrator agent |

If any upstream artifact is missing, this skill **MUST** direct the user to the
corresponding skill rather than proceeding with incomplete inputs (see
[Prerequisites](#prerequisites) and [Failure Modes](#failure-modes--fallbacks)).

## Process

### Step 1: Read and understand the inputs

Read all three input sources thoroughly:

1. **PRD** (`doc/prd.md`):
   - Functional requirements (F-XXX-NN)
   - Non-functional requirements (NFR-*)
   - Technical architecture recommendations
   - Development phases
   - Open questions (§7)

2. **Detailed Design** (`doc/detailed-designed.md`):
   - Architecture overview (layers, module dependency graph)
   - Per-module design: data structures, service interfaces, business rules, error handling
   - Configuration reference (all env vars)
   - Project directory structure

3. **Task Breakdown** (`doc/tasks/module-[a-l].md` + `progress.md`):
   - Per-module task lists with file paths
   - Completion criteria
   - Output file manifests

From these inputs, extract:
- **Total module count** and their dependencies
- **Key technical decisions** (tech stack, algorithms, formats)
- **Cross-cutting concerns** (security, logging, deployment)
- **Phase grouping** and dependency chain
- **PRD open questions** still unresolved

🔴 **CHECKPOINT · 🛑 STOP** — Present your understanding to the user:

```
## Input Summary

| Document | Status | Key Contents |
|----------|--------|-------------|
| PRD | ✓ | X functional areas, Y NFR categories, Z open questions |
| Detailed Design | ✓ | N modules, M configuration items |
| Task Breakdown | ✓ | N task files, total ~T sub-tasks |

**Inferred Project Profile:**
- Tech stack: [summary]
- Module count: N
- Phases: 1-N
- Estimated complexity: [Low/Medium/High]

Does this match your understanding? Ready for implementation strategy Q&A?
```

### Step 2: Q&A — determine implementation strategy

Use `AskUserQuestion` to elicit implementation decisions. **Max 4 questions per round.**
Typically one round is sufficient for this skill (unlike PRD or detailed-design which
need multiple rounds), since the technical decisions are already made in the design docs.

**Standard question set (adapt to project):**

| # | Question | Options | When to ask |
|---|----------|---------|-------------|
| 1 | **Agent mode** | Multi-agent (recommended for ≥6 modules) / Single-agent | Always |
| 2 | **Phase scope** | All phases / Core only / Specific phases | Always |
| 3 | **PRD open questions** | Use suggested defaults / Decide each now / Mark as optional in prompt | If PRD §7 has unresolved items |
| 4 | **Testing rigor** | ≥80% coverage + type check / Per-module tests no hard target / Core modules only | Always |

**Agent mode decision logic:**
- ≥6 modules with clear boundaries → recommend multi-agent (orchestrator + module agents)
- 3-5 modules → either mode; ask with trade-offs
- 1-2 modules → recommend single-agent

**Question design guidelines:**
- First option should be your recommendation with "(推荐 / Recommended)" label
- Each option must explain trade-offs in the description
- If the user's choice would conflict with a PRD requirement, flag it immediately

🔴 **CHECKPOINT · 🛑 STOP** — After the Q&A round, present a summary of all
implementation strategy decisions before writing the prompt:

```
## Implementation Strategy Confirmed

| Decision | Choice |
|----------|--------|
| Agent Mode | Multi-agent (1 orchestrator + N module agents) |
| Phase Scope | Phases 1-N |
| Open Questions | Using suggested defaults |
| Testing | ≥80% coverage + strict type checking |

Does this look correct? Ready to write the Vibe Coding prompt?
```

### Step 3: Generate the prompt document

Create or overwrite `doc/prompt.md`.

**The prompt document is a self-contained instruction manual for an AI agent
(orchestrator) to implement the entire project. It must be:**

1. **Complete** — the AI agent must never need to read the original design docs
   to understand what to build (though it should be directed to read them for details)
2. **Structured** — clear sections for overview, architecture, agent workflow, module specs
3. **Executable** — every module has concrete inputs, outputs, dependencies, and acceptance criteria
4. **Constrained** — security rules, coding standards, and testing requirements are non-negotiable

#### Required Sections

```
# Vibe Coding Prompt — [Project Name]

## 1. Project Overview
- Project definition (1 paragraph)
- Core features (table)
- Input documents (table referencing doc/*.md paths)

## 2. Technical Architecture
- Technology stack table (immutable — must not be changed)
- Layered architecture diagram (ASCII)
- Module dependency graph (ASCII)
- Development order (strict dependency chain)

## 3. Multi-Agent Collaboration Mode (if multi-agent)
### 3.1 Role Definitions
- Orchestrator agent: responsibilities, tools, scope
- Module agent: responsibilities, workflow, deliverables
### 3.2 Scheduling Rules
- Parallel vs sequential rules
- Interface contract handoff
### 3.3 Acceptance Criteria
- Type check passes
- Tests pass with ≥80% coverage
- File manifest matches spec
- Interface contract matches design

## 4. Module Specifications (one per module)
For each module (A, B, C...):
| Attribute | Value |
|-----------|-------|
| Phase | N |
| Dependencies | Module X, Y |
| Design Doc Ref | doc/detailed-designed.md §N |
| Task List Ref | doc/tasks/module-[x].md |

Then:
- **Responsibilities** (1 sentence)
- **Key Requirements** (bullet list of non-negotiable constraints)
- **API Endpoints** (table if applicable)
- **Error Codes** (table if applicable)
- **Interface Contract** (TypeScript interface snippet)
- **Output Files** (tree listing)

## 5. Development Workflow & Quality Gates
### 5.1 Orchestrator Workflow
Step-by-step ASCII flow diagram for the main agent:
  Step 0: Project initialization
  Step 1-N: Per-phase module implementation and verification
  Final Step: Integration verification
### 5.2 Module Agent Workflow
  1. Read inputs → 2. Implement → 3. Test → 4. Self-check
### 5.3 Quality Gates
Per-phase checklist table: type check, tests, contract consistency, security check

## 6. Global Constraints
### 6.1 Coding Standards
- TypeScript strict mode
- Dependency injection pattern
- Interface abstraction
- Error format (RFC 7807)
- Logging format (pino JSON)
- Do not guess rule
### 6.2 Security Constraints (NON-NEGOTIABLE)
Numbered list of must-follow security rules
### 6.3 Configuration Reference
- Required env vars (table)
- Optional env vars with defaults (table)
### 6.4 Project Directory Structure
Full tree with all expected files

## 7. Delivery Checklist
- Code deliverables (checkboxes)
- Documentation deliverables (checkboxes)
- Quality deliverables (checkboxes)
```

#### Writing Rules

1. **Every module section must be self-contained.** The module agent should be able
   to work from its section alone (plus the referenced design doc sections).

2. **Constraints must be non-negotiable.** Use imperative language: "MUST", "MUST NOT",
   "NON-NEGOTIABLE". The AI agent should not second-guess these.

3. **Interface contracts must be exact.** Provide TypeScript interface definitions
   (not pseudocode) so the module agent knows exactly what to implement.

4. **Dependencies must be explicit.** Each module lists exactly which other modules
   it depends on. This determines scheduling order.

5. **File paths must be exact.** The output file tree for each module should match
   what the task list and design doc specify.

6. **Reference but don't duplicate.** The prompt refers to design doc sections for
   full detail; it does not copy the entire design doc. It provides the module's
   "interface contract" and constraints — enough to implement without ambiguity.

7. **The prompt is for an AI agent, not a human developer.** Write in imperative
   ("The orchestrator MUST...", "The module agent MUST..."). Be unambiguous.
   Define acceptance criteria in verifiable terms.

🔴 **CHECKPOINT · 🛑 STOP** — Before delivering to the user, self-review the generated
`doc/prompt.md` against this checklist. Fix any gaps before proceeding to Step 4.

**Self-review checklist:**
- [ ] All 7 required sections (§1–§7) are present and non-empty
- [ ] Module count matches the design doc's module list
- [ ] Every module spec has: Phase, Dependencies, Key Requirements, Interface Contract, Output Files
- [ ] Security constraints (§6.2) use MUST/MUST NOT language — no "should" or "consider"
- [ ] Every "MUST" constraint is verifiable (has a concrete check: command to run, metric, file count)
- [ ] All file paths in output manifests are absolute from project root
- [ ] Environment variable tables (§6.3) list every required/optional var with defaults
- [ ] The orchestrator workflow (§5.1) shows clear scheduling order with phase boundaries
- [ ] No module's interface contract contradicts another's dependency specification
- [ ] Prompt references at least doc/prd.md, doc/detailed-designed.md, and doc/tasks/

If ≥2 items fail, fix them in doc/prompt.md before continuing. If <2 items fail,
note them in the delivery summary so the user is aware.

### Step 4: Deliver and prompt for review

Summarize:
- Total modules covered
- Agent mode chosen and why
- Phase scope
- Key constraints embedded in the prompt
- What the user should verify during review

🔴 **CHECKPOINT · 🛑 STOP** — Explicitly ask: "Please review the Vibe Coding prompt
and let me know if anything needs to change — modules mis-specified, constraints
too strict/loose, sections that need more detail, or the agent mode strategy."

### Step 5: Iterate if needed

The user may want to:
- Adjust module specs (add/remove constraints)
- Change agent mode strategy
- Add or remove phases
- Tighten or relax quality gates
- Fix file paths or interface contracts

Make targeted edits to `doc/prompt.md`. If a module's interface contract changes,
check if it affects other modules' dependency specifications.

## Anti-Patterns — Things You Must NOT Do

| # | 反模式 | 为什么不要做 | 替代做法 |
|---|--------|-------------|---------|
| 1 | **静默决定 Agent 模式** — 项目有 12 个模块，直接假设用多 Agent 而不问用户 | 用户可能有特定约束（token 预算、风格一致性偏好）要求单 Agent | 分析复杂度后推荐一个选项，但让用户通过 Q&A 确认 |
| 2 | **静默选择 Phase 范围** — PRD 列了 6 个 Phase，你只写了前 3 个的 prompt | 用户投入时间写了 Phase 4-6 的设计，跳过它们会浪费已有工作 | 通过 Q&A 确认：全部、核心、或用户指定范围 |
| 3 | **静默处理 PRD 开放问题** — 直接采用 PRD §7 的建议值而不确认 | 用户可能在设计阶段已经有新想法，建议值已过时 | Q&A 中提问：使用建议默认值、现在决定、还是标记为可选 |
| 4 | **过度简化模块规格** — prompt 只写了模块名称和一句话描述 | 子 Agent 无事可做，只能去读设计文档自己理解 → 理解偏差 → 实现不一致 | 每个模块至少包含：职责、关键约束、API 端点、错误码、接口契约、产出文件 |
| 5 | **遗漏安全约束** — prompt 没有强调 "Key 不存明文"、"私钥不入代码" 等 | 子 Agent 可能为了方便把 Key 存明文或硬编码私钥 → 安全漏洞 | 在 §6 全局约束中专设 "安全约束（不可违反）" 节，用 MUST NOT 语言 |
| 6 | **模糊的验收标准** — "测试通过" / "代码质量好" / "基本功能正常" | 子 Agent 无法自我验证；主 Agent 无法客观判断是否通过 | 验收标准必须是可测量的：typecheck 0 errors、test 全部通过、覆盖率 ≥80%、文件清单匹配 |
| 7 | **不引用源文档** — prompt 是独立文档，但没有指向 PRD/设计文档的引用 | 子 Agent 实现的细节可能与设计文档不一致，因为它没有被告知去读 | 每个模块标注 "设计文档: doc/detailed-designed.md §N"，子 Agent 必须先去读 |
| 8 | **忽略配置项** — 模块规格没有列出该模块依赖的环境变量 | 实现完成后才发现某些开关没暴露 → 改代码 | 每个模块列出配置依赖；§6.3 汇总所有环境变量及其默认值 |
| 9 | **Agent 角色模糊** — prompt 说 "实现模块 A" 但没有明确是主 Agent 还是子 Agent 做 | 主 Agent 可能自己写了所有代码（单点瓶颈），或子 Agent 不知道该做什么 | 明确分工：主 Agent 只调度/验收/集成；子 Agent 读设计→实现→测试→输出契约 |

## Failure Modes & Fallbacks

If something goes wrong during the prompt generation process, do NOT silently skip or guess.
For each failure: try the first-line fix; if that still fails, use the fallback.

| Step | 触发条件 | 一线修复 | 仍失败兜底 |
|------|---------|----------|-----------|
| Step 1 | `doc/prd.md` 不存在 | 检查用户是否指定了其他路径: "I don't see a PRD. Do you have requirements elsewhere?" | 引导用户: "Let's create a PRD first — would you like to use the PRD skill?" |
| Step 1 | `doc/detailed-designed.md` 不存在 | 检查是否有类似命名的设计文档 | 引导用户: "I need a detailed design document. Would you like to use the detailed-design skill?" |
| Step 1 | `doc/tasks/` 目录为空或不存在 | 检查是否有其他任务文件 | 引导用户: "The task breakdown is needed for accurate module specs. Use the task-generator skill to create it." |
| Step 1 | PRD 和设计文档之间存在矛盾（如 PRD 说 Phase 1 包含 X，但设计文档说 X 在 Phase 2） | 列出矛盾，要求用户澄清: "The PRD and design doc disagree on [X]. Which is correct?" | 以设计文档为准（更详细），在 prompt 中标注差异 |
| Step 2 | 用户跳过所有 Q&A 问题或全部选 "你推荐" | 采用推荐选项，在 prompt 中标注 "per recommendation" | N/A — 这是合法输入 |
| Step 2 | 用户的 Agent 模式选择与项目复杂度明显不匹配（如 15 模块选单 Agent） | 提醒风险: "Single-agent mode with 15 modules may be very slow. Are you sure?" | 尊重用户选择，但在 prompt 中添加备注建议 |
| Step 3 | 某个模块在任务清单中有 subtask 但在设计文档中缺少对应的设计 section | 在 prompt 中该模块标注 `⚠️ 设计文档缺失: [缺少的内容]` | 如果 ≥3 个模块有缺失，暂停并告知用户先完善设计文档 |
| Step 3 | 设计文档中的接口与任务清单中的文件路径不一致 | 以设计文档为准（设计文档是权威），在 prompt 中统一 | 如果用户后续发现不一致，迭代修正 |
| Step 4 | 用户给出模糊反馈（如 "还行但感觉不对"） | 追问具体点: "Which sections need changes — the agent workflow, module specs, constraints, or something else?" | 逐节确认：列出 prompt 的所有主要章节，让用户指出不满意的 |
| Step 5 | 用户要求修改一个影响多个模块的约束 | 追踪所有受影响章节，更新后汇报影响范围 | "This change affects: Module C (§ contract), Module D (§ requirements), §6.2 (security constraints)" |

## Writing Guidelines

- **Write for an AI agent, not a human.** The reader is an LLM that needs unambiguous,
  verifiable instructions. Prefer checklists and concrete specifications over prose.
- **Every "MUST" must be verifiable.** If you say "tests MUST pass", specify how:
  `pnpm run test` exits 0 with coverage ≥80%.
- **Module specs should be copy-pasteable into a sub-agent prompt.** Each module's
  §4 section should contain everything the sub-agent needs to start working.
- **Don't duplicate the design doc; reference it.** The prompt is the execution
  roadmap; the design doc is the blueprint. The prompt tells the agent WHERE to
  look for details, not what every detail is.
- **Number and cross-reference everything.** Module dependencies reference other
  module sections. Constraints reference PRD requirement IDs. Every cross-reference
  reduces ambiguity.
- **The orchestrator's workflow is the most important section.** If the main agent
  doesn't know what to do, nothing gets built. Be precise about scheduling order,
  parallelization rules, and acceptance gates.
