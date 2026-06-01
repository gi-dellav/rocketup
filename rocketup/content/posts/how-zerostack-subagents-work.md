+++
title = 'Subagents design @ zerostack'
date = 2026-06-01T10:00:00+01:00
draft = false
+++

*tldr* Parallel read-only child agents with simple text input-output

## The problem

When an agent needs to understand a codebase, it has two choices: start probing files one by one (slow, burns tokens) or dump everything into context (impossible for large projects). Either way, the main agent's context fills up with exploration noise, decreasing coding performance.

Following most mainstream coding agents, we decided to build subagents, but we needed to design subagents that follow the UNIX-like minimal philosphy of [zerostack](https://gi-dellav.github.io/zerostack/).


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
- **Opt-in feature** — gated behind `subagents` Cargo feature (enabled by default)

This makes sure that no dangerous actions can be taken by a subagent, and than no condition race issues can happen between subagents, without needing to implement file locking.

## Parallel execution

When you pass multiple prompts to the `task` tool, each spawns its own async task. `futures::future::join_all` gathers results. A failed subagent (panic or error) shows its error to the agent while the rest complete normally, allowing the agent to then either re-send the subagent or to complete the task with its own tools (ex. the task sent to the subagents required using the `git` bash command). Results are ordered by the original prompt index.

The idea shown in the system prompts that ship with zerostack is that subagents are to be treated by the main agents as subordinates that take high-level questions (ex. `How does Authentication work?`), and spit out low-level answers (ex. `In src/auth.rs, there is AuthManager, etc...`).

## Configuration

| Field | Default | Description |
|---|---|---|
| `task_max_turns` | `15` | Max agent turns per subagent |
| `task_enabled` | `true` | Whether the `task` tool is registered |
| `subagent_model` | `deepseek-v4-flash` | Model name or quick-model alias |
| `subagent_provider` | (same as main) | Provider for the subagent |

Subagents can use a different model/provider than the main agent, with a separate API client created at startup, allowing for cheaper and faster models being used for codebase exploration (via subagent), while heavy models execute the real coding tasks.

## Prompts

In our system prompt we:
- describe Subagents with: `- **Subagent use:** The task tool spawns new LLM instances — it is slow and expensive. Use ONLY when answering requires searching 3+ distinct files and cross-referencing their contents (e.g. \"Where is MCP support implemented?\"). Do NOT use for: listing a directory, grepping one pattern, reading one known file, or any single-step operation. Call those tools directly instead. Do NOT use for wide/vague tasks (\"explore the codebase\"). If you already ran a subagent and got results, use those results — do not re-spawn.`
- describe the `task` tool with: `- **task**: Delegate a MULTI-STEP read-only investigation to a subagent. Use ONLY when answering needs several file reads and cross-referencing. NOT for single operations (list_dir, grep, read a known file). Multiple prompts run in parallel. Subagent has read, grep, find_files, list_dir, memory access. Returns findings.`

## What it enables

Subagents let the main agent delegate deep exploration without context bloat. A single turn can spawn 3-5 parallel investigations, each returning structured findings that the main agent incorporates into a coherent plan. This is especially powerful when combined with `ARCHITECTURE.md` — subagents receive the architecture context automatically, so they start exploring with design awareness, not from zero.

Together with our constrained read operations and faster I/O implementations, we managed to build a simpler and faster approach to subagents, that manage to resolve what opencode's subagent do with a 25% gain in code exploration time.

The entire implementation sits at 414 lines of code (written in Rust).

---

Follow us [on Github](https://github.com/gi-dellav/zerostack/).
