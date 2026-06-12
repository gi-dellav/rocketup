+++
title = 'Permission system @ zerostack'
date = 2026-06-02T11:30:00+01:00
draft = true
+++

*tldr* 5 modes, dual-layer rules, doom-loop detection, and a 40-command bash allowlist — permissions that don't get in the way.

## The problem

Coding agents can read your files, write your code, and run shell commands. Most agents give you two choices: ask-for-everything (annoying) or yolo (terrifying). There's no middle ground that adapts to what you're actually doing.

We needed a permission system that was safe by default, configurable when needed, and fast enough that checking a permission never became the bottleneck.

## The five modes

| Mode | Reads | Writes/Edits | Bash | Config rules |
|---|---|---|---|---|
| `restrictive` | Ask | Ask | Ask | Skipped |
| `readonly` | Allow | Deny | Deny | — |
| `guarded` | Allow | Ask | Ask | Applied |
| `standard` | Allow (CWD) | Allow (CWD) | Ask (dangerous) | Applied |
| `yolo` | Allow | Allow | Ask (destructive) | Applied |

`standard` is the default. Files within your project root are auto-allowed. Files outside it prompt you. Bash commands that could be destructive (`rm`, `sudo`, `chmod`, `curl | sh`) prompt you — a 40-command allowlist with safe/unsafe defaults ships built-in.

Switch modes at any time with `/mode <name>`. Prompts can auto-set the mode via `%%mode=<name>` on line 1 — so `/prompt ask` instantly locks you into readonly, and `/prompt code` restores your last choice.

## Dual-layer rules

Permission rules come in two flavors:

**Glob patterns** (fast path, checked first):
```toml
[permission]
read = "allow"
write = "ask"
bash = "allow"          # safe commands only
external_directory = "ask"
"*" = "ask"             # default for anything not specified
```

**Regex patterns** (complex matching, checked second):
```toml
[permission-regex]
".*\.env$" = "deny"
"pip install.*" = "ask"
```

Glob runs first because it's cheap — a simple pattern match. Regex runs second for patterns that need more power. If glob returns a decision, regex is skipped. If glob says "no rule", regex gets a shot.

Both layers can be populated with the TOML-friendly flat format:
```toml
permission-allow = ["read", "write", "grep"]
permission-ask = ["bash", "external_directory"]
permission-deny = ["*.env", "*.pem"]
```

## Tool-level granularity

Every tool gets its own rules:

| Tool | What it checks |
|---|---|
| `read` | Path |
| `write` | Path |
| `edit` | Path |
| `bash` | Command string |
| `grep` | Search pattern |
| `find_files` | Glob pattern |
| `list_dir` | Path |
| `write_todo_list` | Content |
| `mcp_tool:{server}:{tool}` | Server + tool name |
| `*` | Default for unlisted tools |
| `external_directory` | Paths outside CWD |
| `doom_loop` | Repeated identical tool calls |

`external_directory` is a special key that catches any path outside your project root — add one rule instead of enumerating every possible external path.

`doom_loop` catches the Ralph Wiggum pattern: the agent calls the same tool with the same arguments 3+ times in a row. After the third repeat, zerostack warns. After the fifth, it denies and tells the agent to try something else.

## The check pipeline

```
Tool called (read, write, edit, bash, grep, ...)
        │
        ▼
┌──────────────────┐
│ 1. Session       │  "Did user already allow this pattern?"
│    allowlist     │
└──────┬───────────┘
       │ not found
       ▼
┌──────────────────┐
│ 2. Mode-based    │  restrictive → ask everything
│    defaults      │  readonly → deny writes
│                  │  standard → allow within CWD
└──────┬───────────┘
       │ no default decision
       ▼
┌──────────────────┐
│ 3. Glob rules    │  Fast pattern match
│    (permission)  │
└──────┬───────────┘
       │ no match
       ▼
┌──────────────────┐
│ 4. Regex rules   │  Slower, more powerful
│    (perm-regex)  │
└──────┬───────────┘
       │ no match
       ▼
┌──────────────────┐
│ 5. Built-in      │  bash: check safe/unsafe list
│    defaults      │  doom_loop: check repeat count
└──────┬───────────┘
       │
       ▼
   Decision: Allow / Ask / Deny
```

If the decision is `Ask`, the agent sends an `AskRequest` through an `mpsc` channel to the TUI. The user sees:

```
Allow read of /etc/config.toml? [y]es / [a]lways / [n]o
```

- **y** — allow this once
- **a** — allow always (adds pattern to session allowlist)
- **n** — deny

"Allow always" persists for the session. The pattern is added to the allowlist, and step 1 catches it on future calls. No file writes. No permanent changes. Restart zerostack and the allowlist resets.

## Prompt mode integration

Prompts can declare their required security level:

```
%%mode=readonly
You are a code analyst. You read and answer questions.
You never write code, edit files, or run commands.
```

When you switch to `ask` prompt, zerostack automatically sets `readonly` mode. The agent can't write files even if it tries — the permission checker denies it at the tool level, not the prompt level. This is defense in depth: even if the agent ignores its system prompt, the permission system won't let it through.

## Sandbox integration

Beyond the permission system, bash commands can run inside a sandbox:

```toml
sandbox = true                          # uses bubblewrap
sandbox = { backend = "bwrap" }         # explicit bubblewrap
sandbox = { backend = "zerobox" }       # custom sandbox
```

The sandbox adds a second layer: even if a bash command is allowed by permissions, it runs in an isolated environment. Write access to the filesystem is blocked at the kernel level, not the application level.

## What it enables

You can start zerostack in `standard` mode, switch to `yolo` for a refactoring sprint, then drop to `readonly` to review the results — all without restarting. You can configure per-project permission rules checked into version control. You can write a prompt that locks the agent to readonly and share it with your team, knowing nobody can accidentally override it.

The permission system isn't a gate. It's a gradient. Pick the level of trust that matches what you're doing right now.

The entire permission system — modes, dual-layer rules, doom-loop detection, session allowlist, bash command classification, sandbox integration — sits at ~900 lines of Rust.

---

Follow us [on Github](https://github.com/gi-dellav/zerostack/).
