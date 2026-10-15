---
description: Run a task inside a sealed kosa8 microVM instead of on the host
argument-hint: <what to run or try>
---

Do this task inside a kosa8 sandbox, not on the host: $ARGUMENTS

1. Call `kosa8_status`. If kosa8 is not running, stop and tell the user; never
   fall back to running the task on the host.
2. Create a sandbox with `sandbox_create` (a short, descriptive name). Share
   host folders only if the user allowed them when the server started; if a
   share is refused, tell the user which directory to allow.
3. If the task needs network access to an API, ask the user to create the
   sandbox from their terminal with an allow-list instead, for example
   `kosa8 sandbox create sealed --egress-allow api.example.com`, and wait.
   Never put a real API key into the sandbox or a tool argument.
4. Call `sandbox_checkpoint` with a label before any step that would be hard to
   undo.
5. Run the work with `sandbox_pull` and `sandbox_run`. If a step goes wrong,
   `sandbox_rewind` to the checkpoint rather than repairing by hand.
6. Report what ran, the output that matters, and the sandbox name. Ask before
   deleting the sandbox.
