---
description: Rewind a kosa8 sandbox to an earlier checkpoint, still running
argument-hint: <sandbox> [checkpoint id or label]
---

Arguments: $ARGUMENTS

The first word is the sandbox; the rest, if any, is the checkpoint id or label.

1. Call `sandbox_checkpoint_list` for that sandbox and show the checkpoints.
2. Tell the user what a rewind does: the sandbox goes back to that checkpoint,
   the current state is destroyed, and checkpoints taken after it are
   discarded. Offer to take a checkpoint of the current state first.
3. Only after the user confirms, call `sandbox_rewind` (pass `to` if a
   checkpoint was named; otherwise it uses the most recent one) and report the
   result.
