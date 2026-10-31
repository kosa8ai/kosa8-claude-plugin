# Installing the kosa8 MCP server (for AI agents)

These steps are for an agent (Cline, Claude Code, Cursor or similar) setting
up the kosa8 MCP server for a user. kosa8 runs every sandbox as a microVM on
the user's own machine, so the server needs the kosa8 runtime and its daemon
installed locally. It cannot run in a container or on a remote host.

## 1. Check the platform

kosa8 runs on:

- macOS on Apple silicon (M1 or later)
- x86-64 Linux with KVM (`/dev/kvm` must exist)
- Windows through WSL2 (preview)

On any other platform, stop and tell the user kosa8 is not supported there.

## 2. Install kosa8

Check whether it is already installed:

```sh
kosa8 version
```

If the command is not found, ask the user to install kosa8 from
https://kosa8.com/download/ (signed and notarised releases; Homebrew and Linux
packages are listed there). Do not download or run an installer without the
user's consent. kosa8 is commercial software with a free 14-day trial that
needs no account or card.

## 3. Start the daemon

```sh
kosa8 daemon start
kosa8 version
```

`kosa8 version` must print a line like `kosa8 daemon 0.5.0 (up 3s)`. If it
says `kosa8 daemon: not running`, run `kosa8 daemon start` again and check its output.

## 4. Add the MCP server

The server is a local stdio process. Add this to the client's MCP settings
(for Cline, `cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "kosa8": {
      "command": "kosa8",
      "args": ["mcp", "serve"]
    }
  }
}
```

If the client is started outside a terminal and cannot find `kosa8` on its
PATH, use the full path from `command -v kosa8` (commonly
`/opt/homebrew/bin/kosa8` or `/usr/local/bin/kosa8`).

To let sandboxes see a project folder, add `"--allow-mount", "/path/to/project"`
(read-only) or `"--allow-mount", "/path/to/project:rw"` to `args`. Without it,
no host folder can be shared.

## 5. Verify

Call the `kosa8_status` tool. It should report the daemon as running and its
version. Then, as a smoke test, call `sandbox_create`, run
`sandbox_run` with `alpine` and `echo ok`, and `sandbox_delete` it.

## Troubleshooting

- `cannot reach kosa8d`: the daemon is not running; see step 3.
- `kosa8 is not installed`: see step 2, then restart the client.
- Paid features refused after 14 days: the trial has ended; plans are at
  https://kosa8.com/pricing/.
