---
name: task-generator
description: Generate minimum executable task checklists for each module from a Detailed Design Document and PRD. Each task names a specific output file and describes exactly what goes in it, ordered by dependency. Trigger: 任务拆分, 给我拆任务, 生成任务清单, 拆分开发任务, 把设计文档拆成任务, task breakdown, create task lists, implementation tasks.
---

# Task Generator Skill

## Core Principle — Minimum Executable Unit

**Every task must be small enough to complete in one focused session, concrete
enough that a developer knows exactly what file to create and what it must do,
and independently verifiable (has a clear "done" condition).**

A task is NOT "Implement License module" — that's a module. A task IS "Create
the License Key generator function at `src/modules/license/key-generator.ts` that
outputs `<PREFIX>-<TYPE>-<UUID-STYLE>` format using `crypto.randomBytes(16)`."

- Each task names a specific output file and describes exactly what goes in it.
- Each task includes validation criteria (what test to write, what constraint to check).
- Tasks are ordered within each module by dependency: foundational tasks first,
  dependent tasks later.
- No task should take more than ~2 hours for a developer familiar with the stack.
  If a task looks bigger, split it further.

## Prerequisites

This skill requires a Detailed Design Document at `doc/detailed-designed.md` (or
equivalent). It also reads `doc/prd.md` for context on phases and priorities.

If these documents don't exist:
- Point the user to the PRD skill (`/prd`) and Detailed Design skill (`/detailed-design`)
  first. Without a design document, task generation is guesswork.
- If the user insists on skipping the design, treat their verbal description as
  the design source, but **warn that task accuracy will be significantly lower**
  and more tasks will need rework.

🔴 **CHECKPOINT · 🛑 STOP** — Verify both `doc/prd.md` and `doc/detailed-designed.md`
exist and are readable. If either is missing, tell the user which one and offer to
invoke the corresponding skill. Do not proceed without the design document.

## Process

### Step 0: Read and understand the inputs

1. Read `doc/prd.md` thoroughly. Extract:
   - Development phases and their priorities
   - Feature areas and non-functional requirements
   - Technology stack decisions (these determine file paths and tool choices)

2. Read `doc/detailed-designed.md` thoroughly. Extract:
   - All modules (usually labeled Module A, B, C...)
   - For each module: data structures, service interfaces, API endpoints, business
     rules, error handling, configuration, dependencies, testing strategy
   - The module dependency graph

3. Cross-reference: the design should cover all PRD functional requirements.
   If a requirement has no corresponding module section, flag it.

🔴 **CHECKPOINT · 🛑 STOP** — Before generating tasks, present a summary to the user:

```
## Input Summary

**From PRD:**
- Development phases: [e.g., Phase 1-6]
- Technology: [stack summary]
- NFRs: [count] security, [count] performance, [count] deploy

**From Detailed Design:**
- Total modules: [N] (Module A through Module X)
- Module list with task count estimates:
  | Module | Name | Est. Tasks | Dependencies |
  |--------|------|-----------|-------------|
  | A | ... | ~XX | None |
  | B | ... | ~XX | A |
  | ... | ... | ... | ... |

**Questions before I generate:**
- Should I follow the design's implied phase ordering, or do you have a different priority?
- Any modules you want me to skip or de-prioritize?
- Preferred output directory? (default: `doc/tasks/`)

Shall I proceed with task generation?
```

Wait for user confirmation. If they want to adjust module scope or ordering,
apply those changes before generating.

### Step 1: Generate per-module task files

For each module in the design document, create `doc/tasks/module-<letter>.md`.

#### Task Breakdown Rules

**How to decompose a module into tasks:**

1. **Data structures first.** If the module defines database tables, types, or
   interfaces, those come first — everything else depends on them.

2. **Core logic next.** The service implementation: methods, algorithms, business
   rules. Break by method or by logical group (e.g., "CRUD operations" vs
   "lifecycle operations").

3. **API routes after logic.** Route handlers depend on the service being
   implemented.

4. **Integration points.** If the module integrates with other modules
   (e.g., audit logging in license operations), add specific tasks for those
   integration touch points.

5. **Tests last but never optional.** Every module must have unit test tasks
   AND integration test tasks. These are separate tasks.

#### Task Granularity Rules

| Too big (split further) | Right size | Too small (merge) |
|-------------------------|------------|-------------------|
| "Implement LicenseService" | "Implement LicenseService.issue() method" | "Add import statement for crypto" |
| "Create all API routes" | "Create GET /api/v1/admin/licenses route" | "Add a console.log" |
| "Write tests" | "Write unit tests for Key generator: format correctness, collision check" | "Write one test case" |

**Heuristic**: a task should produce exactly **one file** or **one complete method
with its supporting types**. If a task touches 3+ files, it's probably too big.

