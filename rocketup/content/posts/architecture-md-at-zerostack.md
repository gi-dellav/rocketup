+++
title = 'ARCHITECTURE.md @ zerostack'
date = 2026-06-01T11:00:00+01:00
draft = true
+++

*tldr* A single Markdown file that front-loads architectural knowledge so agents don't have to discover it from scratch

## The problem

Every time an AI agent encounters a new codebase, it starts blind. It reads `AGENTS.md` for conventions, then begins probing files one by one. This works, but it costs tokens and turns as the agent builds a mental model from scratch. And every new session repeats the process.

We needed a way to front-load architectural understanding — so the agent enters the conversation already knowing how the project is organized, what the key types are, and where the control flow lives.

## The solution: ARCHITECTURE.md

`ARCHITECTURE.md` is an optional file that gives both the main agent and subagents high-level design context about your project. It sits alongside `AGENTS.md` in the system prompt preamble:

```
AGENTS.md content     → "how to work in this codebase"
ARCHITECTURE.md       → "how this codebase is built"
Custom prompt content → "what to do right now"
```

All three are concatenated into the system prompt. Every LLM call carries awareness of your project's architecture.

## Discovery and loading

zerostack discovers `ARCHITECTURE.md` using the same recursive upward search as `AGENTS.md`:

1. **Global**: `~/.local/share/zerostack/agent/ARCHITECTURE.md` (XDG data dir)
2. **Project**: `ARCHITECTURE.md` in CWD and all parent directories up to root

Files from all levels are concatenated with source-path headers, letting you define organization-wide conventions globally while keeping project-specific architecture per repo.

If no `ARCHITECTURE.md` is found, zerostack offers to create one:

```
No ARCHITECTURE.md found in /home/you/project. Create one? [y/N]
```

Accept, and a template is written with sections for directory layout, key types, control flow, data flow, design decisions, dependencies, and entry points. A system message then instructs the agent to explore the codebase and populate the file.

## What it enables

Without `ARCHITECTURE.md`, an agent spends its early turns probing files, building a mental model, and often misunderstanding design intent. With it, the agent enters the conversation knowing:

| Aspect | Benefit |
|---|---|
| **Directory layout** | Knows where to find things without guessing |
| **Key types/traits** | Understands data structures and relationships |
| **Control flow** | Knows the request lifecycle and async boundaries |
| **Data flow** | Understands how data enters, transforms, and exits |
| **Design decisions** | Why X instead of Y — prevents re-litigating tradeoffs |
| **Dependencies** | Which libraries do what |
| **Entry points** | Where the application boots |

## How subagents use it

When the `task` tool spawns exploration subagents, they receive the same architecture context. This means a subagent tasked with "find where MCP tools are registered" already knows the project layout and can navigate directly to the right directory — no hand-holding from the main agent required.

The `task` tool's system prompt explicitly instructs subagents to read `ARCHITECTURE.md` first before starting their investigation.

## Writing a good one

Aim for 200-500 words for small projects, 500-2000 for larger ones. Focus on structure, relationships, and rationale — don't reproduce code. The recommended sections are:

1. **Directory Layout** — one-line summaries of each top-level directory
2. **Key Types/Traits** — the 5-10 most important data structures
3. **Control Flow** — request lifecycle, main loops, async boundaries
4. **Data Flow** — how data enters, transforms, and exits
5. **Design Decisions** — "why X instead of Y" for critical choices
6. **Dependencies** — key libraries and what they're used for
7. **Entry Points** — binary entry, API handlers, CLI parsing

## The comparison with AGENTS.md

`AGENTS.md` tells the agent how to operate (coding style, conventions, procedures). `ARCHITECTURE.md` tells it what it's operating on (structure, relationships, design intent). They're complementary — both are loaded into the preamble, both are auto-discovered, both can be suppressed with `--no-context-files`.

The key difference: `ARCHITECTURE.md` changes when the codebase changes (refactors, new modules), while `AGENTS.md` changes when conventions change. The agent is prompted to update `ARCHITECTURE.md` when making significant structural changes, keeping it alive as living documentation.

---

Follow us [on Github](https://github.com/gi-dellav/zerostack/). ARCHITECTURE.md ships in v1.4.0.
