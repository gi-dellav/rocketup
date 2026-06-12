+++
title = 'Custom Prompts: zerostack alternative to Skills'
date = 2026-06-02T10:30:00+01:00
draft = false
+++

*tldr* Skills are over-engineered; a markdown file that redefines the system prompt + an agent integration to switch prompt does the same thing.

## The problem

While designing [zerostack](https://gi-dellav.github.io/zerostack/) (a lightweight TUI coding agent written in Rust), I needed to implement Skills, as they are common in most mainstream coding agents, but while coding them up I realized how Skills go against the minimal design of zerostack: most of the time, a skill is just a system prompt with extra rules. The overhead of parsing manifests, resolving dependencies, allowing for external scripts and merging tool permissions, exists because the format demands it, not necessarly because the agent needs it.

We wanted something that a human can read, write, and version-control in 30 seconds, while being way closer to how Skills work in the agent implementation and how users want to change how the agent itself behaves based on the given task.

## The design

Our solution are Prompts: each Prompt is just a single Markdown file that, when selected, overwrites the agent's system prompt.

All Prompts are stored in `~/.local/share/zerostack/prompts` (on Linux), and the content of this directory is indexed at startup time by zerostack.

In zerostack, you switch between Prompts with `/prompt <name>` or `.<name>` (dot shortcut); when selected, the agent rebuilds its context instantly, allowing to cache the content of the Prompt itself (cheaper inference on cloud providers).

In order to allow for slash-based usage like it's typical with Skills, we implemented one-shot prompts via `.<name> <syntax>`, which changes the Prompt, executes your command, and goes back to the previous state.

## Mode directives

Zerostack takes pride in having a great design for our permission system (a detailed blogpost on this topic will come out in the next days), reason why we had to allow Prompts to manage the Permission Mode in which it operates, allowing for protected execution (aka read-only tools) for Prompts like `plan` or `ask`.

The only "metadata" a prompt needs is an optional mode directive on line 1:

```
%%mode=readonly
You are a read-only code analyst...
```

```
%%mode=last_user_mode
You are a coding assistant...
```

That single line tells zerostack which permission mode to activate when the prompt loads.

For example, `%%mode=readonly` locks the agent out of write-based operation.

A special instruction is `%%mode=last_user_mode`, which restores whatever the user last picked; this is used for prompts independent from whatever Mode is currently being used, allowing for the user to manually choose what Mode it should run on (the `Standard` mode is designed to act as close as possible to Claude Code's permissions).

## Built-in prompts

If we don't support Skills, we must build an integrated ecosystem that removes (most of the time) the need of Skills, reason why we built 14 prompts heavily inspired by [superpowers](https://github.com/obra/superpowers/), each with a clear job:

| Prompt | Mode | What it does |
|---|---|---|
| `code` | `last_user_mode` | Default coding with TDD workflow |
| `plan` | `planwrite` | Architecture planning, no code |
| `review` | `last_user_mode` | Code review for correctness and design |
| `debug` | `last_user_mode` | Root-cause debugging |
| `ask` | `readonly` | Read-only codebase questions |
| `brainstorm` | `last_user_mode` | Design exploration |
| `frontend-design` | `last_user_mode` | Production-grade UI |
| `review-security` | `last_user_mode` | Security audit |
| `simplify` | `last_user_mode` | Refinement and simplification |
| `write-prompt` | `last_user_mode` | Prompt authoring |
| `write-text` | `last_user_mode` | Writing and editing prose (ex. email, posts, etc) |
| `refactor` | `last_user_mode` | Refactoring |
| `autoconfig` | `last_user_mode` | Interactive zerostack configuration |

## Custom prompts

Prompts make building custom instructions even easier: just write a `.md` file in `~/.local/share/zerostack/prompts/`; not only that, we ship a Prompt called `write-prompt`, which is an equivalent to Claude's `skill-creator`.

## Autoconfig

The, in my opinion, most interesting Prompt we ship is `autoconfig`, which is designed to read zerostack's documentation (at `~/.local/share/zerostack/docs/`) in order to allow for the customization of zerostack just by talking to the agent itself, similar to how Pi can be customized by allowing Pi itself to build extensions.

(*note*: zerostack doesn't ship with a system as powerful as Pi's extensions, but in the current stage, `autoconfig`+`write-prompt` should be good enough to solve ~90% of the customization needs of traditional users)

## What it enables

A user can install zerostack and find the entire ecosystem that can expect from complete Skill kits, powered by a user-focused implementation with prompts that are easy to customize in order to readapt to the workflow, with experimental possibility (to be shipped in v1.5.0, currently in beta) to have simple yes/no UIs for switching from one Prompt to another in order to build custom chains of prompts (from `brainstorm` to `plan`, from `plan` to `code`) for accelerated development.

The entire prompts system (indexing, loading, switching, mode directives) sits at under 200 lines of Rust, showcasing how simpler code can beat overengineered systems.

## Why is this useful for other agents

We think that not only is this simpler for other agents to implement when compared to Skills, they allow for minimal designs and they allow to import into another agent the 14 prompts we already built, shipping more powerful and more customizable agents.

---

Thanks for reading!

---

*p.s.* Check out [zerostack on Github](https://github.com/gi-dellav/zerostack/).
