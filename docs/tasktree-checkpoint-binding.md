# Tasktree Checkpoint Binding

status: Runtime Binding
version: 2.0
normative: No
depends on: checkpoint-profile, tasktree
implementation: tasktree

---

This document maps [checkpoint-profile](../docs/checkpoint-profile.md)
actions to concrete tasktree CLI invocations.

If you use a different runtime implementation, replace this binding
with the equivalent commands for your tool. The profile remains valid.

---

## Pre-action

```
tasktree list
tasktree show <relevant strand>
```

---

## Pre-state-closing

### Acknowledge `[open]` markers

```
tasktree show <relevant strand>
```

Read the log. If the last entry or any recent entry is `[open]`,
append a decision before proceeding.

### Run orthogonal verification

Verification commands are task-specific and independent of the fix logic.
Examples: `find`, `git grep`, `git diff`.

### Append verification result

```
tasktree append "<content>" <strand_id>
```

### Cross-system scope

```
tasktree list
tasktree show <relevant strand>
git status
```

Compare: strand says done, git doesn't reflect → conflict →
append conflict description.

---

## `[done]` Template

```
[done] verified: <verification command + output>
       no open: confirmed | skip: <reason>
       result appended: <strand_id or description>
```

Append using:

```
tasktree append "[done] verified: find . -type d -name removed
       no open: confirmed
       result appended: 0000019dce0e" <strand_id>
```

---

## CLI Outlook (not P0 — wait for real friction)

```
$ tasktree checkpoint <strand_id>

strand state:
  entries: 34
  last: [open] "waiting for real task"
  open markers: 1 unresolved

git state:
  modified: docs/checkpoint-profile.md
  untracked: none

verification template:
  [done] verified: <command + output>
         no open: <confirmed or skip-reason>
         result appended: <strand_id>
```

Read-only diagnostics. No judgment. No auto-append.
Same as `git status` — shows state, does not act.
