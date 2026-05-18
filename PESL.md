---
name: pre-execution-simulation-layer
description: >
  Activate before writing code, fixing bugs, calling tools, or any action with
  side effects. Forces structured simulation and reasoning before acting.
  Includes confidence-driven model routing to minimize token cost.
  Compatible with Claude Code, Codex, Deepseek, OpenHands, SWE-agent, Cursor,
  and any LLM agent supporting system prompt injection.
---

# Pre-Execution Simulation Layer (PESL)

> Simulate first. Route smart. Learn always.

Most agent failures are not knowledge failures — they are structure failures.
The agent acts before reasoning. PESL fixes this with three lightweight layers:

- Phase 1 — Pre-Action Simulation: reason before writing code or calling tools
- Phase 2 — Pre-Fix Simulation: find root cause before patching errors
- Phase 3 — Confidence Routing: use the cheapest model that can do the job

---

## When to activate

| Trigger | Phase |
|---|---|
| New task received | 1A Planning |
| About to write non-trivial code | 1B Code |
| About to call tool / shell / API | 1C Tool |
| Compile or runtime error received | 2A Fix |
| Tool result is unexpected | 2B Result |
| Any task dispatch | 3A Confidence |
| Task completed | 3C Log |

Skip all phases for: trivial reads, Q&A, single-line edits with no side effects.

---

## Phase 1 — Pre-Action Simulation

### 1A. Planning (before any code)

```
PLAN
goal: [what to achieve]
input: [given data/context]
output: [expected result]
approach: [chosen strategy and why]
risks: [top 2-3 predicted problems with likelihood]
unknowns: [anything unresolved — stop if critical]
steps: [ordered list]
```

### 1B. Code Simulation (before writing complex logic)

```
CODE-SIM [function/module name]
input: [example value]
trace:
  L[N]: [what happens] -> [state]
  L[N+1]: [what happens] -> [state]
  L[N+X]: WARN [predicted issue and why]
output: [predicted result]
compile-risks: [predicted type/ownership/syntax errors]
runtime-risks: [race conditions, nulls, panics]
fix-before-writing: [mitigations to apply now]
```

### 1C. Tool Call Simulation (before any tool / shell / API)

Mandatory for irreversible actions. Required for all others.

```
TOOL-SIM [tool name] [params]
does: [what this tool will do to the system]
affects: [files / DB / state changed]
output: [predicted return value]
risks:
  - [risk] | [severity: H/M/L] | reversible: [Y/N]
depends-on: [required prior state]
next-depends-on: [what downstream steps need from this]
preconditions:
  - [ ] [must be true before calling]
decision: [CALL / HOLD: reason / ABORT: reason]
```

---

## Phase 2 — Pre-Fix Simulation

### 2A. Pre-Fix (before patching any error)

Rule: read the error, then set it aside. Simulate the code mentally first.

```
FIX-SIM
error: [paste error]

trace:
  L[N]: [what happens] -> [state]
  L[N+X]: ROOT CAUSE — [explain why this line is the actual problem]

root-cause: [real cause, not the line compiler complained about]
symptom-line: [line compiler flagged]
cause-line: [line where problem actually originates]

verify: prediction=[...] error-says=[...] match=[Y/N/partial]

wrong-fix: [common patch that treats symptom only]
right-fix: [fix that addresses root cause]
changes:
  - [file:line] [what to change]
side-effects: [could this fix break anything else?]
```

### 2B. Tool Result Reasoning (when output is unexpected)

```
RESULT-SIM
expected: [predicted output]
actual: [real output]
delta: [difference]
cause: [which assumption or precondition was wrong]
impact:
  - step [X]: [needs adjustment / can proceed]
adjusted-plan: [revised next steps]
decision: [CONTINUE / RETRY / ABORT]
```

---

## Phase 3 — Confidence-Driven Model Routing

### 3A. Confidence Check (before every task dispatch)

```
CONFIDENCE
task: [one line]
seen-before: [Y/N/few]
past-success: [Y/N/mixed/unknown]
well-defined: [Y/N]
needs-large-context: [Y/N]
irreversible-if-wrong: [Y/N]
history-tag: [pattern tag if logged before, else "none"]
past-fail-rate: [X% or unknown]

score: [0-100]%
reason: [one line]

route:
  >85% -> small   (haiku / gpt-4o-mini / deepseek-chat)
  40-85% -> medium (sonnet / gpt-4o)
  <40%  -> large  (opus / o3 / deepseek-reasoner)
  override-large: [irreversible action / no history / new domain]

selected: [model name]
```

### 3B. Default Routing Table (no history available)

| Task pattern | Confidence | Tier |
|---|---|---|
| File check, path verify | 95% | small |
| Simple one-line fix | 90% | small |
| Known API call | 85% | small |
| New function, familiar domain | 70% | medium |
| Refactor | 65% | medium |
| Architecture decision | 50% | medium |
| Async / concurrency bug | 35% | large |
| Unfamiliar language | 30% | large |
| Root cause in 500+ line file | 25% | large |
| Irreversible tool call | forced | large |
| First time this task type | forced | large |

### 3C. Outcome Log (after every task)

```
LOG
task: [one line]
tag: [pattern — e.g. rust-borrow, async-deadlock, file-write]
model: [name] | tier: [small/medium/large]
confidence-predicted: [X%]
result: [success-first / success-after-N / escalated / failed]
tier-correct: [Y / should-be-larger / should-be-smaller]
lesson: [one line]
new-baseline: [updated confidence % for this tag]
```

### Multi-Agent Dispatch

When spawning sub-agents, route each sub-task independently:

```
DISPATCH
sub-tasks:
  - [task A] | confidence: 90% | model: small  | parallel: Y
  - [task B] | confidence: 55% | model: medium | parallel: Y
  - [task C] | confidence: 20% | model: large  | depends-on: B

token-estimate: [A+B+C total] vs [all-large total]
savings: [~X%]
```

---

## Full Pipeline

```
task arrives
  -> [3A] confidence check    -> select model
  -> [1A] planning            (if complex)
  -> [1B] code simulation     (if writing code)
  -> [1C] tool simulation     (if calling tool)
  -> execute
  -> error?     -> [2A] fix simulation   -> fix
  -> bad result? -> [2B] result reasoning -> adjust
  -> [3C] outcome log         -> update history
done
```

---

## Operating Rules

Do:
- Write simulation blocks before acting — never keep reasoning internal
- Simulate before every destructive or irreversible action without exception
- Find root cause before fixing — never patch the symptom line
- Log every outcome to improve future routing

Do not:
- Read error then immediately patch — simulate first
- Call destructive tools without completing 1C
- Use large models by default — check confidence first
- Write simulation blocks longer than needed — if a block exceeds 20 lines, split the task

---

## Integration

System prompt injection:
```
Before writing code, calling tools, or fixing errors:
1. Run the appropriate PESL simulation block
2. Complete all fields
3. Then act
Never skip. Never keep reasoning internal.
```

CLAUDE.md entry:
```
## Agent Rules
Follow PESL for all non-trivial actions.
Simulate before acting. Log after completing.
```

Compatible with: Claude Code (CLAUDE.md), Codex (custom instructions),
OpenHands (agent config), SWE-agent (task template), Cursor (.cursorrules),
Deepseek and any local LLM agent (system prompt). No fine-tuning required.
