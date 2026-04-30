# Runtime Governance Checkpoint

status: Operating Profile
version: 2.0
normative: No
depends on: strand-protocol SPEC

---

## Definition

A checkpoint is a mandatory re-read boundary before an agent performs an
irreversible or state-closing action.

A checkpoint is not a lock. It does not block concurrent agents. It gives
a stale cognitive snapshot the opportunity to be corrected before an action
that cannot be undone.

Checkpoint records the agent's observed context before action. It does not
decide whether the action is correct.

---

## v1.0 → v2.0

v1.0 solved "forgot to look."
v2.0 solves "looked but ignored."

Four patches, tested against real agent failure (arch35 deletion →
premature closure):

1. **Post-action verification gate.** Before state-closing, run a
   verification command with no causal link to the fix.
2. **`[open]` processing gate.** Unresolved `[open]` in current strand
   blocks state convergence.
3. **Cross-system scope.** File/strand conflict triggers git/filesystem
   check.
4. **`[done]` preconditions.** Minimum gate: no open `[open]`,
   verification done, results appended.

---

## Trigger Conditions

**Pre-action** (retained from v1.0):
- Destructive operations (delete, move, bulk rename, overwrite)
- State-closing operations (marking resolved, verified, done)
- File/strand conflict detected
- Execution context may have aged past new constraints

**Post-action** (new in v2.0):
- Before any state-closing action, run at least one orthogonal
  verification command

---

## Checkpoint Actions

### Pre-action

```
tasktree list
tasktree show <relevant strand>
```

### Pre-state-closing

1. **Acknowledge `[open]` markers.**

   If `tasktree show` reveals an `[open]`, it MUST be acknowledged before
   proceeding. Acknowledged means: append what you decided to do with it.

   Continue, defer, or hand off — all are valid. Ignoring is not.

   Blocking `[open]`: in current strand, directly related, agent has
   authority. Independent cognitive lines (different topic, layer, or
   agent) do not block. If intentionally not resolved, append explicit
   skip declaration with reason.

2. **Run orthogonal verification.**

   At least one command independent of fix logic.

3. **Append verification result before `[done]`.**

4. **Expand scope on conflict.** Check in order:
   - `tasktree list` / `show` — strand state
   - `git status` — file state
   - Compare: strand says done, git doesn't reflect → conflict →
     must not proceed → append conflict description

---

## Order Constraint

```
execute operation
  → run orthogonal verification
    → append checkpoint declaration (with verification result)
      → continue
      or → append [friction] if verification fails
```

If verification fails, append `[friction]`, not "checkpoint ok."
A failed verification is data, not a veto — but the record must
reflect the failure.

---

## `[done]` Preconditions

```
[done] verified: <verification command + output>
       no open: confirmed | skip: <reason>
       result appended: <strand_id or description>
```

All three lines required. Missing any line = incomplete closure.

---

## Summary

**Before destructive action:**
```
tasktree list
tasktree show <relevant strand>
```

**Before `[done]`:**
1. Acknowledge all blocking `[open]` in current strand
2. Run at least one orthogonal verification command
3. If verification fails → append `[friction]`, never "checkpoint ok"
4. If verification passes → append checkpoint declaration with result
5. Write `[done]` using the three-line template

---

## Boundary

Checkpoint records the agent's observed context before action. It does
not decide whether the action is correct. Verification commands that
pass are evidence. Those that fail are also evidence. Both are appended.
Neither is a veto.

---

## CLI Outlook (not P0 — wait for real friction)

```
$ tasktree checkpoint <strand_id>

strand state:
  entries: 34
  last: [open] "waiting for real task"
  open markers: 1 unresolved

git state:
  modified: docs/runtime-governance-checkpoint.md
  untracked: none

verification template:
  [done] verified: <command + output>
         no open: <confirmed or skip-reason>
         result appended: <strand_id>
```

Read-only diagnostics. No judgment. No auto-append.
Same as `git status` — shows state, does not act.

---

## Protocol Scope

This is a runtime governance protocol, not a data protocol.
It operates at the tasktree usage layer, not the SPEC layer.
It does not modify journal format, strand primitives, or `[open]`
semantics.

---

## Origin

arch35 deletion incident. Three checkpoint attempts, each did a re-read,
each saw `[open]` — and proceeded anyway. v1.0 solved "forgot to look."
v2.0 solved "looked but ignored."
