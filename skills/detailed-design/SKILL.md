---
name: detailed-design
description: 'Generate a structured Detailed Design Document from a PRD with module-level technical specs including data structures, service interfaces, algorithms, error handling, and testing strategy for each module. Trigger: 详细设计, 技术设计, 架构设计, 设计文档, 帮我做详细设计, 基于PRD生成设计, 生成技术方案, technical design, architecture design, design doc.'
---

# Detailed Design Generation Skill

## Core Principle — DO NOT GUESS

**This is the most important rule of this skill. Every technical decision
must be either confirmed by the user or clearly labeled as a recommendation.**

The PRD tells you WHAT to build. The detailed design tells you HOW to build it.
Between them is a gap: the PRD says "use JWT" but not which algorithm; it says
"React SPA" but not which router; it says "rate limiting" but not which
algorithm or what limits. **You must not fill these gaps silently.**

- Every technical choice not specified in the PRD is a **gap**. Gaps must be
  surfaced as questions, not filled by assumption.
- If the PRD says "建议 X" or "推荐 Y", that is the PRD author's suggestion —
  it still needs user confirmation.
- If the user says "you recommend", that is the only case where you may propose
  a technical choice — but label it as your recommendation with reasoning.
- When writing the design doc, every design decision must be traceable to either:
  (a) an explicit statement in the PRD, (b) a question you asked and they answered,
  or (c) a recommendation you explicitly marked as yours.

