---
name: prd
description: "Generate a structured Product Requirements Document (PRD) by eliciting requirements from the user through multi-round Q&A. Use when user asks to create a PRD, write a requirements document, draft a product spec, or define what to build. Also trigger when user says \"需求文档\", \"产品需求\", \"规格说明\", \"帮我写PRD\", \"生成PRD\", \"产品需求文档\", \"写一个PRD\", \"requirements document\", \"product spec\", \"before we start coding\", or any time they want to nail down requirements before implementation."
---

# PRD Generation Skill

## Core Principle — DO NOT GUESS

**This is the most important rule of this skill. Violating it defeats the
entire purpose of the PRD process.**

- Every design decision, technical choice, or feature detail that the user has
  not explicitly stated is a **gap**. Gaps must be surfaced as questions, not
  filled in silently.
- If the user gives an ambiguous answer, **ask for clarification**. Do not
  interpret it in the way that seems most likely.
- If the user says "I'm not sure" or "you recommend", that is the only case
  where you may propose an answer — but you must still **label it as your
  recommendation** and explain why, so the user can reject it.
- When writing the PRD, every claim that did not come directly from the user's
  answers must be traceable to either: (a) a question you asked and they
  answered, or (b) a recommendation you explicitly marked as yours.

**Examples of guessing (DON'T):**
- User says "a web app" → you assume React SPA without asking
- User says "needs login" → you assume JWT + OAuth without asking
- User says "database" → you assume PostgreSQL without asking

**What to do instead:**
- "You said a web app — do you have a frontend framework preference, or should I recommend one?"
- "For login, do you prefer traditional email/password, OAuth/SSO, or both?"
- "Any database preference? PostgreSQL, MySQL, SQLite, or should I recommend?"

## Process

### Step 1: Understand the project at a high level

Ask the user to describe their project in their own words. If they've already
given a description, **paraphrase it back and confirm** before asking any
structured questions.

At this point, note down everything the user has explicitly stated — and
everything they haven't. The blank spots are your question material.

🔴 **CHECKPOINT · 🛑 STOP** — Before proceeding to Q&A rounds, show the user
your understanding (paraphrase) and the gaps you've identified. Ask: "Does this
sound right? Anything I missed?" Only proceed after the user confirms.

### Step 2: Multi-round Q&A — fill the gaps

Use `AskUserQuestion` to elicit requirements. **Max 4 questions per round.**

**How to pick questions:**
- Look at what the user has NOT said yet. Those are your question topics.
- Group related questions into rounds by domain (business → technical → details).
- Each question must offer concrete options plus a "you recommend" or "other" escape hatch.
- Adapt question topics to the project. An API server needs auth and rate-limiting
  questions; a desktop app needs offline and auto-update questions; a static site
  doesn't need database questions at all.

**Typical question domains (use only what applies):**

| Domain | Example topics |
|--------|---------------|
| Product & Users | Product type, target users, use context |
| Business Model | Monetization, license model, subscription tiers |
| Core Features | Feature scope, MVP vs full, integrations |
| Architecture | Service shape (API/SDK/monolith), offline needs |
| Tech Stack | Language, framework, database, deployment target |
| Security | Auth method, device binding, encryption requirements |
| Admin & Ops | Admin UI, stats/monitoring, API docs, CI/CD |

**During each round:**
- Read the user's previous answers carefully. Don't re-ask something they already told you.
- If an answer opens a new gap, queue a follow-up for the next round.
- If the user picks "you recommend", mark it for a recommendation in the PRD.
- **Stop when you have enough clarity to write without guessing.** Usually 3–4 rounds.

🔴 **CHECKPOINT · 🛑 STOP** — After the final Q&A round, present a summary
table of all confirmed decisions before writing the PRD. Ask: "Here's what I've
gathered. Does everything look correct before I write the document?" Wait for
the user to confirm or correct any misunderstandings.

### Step 3: Write the PRD

Create `doc/` if it doesn't exist, then write `doc/prd.md`.

The document must include:

1. **Overview** — Background, core goals. Sourced from the user's own words.
2. **Functional Requirements** — Numbered (F-XXX-NN), organized by feature area.
   Each requirement is something the user confirmed they need.
3. **Non-Functional Requirements** — Organized in sub-sections:
   - Security (NFR-SEC-XX)
   - Performance (NFR-PERF-XX)
   - Deployment & Operations (NFR-DEP-XX)
   - Maintainability (NFR-MNT-XX)
4. **Technical Architecture** — A high-level diagram (ASCII) and a technology
   stack table. For every choice, cite the source: user's answer, or "Recommended"
   with reasoning.
5. **Data Model** (if applicable) — Entity relationship diagram (ASCII).
6. **Development Phases** — Staged from core → production → polish.
7. **Open Questions** — A checklist of decisions the user still needs to make.
   For each, provide a suggested default in parentheses (e.g., "Trial duration? (suggest: 14 days)"),
   but make it clear these are suggestions, not decisions.

**Key rule for writing:** Before adding any requirement to the document, ask
yourself: "Did the user say this, or am I assuming it?" If it's an assumption
and you didn't ask, it goes in Open Questions, not in the requirements.

### Step 4: Deliver and prompt for review

Summarize the key decisions captured in the PRD. Point out:
- What the user explicitly chose
- What you recommended on their behalf (and why)
- What's still open and needs a decision

Ask the user to review and mark anything that needs changing.

🔴 **CHECKPOINT · 🛑 STOP** — Explicitly ask: "Please review the PRD and let
me know if anything needs to change — requirements I misunderstood, sections
you want to add/remove, or open questions you can answer now." Do not proceed
to iteration until the user gives feedback.

### Step 5: Iterate if needed

The user may want to refine the PRD after review — e.g., adding security measures
discovered during threat modeling, or removing items that don't fit their setup.
Make targeted edits and bump the version number.

## Anti-Patterns — Things You Must NOT Do

- ❌ Filling in technical details the user never mentioned
- ❌ Assuming "standard" or "obvious" choices without asking
- ❌ Writing a PRD full of decisions the user didn't make
- ❌ Skipping questions because "the user probably wants X"
- ❌ Treating the user's silence on a topic as agreement with a default

## Failure Modes & Fallbacks

If something goes wrong during the PRD process, do NOT silently skip or guess.
Follow the fallback for each step:

| Step | Failure | Fallback |
|------|---------|----------|
| Step 1 | User's description is too vague to paraphrase | Ask: "Can you tell me more about what problem this solves and who will use it?" |
| Step 1 | User rejects your paraphrase | Ask what specifically was wrong, re-paraphrase, confirm again |
| Step 2 | User skips a question or says "not sure" | Mark it as an open question; do NOT fill in a default silently |
| Step 2 | User's answer contradicts an earlier answer | Point out the contradiction: "Earlier you said X, now you're saying Y — which one reflects your current thinking?" |
| Step 2 | User picks "you recommend" | Note it for a recommendation in the PRD, labelled as yours |
| Step 3 | Not enough information to write a section | Put it in Open Questions; do NOT fabricate requirements |
| Step 3 | A required section doesn't apply to this project | Skip it with a brief note: "N/A — [reason]" |
| Step 4 | User gives vague feedback ("looks good but...") | Ask for specifics: "Which sections need changes? Can you point to what feels off?" |
| Step 5 | Iteration introduces contradictions | Before each edit, re-read the full PRD to ensure consistency |

## Writing Guidelines

- **Be specific.** "JWT signed with Ed25519, private key never leaves the server"
  beats "the system should be secure."
- **Number everything.** Every functional and non-functional requirement gets a
  unique ID for traceability.
- **Show your sources.** For each technical choice, the rationale should make it
  clear whether the user chose it or you recommended it.
- **Open questions are not failures.** They're evidence that you didn't guess.
  A PRD with open questions is more honest and useful than one full of unchecked
  assumptions.
- **Adapt the depth.** A CLI tool needs less detail than a multi-service platform.