#### Each task item MUST contain:

```
- [ ] **<Task ID>** — <One-line description of what to do>
  - File: `<exact file path relative to project root>`
  - What to implement: <concrete deliverable — function name, interface, algorithm>
  - Validation: <how to know it's done — test case, manual check, constraint>
```

Exception: tasks that are purely config/setup (e.g., "configure drizzle-kit")
may omit the file path if they genuinely touch multiple config files.

#### Task file template

Each `module-<letter>.md` must follow this structure:

```markdown
# Module X: [Module Name]

> 基于: [详细设计 §N](../detailed-designed.md#N-module-...)
> 依赖: <list modules this one depends on>
> Phase: <development phase number>

---

## 任务清单

### X1. <Task Group 1 Name>

- [ ] **X1.1** — <Task description>
  - File: `<path>`
  - <details>

- [ ] **X1.2** — <Task description>
  ...

### X2. <Task Group 2 Name>
...

---

## 完成标准

- [ ] <condition 1>
- [ ] <condition 2>
...

---

## 产出文件

<tree of files this module creates>
```

#### Error handling in task design

Each task file must include error-handling tasks for its module:
- Define error codes (RFC 7807 type URIs) as a separate task or as part of the
  route task
- If the design specifies error codes, include them explicitly in the task description
- If the design is silent on specific error codes, add a task to "define and
  implement error responses for [module]"

🔴 **CHECKPOINT · 🛑 STOP** — After generating the **first module's** task file, show it
to the user as a sample before generating the remaining modules:

```
## Sample Task File: Module X

[Show the generated module-x.md content]

Does this format, granularity, and level of detail look right?
I'll apply the same pattern to the remaining [N-1] modules.
Any adjustments before I continue?
```

Wait for user confirmation before generating the remaining module files.
This prevents having to reformat all files if the template needs adjustment.

### Step 2: Generate progress tracker

Create `doc/tasks/progress.md` as the master dashboard.

The progress file must contain:

1. **Header** — Project name, last updated date, total module count,
   completed count (`X / N`).

2. **Phase Overview Table** — One row per development phase showing which
   modules belong to it, status (待开始/进行中/已完成), and completion %.

3. **Per-Module Checklist** — One `- [ ]` per module, grouped by phase.
   Each module entry shows:
   - Module letter and name (linked to its task file)
   - Total task count
   - Sub-group breakdown with sub-counts

4. **Dependency Graph** — ASCII art showing which modules depend on which,
   plus a suggested implementation order.

5. **Task Statistics Table** — Module count, total task count, per-phase counts.

6. **Usage Instructions** — Brief note on how to use the progress tracker
   (check off tasks as completed).

