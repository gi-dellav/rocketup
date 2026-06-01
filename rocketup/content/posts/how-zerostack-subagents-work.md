+++
title = 'Subagents design @ zerostack'
date = 2026-06-01T10:00:00+01:00
draft = false
+++

*tldr* Parallel read-only child agents with simple text input-output

## The problem

When an agent needs to understand a codebase, it has two choices: start probing files one by one (slow, burns tokens) or dump everything into context (impossible for large projects). Either way, the main agent's context fills up with exploration noise, making it worse at its actual job — writing code.

We needed a way to parallelize research without polluting the primary conversation.

## The design

Subagents are **read-only child agents** spawned by the main agent via a `task` tool. Each subagent receives a precise technical question and returns a focused answer. If multiple prompts are given, they run in **parallel** via `tokio::spawn`.

```
Main Agent                          Subagent(s)
┌──────────────┐                   ┌─────────────────────┐
│ read/write   │                   │ read                │
│ edit/bash    │  calls "task"     │ grep                │
│ task ────────│ ─────────────────→│ find_files          │
│              │   with prompt(s)  │ list_dir            │
│              │   spawns parallel │ todo                │
│              │   tokio tasks     │ memory_read*        │
│              │   returns findings│ memory_search*      │
└──────────────┘                   └─────────────────────┘
```

Key constraints:
- **Read-only by design** — no `write`, `edit`, `bash`, `memory_write`, or `mcp_tool`
- **No permission system** — all tools run with `permission: None` (safe because they can only read)
- **Opt-in feature** — gated behind `subagents` Cargo feature

This means the worst a subagent can do is read files, which is exactly what it's designed to do.

## Parallel execution

When you pass multiple prompts to the `task` tool, each spawns its own async task. `futures::future::join_all` gathers results. A failed subagent (panic or error) does not cancel the others — its output shows the error while the rest complete normally. Results are ordered by the original prompt index.

This is where the architecture shines for large refactors: instead of "find the auth module, then find the API routes, then find the database layer" in sequence, the agent fires all three probes simultaneously.

## Configuration

| Field | Default | Description |
|---|---|---|
| `task_max_turns` | `15` | Max agent turns per subagent |
| `task_enabled` | `true` | Whether the `task` tool is registered |
| `subagent_model` | `deepseek-v4-flash` | Model name or quick-model alias |
| `subagent_provider` | (same as main) | Provider for the subagent |

Subagents can use a different model/provider than the main agent, with a separate API client created at startup. Switch at runtime via `/model-subagent` or `/models-subagent`.

## How it stays safe

The subagent is built with no permission system on its tools. This sounds scary until you realize every tool is read-only:
- `read`, `grep`, `find_files`, `list_dir` — these cannot modify state
- `memory_read`, `memory_search` — read-only memory access
- No `write`, `edit`, `bash`, `memory_write`, or `mcp_tool`

The main agent's `task` tool still goes through the normal permission check (`check_perm("task", …)`), so users can allow/ask/deny it via their `opencode.json`.

## What it enables

Subagents let the main agent delegate deep exploration without context bloat. A single turn can spawn 3-5 parallel investigations, each returning structured findings that the main agent incorporates into a coherent plan. This is especially powerful when combined with `ARCHITECTURE.md` — subagents receive the architecture context automatically, so they start exploring with design awareness, not from zero.

---

Follow us [on Github](https://github.com/gi-dellav/zerostack/). Subagents ship in v1.4.0.
