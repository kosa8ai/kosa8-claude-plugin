# Installing the kosa8 MCP server (for AI agents)

These steps are for an agent (Cline, Claude Code, Cursor or similar) setting
up the kosa8 MCP server for a user. There are two ways; ask the user which one
they want if it is not clear:

- **kosa8 Cloud (remote, nothing to install):** sandboxes run on kosa8's
  hosts. Needs only a kosa8 Cloud API key. Go to "Option A".
- **Local:** sandboxes run as microVMs on the user's own machine, through the
  kosa8 runtime and its daemon. Go to "Option B" (steps 1 to 5).

kosa8 is commercial software with no free tier: either way, the user needs a
kosa8 account on a plan (plans start at $29 a month, https://kosa8.com/pricing/).
Never create the account or enter card details for the user.

## Option A: kosa8 Cloud

1. Ask the user for a kosa8 Cloud API key (it starts with `k8c_live_`). If they
   have none, they make one at https://app.kosa8.com/app#keys after signing up
   at https://app.kosa8.com.
2. Add this to the client's MCP settings (for Cline, `cline_mcp_settings.json`),
   with their key in place of `k8c_live_...`:

```json
{
  "mcpServers": {
    "kosa8-cloud": {
      "type": "streamableHttp",
      "url": "https://api.kosa8.com/mcp",
      "headers": { "Authorization": "Bearer k8c_live_..." }
    }
  }
}
```

3. Verify: the server should list tools such as `sandbox_create`,
   `sandbox_exec` and `sandbox_delete`. As a smoke test, create a sandbox, run
   `echo ok` in it, and delete it.

A 401 means the key is missing or wrong; a 402 means the account has no plan
or hit a plan limit, and the error says what the user can do.

## Option B: local

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
https://kosa8.com/download/ (signed and notarised releases; Homebrew, winget,
Scoop and Linux packages are listed there). Do not download or run an
installer without the user's consent. Then the user signs in with
`kosa8 login` and picks a plan if they have none.

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
- Refused with a plan message: the account has no plan or hit a plan limit;
  plans are at https://kosa8.com/pricing/.
