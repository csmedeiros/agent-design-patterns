---
name: agent-design-patterns
description: Use when you need to structure a multi-agent task (review, migration, audit, research, refactor) as a dynamic Workflow. Maps the Google Cloud agentic-AI design patterns (Sequential, Parallel, Loop, Coordinator, Hierarchical) onto Workflow primitives, points you at the matching pattern markdown, and each markdown carries a copyable Workflow template. Trigger on "which agent pattern", "orchestrate this with subagents", "fan out", "multi-agent workflow", "pipeline this", "run a workflow for".
---

# Agent design patterns → dynamic Workflows

You are the entry point. Pick the pattern that matches the task's **control-flow shape**,
open its markdown in `patterns/`, copy the template, adapt it to the concrete task, run it
via the `Workflow` tool. Do NOT invent a shape — pick from the five below.

> Prerequisite: the caller must have opted into multi-agent orchestration (said "use a
> workflow", "ultracode", named a pattern, etc.). If they haven't, describe the pattern in
> prose and ask before calling `Workflow`.

## Pick by shape

| Task shape | Pattern | Primitive | Markdown |
|---|---|---|---|
| Fixed ordered stages, each feeds the next | **Sequential** | `pipeline()` (1 item) | [patterns/sequential.md](patterns/sequential.md) |
| Independent subtasks, no shared state | **Parallel** | `parallel()` | [patterns/parallel.md](patterns/parallel.md) |
| Repeat until an exit condition / dry | **Loop** | `while` + `agent()` | [patterns/loop.md](patterns/loop.md) |
| Analyze, then route each part to a specialist | **Coordinator** | classify → `parallel()` by route | [patterns/coordinator.md](patterns/coordinator.md) |
| Big ambiguous task → decompose into subtasks recursively | **Hierarchical** | `agent(plan)` → `pipeline()` over units | [patterns/hierarchical.md](patterns/hierarchical.md) |

## How to choose when two fit

- Stages independent → **Parallel**. Stages ordered → **Sequential**. Both (per-item chain,
  many items) → `pipeline()` handles it; read Sequential.
- Unknown-size discovery (bugs, edge cases) → **Loop** (loop-until-dry), not a fixed fan-out.
- You don't know the work-list until an agent looks → **Hierarchical** (plan stage produces
  the list) or **Coordinator** (classify stage produces the routes).