**Examples of guessing (DON'T):**
- PRD says "JWT" → you assume RS256 without asking
- PRD says "paginated API" → you assume cursor-based without asking
- PRD says "migration support" → you assume auto-migrate-on-start without asking
- PRD says "error responses" → you assume `{error, message}` without asking

**What to do instead:**
- "The PRD mentions JWT — should we use symmetric (HS256) or asymmetric (Ed25519/RS256) signing?"
- "For API pagination: offset-based (page/limit) or cursor-based? Each has trade-offs for the admin UI."
- "Should database migrations run automatically on container startup, or be triggered manually?"
- "Which error response format: RFC 7807 Problem Details, JSON:API, or a simple custom format?"

## Prerequisites

The user must have a PRD (or equivalent requirements document) before this skill
can be used. If they don't:

- Point them to the PRD skill first: "I need a requirements document to work from.
  Let's create a PRD first — would you like to use the PRD skill?"
- If they insist on skipping the PRD, treat their verbal description as the
  requirements source and **be even more vigilant about gaps** — you're now
  missing both product requirements AND technical decisions.

## Process

### Step 0: Verify the input

Check that `doc/prd.md` exists and is readable. If the user points to a different
file, use that. **Read the PRD thoroughly** — you cannot design what you don't
understand.

If the PRD is incomplete (missing sections, vague requirements), tell the user
what's missing and ask if they want to refine the PRD first. Do not design
around holes in the requirements.

### Step 1: Extract the module list and identify gaps

From the PRD, extract:
1. **Feature areas** — these become your module candidates
2. **Non-functional requirements** — these become cross-cutting modules (security, logging, deployment)
3. **Technical choices already made** — e.g., "Fastify", "PostgreSQL", "React + shadcn/ui"
4. **Gaps** — technical decisions the PRD is silent on

🔴 **CHECKPOINT · 🛑 STOP** — Present to the user:

A structured gap analysis showing:
- **Confirmed** (from PRD): what's already decided
- **Open Questions** (PRD Section 7 + gaps you found): organized by domain
- **Module Breakdown** (proposed): how you plan to decompose the system

Format:
```
## Confirmed Decisions (from PRD)
| Area | Decision | Source |
|------|----------|--------|
| Runtime | Node.js 20+ | PRD §4.1 |
| Framework | Fastify | PRD §4.1 |
| ... | ... | ... |

## Open Questions
### From PRD (§7)
- [ ] Trial duration? (7 or 14 days — PRD suggests 7)

### Design Gaps I Found
- [ ] Session management: JWT cookie or server-side session?
- [ ] Error format: RFC 7807 or custom?
- ...

## Proposed Module Breakdown
1. Module A: ... (depends on: ...)
2. Module B: ... (depends on: Module A)
...

Does this look right? Ready to proceed to Q&A?
```

Wait for user confirmation before proceeding.

### Step 2: Multi-round Q&A — fill technical gaps

Use `AskUserQuestion` to elicit technical decisions. **Max 4 questions per round.**

**How to structure the rounds:**

| Round | Focus | Typical Topics |
|-------|-------|---------------|
| Round 1 | PRD Open Questions | Trial duration, grace period, unconfirmed PRD items |
| Round 2 | Architecture & Infrastructure | Session strategy, migration policy, error/pagination format |
| Round 3 | Module Details | Data storage strategy, key format, feature flags vs quotas |
| Round 4 | Remaining Gaps | Anything still unclear after rounds 1–3 |

**How to pick questions for each round:**
- Start with the PRD's own "Open Questions" section (§7). Answer those first.
- Then address architectural-level gaps (auth, data storage, API design).
- Then module-specific details (key formats, fingerprint strategy, feature flags).
- Each question must offer 2–4 concrete options with trade-offs explained.
- Always include "Other" as an escape hatch.
- For technical recommendations, provide a "(Recommended)" label with reasoning
  in the description.

**Question design guidelines:**
- Don't ask "What algorithm should we use?" — instead, present 2–3 viable options
  with their trade-offs and let the user choose.
- If one option is clearly better for the user's stated constraints, mark it as
  recommended and explain why.
- Never present an option you wouldn't actually recommend implementing.
- If the user's choice would conflict with a PRD requirement, flag it immediately.

🔴 **CHECKPOINT · 🛑 STOP** — After the final Q&A round, present a summary table
of all confirmed technical decisions before writing the document. Format:

```
## All Confirmed Decisions
| # | Decision | Choice | Source |
|----|----------|--------|--------|
| 1 | Trial duration | 7 days | User Q1R1 |
| 2 | Offline grace period | 7 days | User Q2R1 |
| 3 | Key format | UUID-style 8-4-4-4-12 | User Q2R3 |
| ... | ... | ... | ... |

Does everything look correct before I write the detailed design document?
```

### Step 3: Design each module

For each module in the breakdown, write the detailed design. Every module section
must include:

#### Required Sections for Each Module

1. **Overview** — 2–3 sentences on what this module does and why it exists.
2. **Data Structures** — Database tables (DDL + Drizzle schema), TypeScript interfaces,
   type definitions. Every field must have a purpose.
3. **API / Service Interface** — The public contract. For service modules, the
   TypeScript interface. For API endpoints, the method/path/description/auth table.
4. **Core Logic / Algorithms** — Pseudocode or flowcharts for non-trivial logic
   (state machines, validation flows, offline grace decisions, key generation).
5. **Business Rules** — Constraints and validation rules that aren't obvious from
   the data model (e.g., "TRIAL cannot be renewed", "prefix is 2-8 uppercase
   alphanumeric").
6. **Error Handling** — Specific error cases this module produces, with HTTP
   status codes and RFC 7807 type URIs.
7. **Configuration** — What environment variables or config keys this module reads.
8. **Testing Strategy** — How to test this module in isolation (what to mock) and
   what integration tests it needs.
9. **Dependencies** — Explicit list of what other modules this module depends on.
   If a module has zero dependencies beyond the database layer, say so.

#### Module Independence Rule

**Every module must be testable in isolation.** This means:

- Dependencies are injected via constructor parameters (not imported as singletons).
- Every service exposes an interface (`IServiceName`) so callers can mock it.
- Database access is the only hard dependency — everything else is swappable.

```typescript
// Correct: dependencies injected
class LicenseService implements ILicenseService {
  constructor(
    private db: Database,
    private productService: IProductService,  // interface, not concrete class
    private auditService: IAuditService,
  ) {}
}

// Wrong: hard import dependency
import { productService } from '../product/product.service';
class LicenseService {
  // can't mock productService in unit tests
}
```

#### Design Depth

- **Database tables**: Full DDL with column types, constraints, indexes.
- **API endpoints**: Method, path, auth requirements, request/response shape.
- **Algorithms**: Enough detail that a developer could implement without guessing.
- **Config**: Every configurable value listed with default, env var name, and unit.

🔴 **CHECKPOINT · 🛑 STOP** — After designing all modules but before writing the full document,
present a module-level design summary to the user:

```
## Module Design Summary (before writing document)

| Module | Sections Complete | Dependencies | Open TODOs |
|--------|-------------------|-------------|------------|
| A: Database | 9/9 | None | 0 |
| B: Product | 9/9 | A | 0 |
| C: License | 9/9 | A, B, H | 1 (features schema confirmation) |
| ... | ... | ... | ... |

Key decisions to confirm:
- [List any design choices made during module design that the user hasn't seen yet]
- [List any ⚠️ TODO markers and whether they block the document]

Does each module look correct? Any modules you want to split, merge, or redesign before I write the full document?
```

Wait for user confirmation before writing. If the user requests changes to module design,
apply them and re-present the summary. Once confirmed, proceed to Step 4.

### Step 4: Write the document

Create or overwrite `doc/detailed-designed.md`.

**Document structure:**

```
# Project Name — Detailed Design Document

> Version, date, status, based-on PRD version

## 1. Architecture Overview
- Layered architecture diagram (ASCII)
- Module dependency graph (ASCII)
- Module independence / testing strategy overview

## 2–N. Module A, B, C... (one per module)
Each following the 9-section template from Step 3.

## Appendix: Configuration Reference
- Every environment variable, default, description in one table
- Project directory structure (tree)
```

**Writing rules:**
- Be specific. "SHA-256 hash" not "hashed". "Ed25519 via jose library" not "JWT signing".
- Every table column, every API field, every config key must be documented.
- Use TypeScript interface syntax for data structures (consistent, readable).
- ASCII diagrams for flows, state machines, and architecture.
- Link between modules: if Module C depends on Module B's interface, reference it.

### Step 5: Deliver and prompt for review

Summarize:
- Total modules designed
- Key architectural decisions made
- Any remaining open questions (if any)
- What the user should focus on during review

🔴 **CHECKPOINT · 🛑 STOP** — Explicitly ask: "Please review the detailed design
and let me know if anything needs to change — modules you want to split/merge,
design decisions you disagree with, sections that need more/less detail."

### Step 6: Iterate if needed

The user may want to refine the design — splitting a module, adding detail to a
section, changing a technical decision. Make targeted edits and bump the version
number. When a design decision changes, check if it affects other modules (e.g.,
changing key format affects both License and Activation modules).

## Anti-Patterns — Things You Must NOT Do

Every anti-pattern below is a real failure mode observed in design document generation.
Check against this list before writing each module section.

| # | 反模式 | 为什么不要做 | 替代做法 |
|---|--------|-------------|---------|
| 1 | **静默填补技术缺口** — PRD 没说用什么算法/格式/策略，设计者自行选定但不告知用户 | 用户不知道你替他做了决定，后续发现时已晚。设计文档变成了「你的设计」而非「用户的设计」 | 每个 PRD 未指定的技术决策必须通过 Q&A 确认，或标注 "Recommended" |
| 2 | **设计无接口的模块** — 模块之间通过 import 具体类而非 interface 耦合 | 无法独立测试；换一个模块的实现会级联破坏其他模块 | 每个 Service 暴露 interface，依赖通过构造函数注入；见 Module Independence Rule |
| 3 | **跳过依赖声明** — 模块设计写了 9 个 section 但没有列出依赖 | 开发者不知道模块启动需要哪些前置条件，集成时才发现循环依赖 | 每个模块末尾必须有 "Dependencies" section，列出所有依赖模块及用途 |
| 4 | **设计不可测的模块** — 模块逻辑与数据库/文件系统/网络调用硬编码在一起 | 单元测试无法 mock；只能写昂贵的集成测试 → 覆盖率不达标 | 所有外部依赖注入；测试 strategy section 必须说明 mock 什么 |
| 5 | **省略类型和配置** — 接口只写方法名和参数名，没有类型定义 | 开发者自行解读类型 → 实现不一致 → 集成时类型冲突 | 所有接口用 TypeScript 语法定义完整类型；所有配置项列出 env var 名+默认值+单位 |
| 6 | **范围蔓延** — 根据「最佳实践」添加 PRD 未提及的功能（如 PRD 没说 Webhook，设计者加了 Webhook 模块） | 增加不必要的复杂度，浪费时间，用户不想要 | 设计只覆盖 PRD 明确要求的范围；额外想法放入文档末尾的 "Future Considerations" |
| 7 | **模糊描述** — "the system handles errors gracefully" / "appropriate security measures are applied" | 不可执行，不可验证，不可测试。给开发者留下的空白比答案还多 | 每个错误场景必须定义：触发条件、HTTP 状态码、RFC 7807 type URI、用户看到的消息 |
| 8 | **无错误契约的模块** — 模块只描述 happy path，没有定义它会产生哪些错误 | 调用方不知道要处理什么异常；集成时错误传播失控 | 每个模块必须有 "Error Handling" section，列出所有错误类型 |
| 9 | **把 PRD 建议当决策** — PRD 写 "建议 7 天"，设计者直接当 7 天用而不确认 | PRD 的建议是作者的偏好，用户可能不同意。设计文档中的决策必须有明确来源 | 所有 PRD 中的 "建议" 必须通过 Q&A 确认或标注 "per PRD suggestion, pending confirmation" |
| 10 | **过度设计模块** — 一个模块写了 2000 行设计，覆盖了 3 年后才需要的扩展点 | 核心设计淹没在噪声中；评审者找不到重点；实现者无从下手 | 设计文档只覆盖 Phase 1-2 的实现；扩展点用 1-2 句话标注 "Extension point: [方向]" |
| 11 | **模块拆分过细或过粗** — 5 行代码的 util 独立成模块，或整个业务逻辑塞进一个模块 | 过细：接口膨胀、依赖图混乱；过粗：无法独立测试、无法并行开发 | 一个模块 = 一个可独立部署/测试的职责单元；如果不确定，按 PRD 的功能区域拆分 |

## Failure Modes & Fallbacks

If something goes wrong during the design process, do NOT silently skip or guess.
For each failure: try the first-line fix; if that still fails, use the fallback.

| Step | 触发条件 | 一线修复 | 仍失败兜底 |
|------|---------|----------|-----------|
| Step 0 | `doc/prd.md` 不存在 | 检查用户是否指定了其他路径: "I don't see a PRD. Do you have requirements elsewhere?" | 引导用户先用 PRD skill: "Let's create a PRD first — would you like to use the PRD skill?" |
| Step 0 | PRD 缺少核心章节（无功能需求/无非功能需求/无技术选型） | 列出缺失章节: "The PRD is missing [sections]. Can you provide these, or should we define them now?" | 将缺失章节标为 `⚠️ PENDING` 在设计文档中，基于已有信息继续；不编造需求 |
| Step 1 | 无法从 PRD 中提取自然的模块边界 | 按 PRD 的功能区域提议模块划分，主动询问: "Does this decomposition make sense?" | 请用户描述他们期望的系统分解方式；调整后重新展示 |
| Step 1 | 用户拒绝了模块划分方案 | 追问拒绝原因: "Which part doesn't work — the boundaries, the granularity, or the grouping?" | 根据反馈重划模块，最多重试 2 次；第 3 次仍拒绝则逐模块询问用户意图 |
| Step 2 | 用户跳过问题或回复 "不确定" | 标记为开放设计决策，记录在文档的 Open Questions 中 | 如果该决策阻塞了多个模块的设计，在后一轮 Q&A 中换一个角度重新提问 |
| Step 2 | 用户答案与 PRD 明确要求矛盾 | 立即标记矛盾: "The PRD says X, but you're choosing Y. Which one should the design follow?" | 以用户最新回答为准，在文档中标注 "偏离 PRD §X: [说明]" |
| Step 2 | 用户选择 "你推荐" | 给出最佳选项 + 2 句以内的推荐理由（基于用户已确认的约束） | 标注 "Recommended"；用户仍可拒绝 |
| Step 3 | 某个模块的 section 信息不足 | 在该 section 插入 `⚠️ TODO: [需要什么信息]` 标记，继续设计其他 section | 如果 ≥3 个 section 都是 TODO，暂停该模块，先完成其他模块后回头补齐 |
| Step 3 | 模块设计过程中发现 PRD 未覆盖的需求 | 立即记录: "While designing [module], I found [gap] — this wasn't in the PRD." | 将该 gap 加入文档的 Open Questions；不自行决定需求 |
| Step 4 | 某个模块不需要设计模板中的某个 section | 跳过该 section，标注: "N/A — [原因，如: 纯逻辑模块无 API 端点]" | 确保至少 5/9 个 section 有实质内容；否则该模块可能需要合并 |
| Step 5 | 用户给出模糊反馈（如 "还行但感觉不太对"） | 追问具体点: "Which modules or sections need changes? Is it the level of detail, a design choice, or the structure?" | 逐模块确认：列出所有模块，让用户指出具体哪个不满意 |
| Step 6 | 设计变更影响多个模块 | 编辑前追踪所有依赖链，更新每个受影响模块 | 在响应中列出完整的影响范围清单: "This change affects: Module A (§X), Module C (§Y)" |

## Writing Guidelines

- **TypeScript over pseudocode.** Real type definitions are more precise and
  directly usable for implementation.
- **Show the "why".** For non-obvious design choices, add a one-sentence rationale.
- **Number everything.** Database columns are in DDL; API endpoints are in tables;
  config keys are in the appendix.
- **Link modules.** "See Module D §5.5.1" beats repeating the validate flow.
- **Error codes are part of the design.** Every module defines its error contract.
- **Testing strategy is not optional.** If you can't explain how to test a module
  in isolation, the module isn't well-designed.
- **Adapt depth to complexity.** A simple CRUD module needs less detail than a
  cryptographic validation module. Spend words where the complexity is.

## Module Design Template

Use this as a checklist when writing each module section:

```
## Module X: [Name]

### X.1 Overview
[2–3 sentences: what, why, where it fits]

### X.2 Data Structures
- Database tables (SQL DDL + Drizzle schema)
- TypeScript interfaces / types
- Validation rules for each field

### X.3 Service Interface
- TypeScript interface for the service
- Each method: parameters, return type, side effects

### X.4 Core Logic / Algorithms
- Flow diagrams (ASCII) for complex operations
- Pseudocode for algorithms
- State machines with all valid transitions

### X.5 Business Rules
- Constraints
- Validation rules
- Edge cases handled

### X.6 API Endpoints (if applicable)
| Method | Path | Description | Auth |

### X.7 Error Handling
| HTTP | type URI | Description |

### X.8 Configuration
| Config | Default | Env Variable | Description |

### X.9 Testing Strategy
- Unit tests (what to mock)
- Integration tests (what to verify)

### X.10 Dependencies
- Module A (Database) — for persistence
- Module B (Product) — for product validation
```
