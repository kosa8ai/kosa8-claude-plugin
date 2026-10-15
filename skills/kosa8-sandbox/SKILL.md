---
name: kosa8-sandbox
description: Use kosa8 to run untrusted, unfamiliar or agent-written code in a sealed microVM instead of on the host. Use it before running code you did not write or cannot fully read, installing unfamiliar packages, running migrations or mass deletes, trying several approaches in parallel, or any step that would be hard to undo. Covers sealed sandboxes, keeping API keys on the host with an egress allow-list, checkpoint and rewind, fork and explore, and human approvals.
---

# Working in a kosa8 sandbox

kosa8 gives an agent its own sealed machine: a real microVM with its own kernel,
not a container sharing the host's kernel. The machine sees one folder (the
workspace) and nothing else on the host. The network is closed except for hosts
on an allow-list the user writes; anything else gets a refused connection.

The `kosa8` MCP server in this plugin exposes that as tools (`sandbox_*`,
`snapshot_*`, `container_*`, `image_*`, `network_*`, `volume_*`, `kosa8_status`).

## Before anything else

Call `kosa8_status`. If the daemon is not running or `kosa8` is not installed,
tell the user and stop; do not fall back to running the risky code on the host.
Install is one line: `curl -fsSL https://kosa8.com/install.sh | sh`.

## When to use a sandbox

Prefer a sandbox over the host whenever you are about to:

- run code you did not write, or code you (or another agent) just generated and
  have not reviewed;
- install an unfamiliar package, run a downloaded script, or build a repo you
  do not know;
- run something destructive or hard to undo: a migration, a mass delete, a
  force push, a rewrite of many files;
- compare several approaches (different fixes, test commands, dependency
  versions) and keep the one that works.

For quick, read-only work on trusted code, the host is fine.

## Create a sealed sandbox

`sandbox_create` boots a microVM with its own kernel and container engine. Give
it a short `name`. Host folders can be shared only from directories the user
allowed when starting the server (`kosa8 mcp serve --allow-mount DIR` or
`DIR:rw`); shares are read-only unless the user allowed `:rw`. If a share is
refused, tell the user which directory to allow rather than looking for a way
around it.

Then use `sandbox_pull` and `sandbox_run` to run containers inside that
sandbox's private engine, isolated from the host.

## Keep keys on the host

Never paste a real API key into a sandbox, a file in the workspace, or a tool
argument. With kosa8, API keys stay on the host: an egress broker injects them
on the way out, and the sandbox holds only a placeholder, so there are 0 real
keys inside the sandbox.

Network access is set when the sandbox is created, from the user's terminal.
When the work needs to reach an API, ask the user to create the sandbox with an
allow-list, for example:

```
kosa8 sandbox create sealed --egress-allow api.anthropic.com
```

Everything not on the list is refused. Allow only the hosts the task needs.

## Checkpoint before risky steps, rewind when something breaks

A checkpoint saves the whole running machine, memory and processes, in about
0.7 seconds. A rewind puts it back, still running, in about a second.

- Before any step you could not undo, call `sandbox_checkpoint` with a `label`
  saying why (for example "before dropping the users table").
- If the step goes wrong, call `sandbox_rewind`. It defaults to the most recent
  checkpoint; pass `to` with an id or label for an earlier one.
  `sandbox_checkpoint_list` shows what is available.
- Rewinding is almost always cheaper than repairing the damage. Checkpoints
  taken after the one you return to are discarded, and the current state is
  destroyed, so checkpoint first if you might want it back.

## Fork to try variants

- `sandbox_fork` copies a running sandbox into N live copies (`count`), each
  resuming from the parent's exact state with processes intact. Use it to try
  approaches in parallel from one starting point.
- `sandbox_explore` does best-of-N in one call: give it at least two shell
  commands in `variants`; it forks once per command, runs them concurrently,
  reports which succeeded with timings and output, and keeps the fastest
  successful branch (unless `keep_all` is set). Do the shared setup in the
  parent first, so every branch starts from it.

## Approvals

Risky actions can wait for one or two named human approvers (the two-person
rule). If a tool call is refused because the org policy requires an approval,
the refusal says how a human gets it. Tell the user what you were trying to do
and why, and wait. A person reviews held calls at their own terminal
(`kosa8 mcp asks`, then `kosa8 mcp approve ID` or `kosa8 mcp refuse ID`), or
signs a request with `kosa8 approve sign`. Once approved, repeat the same call,
passing the approval id in `approval` when the refusal asked for one.

Never try to approve your own call, edit kosa8 policy, or switch to running the
action on the host to avoid an approval.

## Audit

Every action is written to a hash-chained, tamper-evident audit log, marked
human or agent. The user can check it with `kosa8 audit verify`.

## Cleaning up

When the work is done, report what you ran and where. Use `sandbox_stop` to
keep a sandbox's disk for later, or `sandbox_delete` to remove it, but ask
before deleting a sandbox you did not create in this session.
