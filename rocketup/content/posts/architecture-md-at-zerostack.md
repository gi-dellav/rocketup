+++
title = 'ARCHITECTURE.md @ zerostack'
date = 2026-06-02T10:00:00+01:00
draft = true
+++

*tldr* One file. Every agent reads it. Zero context wasted on rediscovery.

## The problem

You start a new coding session. The agent doesn't know your project. It pokes around — reads a file, greps a pattern, reads another file. Ten turns later, it finally understands the layout. You pay for every one of those turns.

Next session? Same thing. The agent has amnesia.

`AGENTS.md` tells the agent *how* to operate. But nobody tells it *what* it's operating on. That gap is expensive.

## The design

`ARCHITECTURE.md` is a single Markdown file that lives in your project root. It describes your codebase at the design level — directory layout, key types, control flow, data flow, design decisions, dependencies, entry points.

On startup, zerostack walks up from your CWD collecting every `ARCHITECTURE.md` it finds (project-level, monorepo-level, global-level), concatenates them with source-path headers, and injects the whole thing into the system prompt **before** the agent ever thinks.

```
System prompt = reasoning prefix 
              + AGENTS.md (how to operate)
              + ARCHITECTURE.md (what to operate on)  ← injected here
              + custom prompt
              + CWD + memory + extra files
```

Every turn. Every subagent. Architecture awareness is not opt-in — it's ambient.

## How it gets created

First time you run zerostack in a project without one, it asks: *"No ARCHITECTURE.md found. Create one? [y/N]"*

Say yes, and it writes a template with seven sections:

| Section | What goes in it |
|---|---|
| Directory Layout | Where things live at a glance |
| Key Types/Traits | The important structs, enums, traits |
| Control Flow | How execution moves through the system |
| Data Flow | How data transforms from input to output |
| Design Decisions | *Why* things are the way they are |
| Dependencies | Major crates/packages and their roles |
| Entry Points | CLI, API, main loops — where it starts |

The agent then auto-fills it by exploring your codebase. You review, tweak, commit. Done.

## What it changes

Without `ARCHITECTURE.md`, a subagent exploring "how does auth work?" starts from a blank slate — it discovers the directory structure, guesses at module boundaries, and wastes turns orienting itself.

With `ARCHITECTURE.md`, it starts already knowing that auth lives in `src/auth.rs`, uses `AuthResolver` with a 4-tier key resolution chain, and was designed to support custom providers. Its first grep goes straight to the right file.

The main agent benefits too. It doesn't need to "explore the codebase" before every non-trivial task. It already knows the shape of the project. First-turn accuracy goes up. Wasted exploration turns go down.

## Configuration

| Field | Default | Description |
|---|---|---|
| `no_context_files` | `false` | Skip all context files (AGENTS.md + ARCHITECTURE.md) |
| (implicit) | — | Walks from CWD to `/`, collects every `ARCHITECTURE.md` |

Global `ARCHITECTURE.md` lives at `~/.local/share/zerostack/agent/ARCHITECTURE.md`. Use it for conventions that span all your projects.

## Architecture vs. Agents

| | `AGENTS.md` | `ARCHITECTURE.md` |
|---|---|---|
| Tells the agent | How to behave | What it's working on |
| Scope | Project conventions | Project design |
| Changes | Rarely | With every major refactor |
| Length | 50–200 words | 200–2000 words |

They complement each other. One without the other is half the picture.

## What it enables

`ARCHITECTURE.md` turns project-specific knowledge into a persistent asset. Every agent — main agent, subagents, even future sessions — starts with design awareness. No warm-up cost. No rediscovery tax.

Combined with subagents, it's especially powerful: 3 parallel subagents, each starting with full architecture context, return structured findings in a single turn that would otherwise take 10+ sequential exploration steps.

The entire implementation sits at ~180 lines of Rust, plus the template generation logic.

---

Follow us [on Github](https://github.com/gi-dellav/zerostack/).
