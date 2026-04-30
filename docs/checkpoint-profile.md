# Checkpoint Profile

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

Four patches, tested against real agent failure:

1. **Post-action verification gate.** Before state-closing, run a
   verification command with no causal link to the fix.
2. **`[open]` processing gate.** Unresolved `[open]` in current strand
   blocks state convergence.
3. **Cross-system scope.** File/strand conflict triggers filesystem
   check.
4. **`[done]` preconditions.** Minimum gate: no open `[open]`,
   verification done, results appended.

---

## Trigger Conditions

**Pre-action:**
- Destructive operations (delete, move, bulk rename, overwrite)
- State-closing operations (marking resolved, verified, done)
- File/strand conflict detected
- Execution context may have aged past new constraints

**Post-action:**
- Before any state-closing action, run at least one orthogonal
  verification command

---

## Actions

### Pre-action

Re-read the current strand head and list all active strands.
If new constraints are found, incorporate them before proceeding.

### Pre-state-closing

1. **Acknowledge `[open]` markers.**

   If the current strand contains an `[open]`, it MUST be acknowledged
   before proceeding. Acknowledged means: append what you decided to do
   with it.

   Continue, defer, or hand off — all are valid. Ignoring is not.

   Blocking `[open]`: in current strand, directly related, agent has
   authority. Independent cognitive lines (different topic, layer, or
   agent) do not block. If intentionally not resolved, append explicit
   skip declaration with reason.

2. **Run orthogonal verification.**

   At least one command independent of fix logic.

3. **Append verification result before `[done]`.**

4. **Expand scope on conflict.** Check in order:
   - Strand state
   - File state
   - Compare: strand says done, filesystem doesn't reflect → conflict →
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

## Boundary

Checkpoint records the agent's observed context before action. It does
not decide whether the action is correct. Verification commands that
pass are evidence. Those that fail are also evidence. Both are appended.
Neither is a veto.

---

## Dependency Direction

Checkpoint depends on strand-protocol SPEC. strand-protocol SPEC does
not depend on checkpoint.

Changes to strand-protocol SPEC may require changes to checkpoint.
Changes to checkpoint must never require changes to strand-protocol SPEC.

The dependency is one-way and non-normative.

---

## Protocol Scope

This is a runtime governance profile, not a data protocol.
It defines what to check and why, not how to invoke a specific tool.
It does not modify journal format, strand primitives, or `[open]`
semantics.

For tool-specific bindings (e.g. tasktree CLI commands), see the
examples/ directory. A runtime binding maps profile actions to
concrete invocations for a given implementation.

---

## Origin

Deletion incident. Three checkpoint attempts in succession — each
did a re-read, each saw `[open]` — and proceeded to close anyway.
v1.0 solved "forgot to look." v2.0 solved "looked but ignored."
