<!-- strand-checkpoint-profile:start -->

## Runtime Governance Checkpoint

This project follows [checkpoint-profile](../docs/checkpoint-profile.md).
Tasktree binding: [tasktree-checkpoint-binding](../docs/tasktree-checkpoint-binding.md).

Before destructive action:
1. Run `tasktree list`
2. Run `tasktree show <relevant-strand>`
3. If blocking `[open]` exists, append a decision before proceeding.

Before `[done]`:
1. Acknowledge blocking `[open]`
2. Run one orthogonal verification command
3. Append verification result
4. Only then append `[done]`

`[done]` format:

```
[done] verified: <verification command + output>
       no open: confirmed | skip: <reason>
       result appended: <strand_id or description>
```

<!-- strand-checkpoint-profile:end -->
