+++
title = 'What we built in 2 weeks @ zerostack'
date = 2026-15-31T18:56:49+01:00
draft = false
+++

*tldr* Ship fast + break and repair + interact with the community

## What is zerostack?

[zerostack](https://gi-dellav.github.io/zerostack/) is a minimal coding agent written in Rust, optimized for memory footprint (~20MB of RAM, optimized for low-end devices, parallel agents and cloud platforms) and performance (0% CPU usage when idle), inspired by the UNIX philosophy.

*zerostack* ships different innovative features to try to compete with the more mainstream CLI coding agents:
- A better, redesigned permission system split in 5 modes (Restrictive, ReadOnly, Guarded, Standard and YOLO), integrated with prompts and with sandboxing
- Integrated Ralph Wiggum loops via `/loop`
- Integrated Git Worktrees with auto-merge support and `--parallel` for agent-managed worktrees
- Introduced ARCHITECTURE.md, an hybrid between AGENTS.md and Memory that acts as a 1-file core reference for all AI agents
- Shipping with the Custom Prompts, an alternative to Skills, and the Prompts Library, our alternative to [superpowers](https://github.com/obra/superpowers/)

*zerostack v1.0.0* was released on the 17th of May, and it was met by positive feedback from Hacker News: today, we want to show what was build in the last two weeks.

## What did we do in 14 days?

In chronlogical order:
- Added `max_agent_turns` to control the agent's turn
- Added support for local models via Ollama and empty API keys
- Added ACP support
- Added ARM64 builds
- Added terminal resizing logic
- Added chat history with arrowkeys support
- Added Ctrl+G to edit the user message in an external editor
- Added Quick Models via `/models`
- Added themes support
- Redesigned authentication logic
- Added Emacs-style keybindings
- Implemented lazy loading for all memory-heavy elements
- Added support for using .TOML as config files
- Added support for ~/.config for storing config files
- Added cost tracking
- Added `/prompt autoconfig`, an integrated agent for configuring zerostack
- Implemented `--parallel` and other worktree-based CLI flags for flexible workflows
- Implemented an experimental edit tool based on [this antirez's article](https://antirez.com/news/166)
- Fix CRLF bug
- Fix path traversal bug
- Implemented safer permission checker
- Added `/btw` and `/undo`, inspired by Claude Code
- Added ability for Prompts to set what Permission Mode it wants to run in
- Added `!` to run slash commands
- Added `.` to quickly switch between prompts
- Implemented Memory support (*only in preview releases*)
- Implemented Subagents support (*only in preview releases*)
- Implemented ARCHITECTURE.md support (*only in preview releases*)

## What do we want to show?

We wanted to show that the new paradigm in the AI agents era is to build fast, ship faster, and keep listening to your community: non-trivial issues and pull requests should always be resolved in less than 24 hours, and even when you are not building you have to keep thinking on how you can improve your projects in order to adapt to the userbase.

Also, I wanted to write a short note on the amount of lines often shipped in vibe coded tools: even if you want to ship lots of features, you still need to focus on clean code, even if the codebase if fully agent-built, in order to improve human reviews and agent performances: our codebase is currently ~16k LoC, while other equivalent agents sit at around ~75k to ~150k LoC.

## What is coming next?

We are going to ship:
- v1.4.0 w/ Memory and Subagent
- VSCode, Zed and JetBrains extensions
- Hooks via JSON RPC and via shell scripts
- Workflows & Macros
- Agent Interfaces via UNIX sockets
- Optimized prompts

Follow us [on Github](https://github.com/gi-dellav/zerostack/) in order to recieve updates from us.

---

Happy building,
Giuseppe Della Vedova @ zerostack
