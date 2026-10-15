---
description: Try several commands in parallel forks of one kosa8 sandbox and keep the one that works
argument-hint: <sandbox> -- <command 1> ;; <command 2> [;; ...]
---

Arguments: $ARGUMENTS

Parse the sandbox name before `--` and the candidate commands after it,
separated by `;;`. There must be at least two; if not, ask for more.

Call `sandbox_explore` with that sandbox and the commands as variants. Each
variant runs in its own fork of the sandbox, resumed with its running state
intact. Report a table of the variants: succeeded or failed, time taken, and
the key output. Say which branch was kept. Note that forks count toward the
plan's concurrency, and that a refusal for that reason names the limit.
