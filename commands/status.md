---
description: Check that kosa8 is installed and its daemon is running, and list the sandboxes
---

Call the kosa8 `kosa8_status` tool. If the daemon is not running or kosa8 is not
installed, say so plainly and point the user to https://kosa8.com/download/ (or
`kosa8 daemon start` if it is installed but stopped). Do not try to install or
start anything yourself.

If it is running, call `sandbox_list` and show a short table: name, state, and
the parent it was forked from, if any.
