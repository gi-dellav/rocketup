+++
title = 'Building a fast TUI in Rust @ zerostack'
date = 2026-06-02T11:00:00+01:00
draft = true
+++

*tldr* No frameworks. 0% CPU idle. ~16 MB RAM. A custom TUI built from scratch on crossterm.

## The problem

Most terminal coding agents are built on JavaScript. They sit at 150–700 MB of RAM at idle. Their TUIs use React (Ink), which runs a virtual DOM in a Node.js process inside your terminal. CPU spins even when nothing is happening.

We wanted zerostack to run on a Raspberry Pi. On a cloud instance with 512 MB of RAM. Alongside 3 other parallel agents without swapping. That meant building the TUI ourselves.

## The stack

| Layer | Choice | Why |
|---|---|---|
| Terminal | `crossterm` 0.29 | Cross-platform, raw mode, no rendering opinions |
| Async | `tokio` | Single-threaded runtime, 0% CPU at idle |
| Markdown | `pulldown-cmark` | Streaming parser, no allocations for inline text |
| Memory | `mimalloc` | Drop-in `malloc` replacement, ~10% smaller heap |
| Strings | `compact_str` + `smallvec` | Inline strings up to 24 bytes, small vecs on the stack |

We explicitly did **not** use `ratatui` (the Rust TUI framework). We needed pixel-level control over rendering, a custom line buffer, and zero-cost integration with our event system. Frameworks optimize for general-purpose apps. We optimized for one app.

## The renderer

The heart of the TUI is a `Vec<LineEntry>` — a flat array of `(text, Color)` pairs. No widgets. No layout engine. No component tree.

```
Renderer
┌──────────────────────────────────┐
│ Vec<LineEntry>   ← scrollable    │
│ [(text, color), ...]             │
│                                  │
│ viewport_start   ← top of visible│
│ viewport_height  ← terminal rows │
│ scroll_offset    ← how far up    │
└──────────────────────────────────┘
```

Each agent event appends to this buffer. Tokens stream in → markdown is parsed incrementally → styled lines are pushed. The viewport scrolls. No diffs. No reconciliation. Just append-and-render.

Word wrapping handles CJK characters correctly (via `unicode-width`). The scroll indicator shows a percentage when you've scrolled up. Background colors are configurable per area (chat, input, status bar).

## The event loop

A single `tokio::select!` with four branches:

```rust
loop {
    tokio::select! {
        event = user_event_rx.recv() => { /* keyboard, resize, paste */ }
        event = agent_event_rx.recv() => { /* tokens, tool calls, errors */ }
        request = ask_rx.recv()        => { /* permission prompts */ }
        _ = tick.tick()                => { /* 100ms spinner refresh */ }
    }
}
```

No threads for I/O — a background task polls `crossterm` every 50ms and sends `UserEvent`s through an `mpsc` channel. The agent streams events through another channel. Permission requests use a third. Everything flows into one loop.

When the agent is idle, `tokio::select!` parks the thread. CPU drops to 0.0%. The terminal is still responsive — keypresses wake the loop instantly.

This is what "0% CPU idle" actually means: not a low-power mode, not a sleep timer — the async runtime literally has nothing scheduled.

## The input editor

No `rustyline`. No `reedline`. A custom text editor built on raw `crossterm` key events:

| Key | Action |
|---|---|
| Enter | Send message |
| Shift+Enter | Newline |
| Ctrl+W | Delete word backwards |
| Ctrl+U | Delete to line start |
| Ctrl+A/E | Home/End |
| Ctrl+B/F | Back/Forward (Emacs) |
| Ctrl+P/N | Previous/Next (Emacs) |
| Ctrl+K | Kill to end of line |
| Ctrl+Y | Yank (paste kill ring) |
| Ctrl+G | Open in `$EDITOR` |
| Ctrl+H | Launch `lazygit` |
| Tab | File picker / autocomplete |
| Up/Down | Command history |

The editor maintains a kill ring, cursor position, selection state, and history — all in ~700 lines. It supports multi-line input, horizontal scrolling for long lines, and clipboard integration via OSC-52 sequences with platform fallbacks (wl-copy, xclip, pbcopy, clip.exe).

## Slash commands

Every `/command` is a match arm in a dispatcher. No plugin system. No dynamic loading. Just a function call:

| Category | Commands |
|---|---|
| Session | `/clear`, `/undo`, `/retry`, `/quit`, `/sessions`, `/history` |
| Provider | `/provider`, `/model`, `/models`, `/models-add` |
| Context | `/add`, `/drop`, `/drop-all`, `/prompt`, `/theme` |
| Security | `/mode` (5 modes) |
| Conversation | `/compress`, `/compact`, `/editsys`, `/btw`, `/reasoning` |
| Features | `/loop`, `/worktree`, `/memory`, `/mcp` |

`/btw` deserves a special mention: it forks the current context (including in-flight agent turns), spawns a read-only agent to answer a side question, and prints the answer inline — without touching the main conversation history. Token cost is tracked separately as `btw:$0.003`.

## Pickers

Three picker types, each a function that renders a modal and returns a selection:

- **File picker**: Triggered by `@` in the input. Walks the filesystem, filters on each keystroke, shows matches.
- **Model picker**: Quick models + live model catalog. Switch models in two keystrokes.
- **List picker**: Generic selection for prompts, themes, providers.

No framework. Each picker is ~100 lines of rendering logic.

## Memory numbers

These are measured, not estimated:

| Metric | zerostack | Typical JS agent |
|---|---|---|
| Binary size | 26 MB | 200–400 MB (node_modules) |
| RAM (idle) | ~16 MB | ~150–700 MB |
| RAM (peak) | ~24 MB | ~400–1200 MB |
| CPU (idle) | 0.0% | 1–5% |
| CPU (under load) | ~1.5% | 5–15% |
| LoC | ~16,000 | ~75,000–150,000 |

The binary is a single statically-linked executable. No runtime. No package manager. Copy it to a server and run it.

## What it enables

You can run zerostack alongside your LSP, your database, your dev server, and 3 other parallel agents — on a machine with 512 MB of RAM. You can deploy it to a cloud instance and agent-manage worktrees without swapping. You can use it on a Raspberry Pi as a lightweight coding companion.

The TUI isn't a product decision. It's a performance decision. Every line of rendering code exists because the alternative — importing a framework — would cost more RAM than our entire agent.

The entire TUI sits at ~4,800 lines of Rust.

---

Follow us [on Github](https://github.com/gi-dellav/zerostack/).
