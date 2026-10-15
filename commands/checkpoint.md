---
description: Save a restore point of a running kosa8 sandbox (memory, processes and files)
argument-hint: <sandbox> [label]
---

Arguments: $ARGUMENTS

The first word is the sandbox name; anything after it is the label. If no
sandbox is given, call `sandbox_list` and ask which one. If no label is given,
write a short one saying why (for example "before migration").

Call `sandbox_checkpoint` with that sandbox and label, then show the checkpoint
id and label so the user can rewind to it later with /kosa8:rewind.
