# PESL — Pre-Execution Simulation Layer

> *Simulate first. Route smart. Learn always.*

PESL is a lightweight cognitive protocol for coding agents.

Instead of:
```
generate → compile → error → patch → retry → patch → retry
```

PESL forces:
```
simulate → evaluate → route → execute
```

**PESL is a skill file — not a framework, not a model, not a library.**
Drop `SKILL.md` into your agent's context and it works immediately.

---

## The Problem

Most coding agents are reactive. They discover mistakes through execution, not prediction.

This causes blind retries, symptom patching, token waste, and dangerous tool calls that cannot be undone.

---

## What PESL Does

Before writing code, fixing bugs, or calling tools — the agent must first simulate.

PESL adds three layers:

**Phase 1 — Pre-Action Simulation**
The agent predicts compile errors, runtime behavior, and tool side effects before acting.

**Phase 2 — Pre-Fix Simulation**
When an error occurs, the agent runs the code mentally to find the root cause — not the symptom line.

**Phase 3 — Confidence Routing**
The agent rates its own certainty, then routes to the cheapest model that can handle the task. Outcomes are logged to improve future routing.

---

## Core Insight

Markdown is not documentation. In PESL, it becomes a cognitive workspace — a lightweight world model where the agent reasons about future state before committing to execution.

Reasoning is externalized and inspectable instead of hidden in the token stream.

---

## Quick Example

**Without PESL — Rust borrow checker:**
```
let v = vec![1, 2, 3];
process(v);           // ownership moved
println!("{:?}", v);  // used after move

→ cargo build → E0382 → patch → compile → new error → retry loop
```

**With PESL:**
```
CODE-SIM process_vector
trace:
  L2: process(v) -> ownership moved into function
  L3: WARN — use-after-move predicted
compile-risks: E0382
fix-before-writing: pass &v instead of v

→ issue predicted before compilation. no retry needed.
```

**Without PESL — shell command:**
```
rm -rf ./dist → deployment fails → artifacts missing → recovery loop
```

**With PESL:**
```
TOOL-SIM rm -rf ./dist
affects: build artifacts, deployment output
risks: irreversible deletion | H | reversible: N
preconditions: [ ] new build completed? -> NOT VERIFIED
decision: HOLD — run build first, then delete

→ destructive call blocked before damage.
```

---

## Token Efficiency

PESL reduces token waste in two ways.

Fewer retries — simulating before acting eliminates most compile loops and bad tool calls.

Smarter routing — not every task needs a frontier model:

| Task | Without PESL | With PESL |
|---|---|---|
| File existence check | large model | small model |
| One-line rename | large model | small model |
| Architecture decision | large model | medium model |
| Async concurrency bug | large model | large model |
The expensive model runs only when uncertainty is genuinely high.

---

## Full Pipeline

```
task arrives
  -> [3A] confidence check   select model
  -> [1A] planning           if complex
  -> [1B] code simulation    if writing code
  -> [1C] tool simulation    if calling tool
  -> execute
  -> error?        -> [2A] fix simulation    -> fix
  -> bad result?   -> [2B] result reasoning  -> adjust
  -> [3C] outcome log        update history
done
```

---

## What PESL Is Not

PESL is not a finetuned model, a compiler, a framework, or a symbolic execution engine.

It is a cognitive execution protocol implemented entirely through prompting, structured simulation blocks, and execution feedback. No infrastructure required.

---

## Usage

**As a system prompt:**
```
Before writing code, calling tools, or fixing errors:
1. Run the appropriate PESL simulation block
2. Complete all fields
3. Then act
Never patch blindly. Never skip destructive-action simulation.
```

**As a CLAUDE.md entry:**
```
## Agent Rules
Follow PESL for all non-trivial actions.
Simulate before acting. Log after completing.
```

**Compatible with:** Claude Code, Codex, Cursor, OpenHands, SWE-agent, Deepseek, any local LLM agent. No fine-tuning required.

---

## File

| File | Purpose |
|---|---|
| `PESL.md` | The full protocol — drop into agent context |
| `README.md` | This file |

---

## Design Philosophy

The agent should not discover every mistake through execution.
It should anticipate predictable failures before they happen.

PESL does not require perfect prediction. It only needs to eliminate the most common failure paths and reduce the cost of the ones that remain.