🔴 **CHECKPOINT · 🛑 STOP** — After writing `progress.md`, verify consistency before
proceeding:
- All modules are listed (cross-reference with the design document's module list)
- Task counts per module match the actual `- [ ]` counts in each `module-*.md`
- Dependency graph is correct and matches the design document
- Statistics table sums match across all categories
- Fix any discrepancies before moving to Step 3.

### Step 3: Deliver and confirm

After all files are written:

1. List every file created with its path.
2. Provide a summary table: modules, task counts, estimated effort.
3. Point out any modules that have especially high or low task density
   (may indicate design gaps or over/under-decomposition).

🔴 **CHECKPOINT · 🛑 STOP** — Ask: "Do the task breakdowns look actionable?
Any modules need finer or coarser granularity? Should I adjust any task grouping?"

### Step 4: Iterate if needed

The user may want to:
- Split a task that's too big
- Merge tasks that are too granular
- Reorder tasks within a module
- Add tasks for missing design elements discovered during review

Make targeted edits to the affected `module-*.md` file(s). If the change affects
the overall progress structure, update `progress.md` accordingly.

🔴 **CHECKPOINT · 🛑 STOP** — After completing all iterations, confirm with the user:
"All requested changes have been applied. Are the task files ready for development,
or are there any final adjustments needed?" Do not proceed until the user signals
the task files are final.

## Anti-Patterns — Things You Must NOT Do

| # | 反模式 | 为什么不要做 | 替代做法 |
|---|--------|-------------|---------|
| 1 | **任务太粗** — "实现 License 模块" 作为一个 task | 无法追踪进度；无法判断是否完成；可能耗时数天 | 拆到单个文件/单个方法级别："实现 `generateLicenseKey()` 函数" |
| 2 | **任务太细** — "import crypto 模块" 作为独立 task | 噪音淹没真正的工作项；checklist 膨胀到不可用 | 合并到包含它的逻辑任务中；import 是实现细节 |
| 3 | **跳过测试任务** — 只写实现 task 不写测试 task | 测试会被无限期推迟；质量无法保证 | 每个模块至少 1 个单元测试 task + 1 个集成测试 task |
| 4 | **模糊描述** — "实现产品管理的 CRUD" | 不知道要创建什么文件、什么方法；无法判断完成 | 列出具体方法名和文件路径："实现 `ProductService.create(input)` — 文件: `src/modules/product/product.service.ts`" |
| 5 | **忽略依赖顺序** — task 列表不按依赖排序 | 开发者先做依赖下游再做上游 → 阻塞或返工 | Schema 定义在服务实现之前；服务实现在路由注册之前；核心逻辑在集成之前 |
| 6 | **凭空添加任务** — 设计文档没提到的功能被拆成 task | 范围蔓延；用户没要求的功能被实现 | 所有 task 必须可追溯到设计文档的某个 section |
| 7 | **漏掉集成点** — License 模块的 task 不包含审计日志集成 | 模块间契约未实现；审计追踪断裂 | 如果设计文档说 "Module C 依赖 Module H"，那么 Module C 的 task 中必须有集成审计日志的 task |
| 8 | **忽略配置和错误处理** — task 只覆盖 happy path | 生产就绪所需的边界逻辑缺失 | 每个模块至少 1 个错误定义 task + 配置项 task（如适用） |
| 9 | **硬编码绝对路径** — task 中的文件路径写死 Windows 或 Linux 格式 | 跨平台开发者无法使用 | 使用正斜杠的相对路径：`src/modules/license/key-generator.ts` |
| 10 | **没有 Done 条件** — task 只有标题没有验证标准 | 开发者声称 "做完了" 但可能遗漏关键约束 | 每个 task 必须有 Validation 行：要跑通的测试、要满足的约束 |

## Failure Modes & Fallbacks

| Step | 触发条件 | 一线修复 | 仍失败兜底 |
|------|---------|----------|-----------|
| Step 0 | `doc/prd.md` 或 `doc/detailed-designed.md` 不存在 | 检查用户是否指定了其他路径 | 引导用户先用对应 skill 生成缺失文档 |
| Step 0 | 设计文档缺少关键 section（如某模块无测试策略/无错误处理） | 标记缺失: "Module X is missing [sections]. I'll add a ⚠️ TODO task for it." | 将缺失内容标记为 `⚠️ PENDING DESIGN`，生成 task 但标注 "需要设计补充" |
| Step 0 | 设计文档的模块依赖关系不清晰 | 根据文档内容推导依赖图，展示给用户确认 | 按文档阅读顺序假设依赖，标注 "Inferred — please verify" |
| Step 1 | 某个模块太大（设计文档超过 500 行），不确定如何拆分 task | 按设计的 sub-section 自然拆分；将大方法拆成独立 task | 展示拆分方案给用户："Here's how I plan to split Module X — does this granularity work?" |
| Step 1 | 两个模块的 task 有循环依赖嫌疑 | 立即标记: "Module X and Y appear to have circular dependency — this may indicate a design issue." | 按设计文档的依赖声明拆分；在 progress.md 中标注 ⚠️ |
| Step 1 | 某个设计 section 只有概要没有细节（如 "测试策略: 单元测试 + 集成测试"） | 生成合理的测试 task 列表（基于模块接口推断），标注 "Inferred from interface" | 将测试 task 数量保持与模块复杂度成比例 |
| Step 2 | progress.md 的模块统计与 module-*.md 的 task 数量不一致 | 重新计数: 遍历每个 module-*.md 统计 `- [ ]` 行数 | 在 progress.md 中标注 "Approximate ~N tasks" |
| Step 3 | 用户觉得 task 太粗或太细 | 追问具体模块: "Which module's tasks feel wrong? Should I split them further or merge some?" | 对该模块重新生成，最多重试 2 次 |
| Step 4 | 用户要求添加设计文档中没有的功能 task | 提醒: "This feature isn't in the detailed design. Should I note it as a design gap, or would you like to update the design document first?" | 将额外 task 放在模块末尾，标注 `⚠️ EXTRA (not in design doc)` |

## Writing Guidelines

- **Every task starts with a verb.** "实现", "创建", "编写", "配置", "集成".
  No passive descriptions.
- **File paths are absolute from project root.** Use forward slashes: `src/db/schema/products.ts`.
- **Group tasks by dependency order.** Within a module, tasks that produce
  foundational files (types, schemas) come before tasks that consume them.
- **Test tasks name the specific test scenarios.** "编写 Key 生成器单元测试：格式正确性、随机性（连续100个无碰撞）、TYPE 编码、prefix 拼接" — not just "编写测试".
- **Reference the design document.** Each task file links back to the relevant
  design section so developers can read the full context.
- **Count tasks.** Each module file shows its task count in the header;
  progress.md aggregates all counts.
