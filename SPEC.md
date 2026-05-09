# strand Protocol Specification v1.0

## Bootstrap Declaration

This protocol was not designed in advance. It emerged from friction
encountered during the development of tasktree itself — an AI-native
tool for cognitive external memory. Every primitive defined here has a
traceable origin in a specific event that blocked real work.

The protocol exists because the primitives proved necessary. It is
published because the primitives proved portable.

勿谓言之不预也。

The protocol exposes the record as written. There is no summary layer,
no automatic triage, and no hidden priority queue. Every strand and
every log entry — including recovery markers such as [open] — remains
visible as part of the record. Visibility is not prioritization. The
agent and the human operator must decide what matters.

This places a demand on model quality. Use the strongest reasoning
model available for sessions that involve irreversible actions,
state-closing decisions, or evidence used later. The protocol has been
empirically validated with:

  Claude Code with DeepSeek V4 Pro / Flash, 1M context window
  Pi (no plugins) with DeepSeek V4 Pro / Flash, 1M context window
  Codex with GPT-5.5, medium reasoning or above

The requirement is not a specific vendor or model family. It is the
ability to maintain layer awareness across a long, unfiltered record
and independently distinguish signal from noise.

If you find yourself wanting the protocol to hide things or tell you
what matters — you are looking for a workflow manager, not a memory
protocol. This protocol does not fit that role. It does not filter. It
does not prioritize. It records.

The judgment is yours.

The protocol exposes complexity honestly. It does not hide, summarize,
or triage. Models and operators without the judgment to navigate raw
records will get lost in the details.

That is not a bug.

## Agent Quick Start (Non-Normative)

This section guides agents using the bundled reference implementation.
It is not a protocol requirement.

If you are an agent receiving only this repository:

1. read the protocol;
2. identify your boundary using the Boundary Rule below;
3. on Windows, unpack the bundled tasktree release and locate the
   `tasktree.exe` binary;
4. run `tasktree.exe list`;
5. run `tasktree.exe show <latest-open-strand>`;
6. follow the latest `[open]`;
7. if you get stuck, append what happened before trying a workaround.

Current release boundary: the bundled reference binary is Windows-only.
Other platform support is pending community contributions or future
releases. If you are not on Windows and no compatible binary is present,
do not invent a platform path; record the platform limitation as the
blocking point.

## Plain-Text Subject Entry (Non-Normative)

This section guides agents that can only read this specification as
plain text. It is not a journal format requirement.

If you can only read this specification as plain text and cannot access
a repository, run `tasktree.exe`, or write `.tasktree/journal.jsonl`, do
not fabricate journal events, strand ids, or timestamps.

Your compliant action is to state the boundary and, if useful, propose
an append in plain text. A proposed append is not part of the journal
until a subject with write access appends it.

Do not treat the chat transcript as a substitute journal. Plain-text
simulation may help explain the protocol, but it does not create durable
strand state. The next recoverable action should point to a subject that
can access the repository and append to the journal.

## Design Intent

This protocol is not designed to prevent agents from making mistakes.
It is designed to make mistakes recoverable and locatable.

A successful implementation should make it possible to answer:

- What was the agent trying to do?
- What did it know at that point?
- Where did it get stuck?
- Which step or boundary produced the failure?

## Boundary Rule

Before acting, identify the effective boundary:

1. Which repository am I in?
2. Which subject am I acting as?
3. What files and tools does this subject actually have?
4. What information must not be assumed across this boundary?

Do not assume tools, history, or governance rules are available unless
the current subject can directly invoke them.

## Success Criterion

The protocol succeeds when an agent with only this repository and the
tasktree release can recover working state from files alone.

In the simplest useful case: five lines, one strand, full session
record — problem, decision, progress, recovery point.

The standard is not perfect agent behavior. The standard is whether a
later agent can recover the working state from files alone and locate
where a failure occurred.

## Failure Is Evidence

A failed command, wrong assumption, confused handoff, or crossed
information boundary is not outside the protocol. It is one of the main
inputs.

When an agent misreads a task or resumes the wrong strand, the protocol
should preserve enough context for a later agent to identify the failure
point in the strand history.

## 1. Overview

Strand is a protocol for **cognitive external memory** — persisting
and recovering reasoning context across AI sessions, tools, and models.

It defines three primitives and one storage format. That is all.

A strand is a view projected from append-only journal events, not a
separate stored object. See §6 for the projection rules.

### 1.1 Three Primitives

| Primitive | Verb | Definition |
|-----------|------|-----------|
| **strand** | contains | A named causal line — groups reasoning about one topic into a traversable sequence |
| **append** | records | An atomic reasoning increment — one thought, one record |
| **[open]** | hands off | A recovery marker — where the next session should resume |

### 1.2 What This Protocol Is NOT

- **Not a task manager.** No status fields, no assignment, no due dates.
- **Not a state machine.** Strands have no enforced lifecycle.
- **Not a workflow engine.** No automation, no triggers, no pipelines.
- **Not a database.** No query language, no indexing, no transactions.
- **Not a compliance checklist.** The protocol does not replace judgment
  with fields to fill out.
- **Not a governance stack.** It does not require external governance
  documents, project histories, or central coordination.

These are not omissions. Judgment belongs to the agent.
The protocol records the trace of judgment; it does not define the judgment itself.

It is a scratchpad. It persists. That is all.

### 1.3 Reference Implementation

tasktree is the Windows reference implementation of this protocol.
Current version: v0.1.4.

[GitHub Releases](https://github.com/GodOnlyKn0w/strand-protocol/releases)

Other platform support is pending community contributions or future releases.

---

## 2. Strand

### 2.1 Definition

A strand is a **topic-bound causal line**. All appends within a strand
share a common subject. Strands are causally independent of each other.

A strand is projected from journal events at read time. It is not stored
as a separate file or database row. See §6.

### 2.2 Identity

Each strand has a unique, immutable identifier. The identifier:

- MUST be stable across sessions
- MUST NOT collide with other strand identifiers in the same journal
- SHOULD be time-sortable (for chronological listing)

The reference implementation uses a timestamp-prefixed hex string.

### 2.3 Lifecycle

```
create → append → append → ... → [done]
                                  or
                                  archive ([open] → new strand)
```

A strand is created with its first append (which serves as the title).
It accumulates appends until the topic is complete or the strand
exceeds its cognitive unit.

### 2.4 Cognitive Unit Boundary

A strand MUST NOT mix different granularities of cognitive objects.
Processes, states, and decisions about different topics belong in
different strands.

When a strand exceeds this boundary, the final append is `[open]`
pointing to a new strand. The new strand begins with an append
referencing the old strand.

Example: "testing a repo entry point" and "reflecting on this agent's
mistake while testing it" are related, but they are different cognitive
objects. They belong in different strands, with an append linking the
relationship when needed.

### 2.5 Strand Independence

Strands are causally independent. Strand A does not depend on strand B.
Strand A does not need to know strand B exists. There is no inter-strand
ordering guarantee.

---

## 3. Append

### 3.1 Definition

An append is an **atomic reasoning increment**. One thought, one decision,
one observation, one correction — one record.

### 3.2 Properties

| Property | Guarantee |
|----------|-----------|
| **Atomic** | An append is indivisible. Partial appends are never visible. |
| **Immutable** | Once written, never modified or deleted. Corrections are new appends. |
| **Ordered** | Within a strand, appends are ordered by creation time. |
| **Free-form** | Content has no enforced schema. Any structure is convention, not contract. |

### 3.3 Timestamp

Each append carries a timestamp set by the writer at append time. The
timestamp MUST be in a sortable format (ISO 8601 recommended).

Implementations MUST NOT assume strict monotonicity across writers.
Concurrent writers may have unsynchronized clocks.

---

## 4. [open]

### 4.1 Definition

`[open]` is a **recovery point marker** — the last append of a session
that tells the next agent where to resume.

### 4.2 Required Content

An `[open]` append MUST contain enough context for the next agent to
continue without reading prior history:

- Where we are (current state)
- What we verified (completed work)
- What is blocked (if any)
- What to do next (recovery action)

### 4.3 Properties

| Property | Guarantee |
|----------|-----------|
| **Explicit handoff** | Contains the minimum context for resumption |
| **Last-position visibility** | Always the final append of an active strand |
| **Cross-model** | Plain text, readable by any model at any capability level |

### 4.4 Concurrency and Asynchrony

`[open]` has two roles:

- **Attention concurrency:** multiple strands may be open at the same
  time. Listing them shows the agent which cognitive lines can be
  resumed.
- **Recoverable asynchrony:** a strand can be resumed later by another
  session, model, or agent because the handoff is stored in plain text.

This protocol does not schedule work or run agents in parallel. It only
makes multiple unfinished cognitive lines visible and resumable.

### 4.5 Non-Guarantees

- `[open]` does not trigger automatic action. It is a signpost.
- `[open]` does not guarantee freshness. The agent must validate.
- `[open]` does not route to other strands. Discover them via listing.

---

## 5. Storage Format

### 5.1 Journal File

The canonical storage is a file named `journal.jsonl` in a `.tasktree/`
directory. (The directory is named after the reference implementation.)

```
.tasktree/
└── journal.jsonl
```

### 5.2 JSONL Format

Each line is a valid JSON object. No cross-line dependencies. No
trailing commas. No multi-line records.

### 5.2b Causal Order

Journal line position is the canonical causal order. The `ts` field is
observational metadata recorded at write time. It is used for display
and diagnostics only.

```
causal coordinate = journal line number (0-indexed)
                     ↑
                     append-order, not wall-clock
```

When two events' timestamps contradict their line positions (a later
line has an earlier `ts`), the line order wins. Timestamps are not
ordering authority.

Offset references (`ack_offset`, `state_offset`, `dispatch_offset`)
are valid only within one uninterrupted journal lineage. Truncation
or rebuild invalidates all offset references.

### 5.3 Required Events

Every journal MUST support these event types:

#### strand_created

Creates a new strand.

```json
{"type":"strand_created","id":"<strand_id>","ts":"<iso8601>"}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | yes | `"strand_created"` |
| `id` | string | yes | Unique strand identifier |
| `ts` | string | yes | ISO 8601 timestamp |

#### log_appended

Appends a reasoning increment to a strand.

```json
{"type":"log_appended","id":"<strand_id>","ts":"<iso8601>","content":"<text>"}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | yes | `"log_appended"` |
| `id` | string | yes | Target strand identifier |
| `ts` | string | yes | ISO 8601 timestamp |
| `content` | string | yes | Free-form text content |

### 5.4 Optional Events

Implementations MAY support these events:

| Event | Purpose |
|-------|---------|
| `strand_hidden` | Exclude strand from default listing |
| `strand_unhidden` | Restore strand to default listing |
| `edge_linked` | Create directional reference between strands |
| `edge_unlinked` | Remove directional reference |

### 5.5 Backward Compatibility

Implementations reading a journal:

- MUST ignore unknown event types
- MUST NOT fail on unknown fields within known event types
- MUST preserve all events when rewriting (append-only)

### 5.6 Append-Only Invariant

The journal is append-only. Implementations:

- MUST NOT modify existing lines
- MUST NOT delete existing lines
- MUST NOT reorder existing lines
- MUST append new events at the end

Violating this invariant breaks the cognitive timeline for all readers.

### 5.7 Example

```json
{"type":"strand_created","id":"0000019dcd67","ts":"2026-04-28T03:00:00Z"}
{"type":"log_appended","id":"0000019dcd67","ts":"2026-04-28T03:00:00Z","content":"Pi startup blocked — OpenRouter 402 credits"}
{"type":"log_appended","id":"0000019dcd67","ts":"2026-04-28T03:15:00Z","content":"[decision] switch to DeepSeek V4 Pro, 1M context"}
{"type":"log_appended","id":"0000019dcd67","ts":"2026-04-28T03:45:00Z","content":"[progress] model switch complete, session running"}
{"type":"log_appended","id":"0000019dcd67","ts":"2026-04-28T04:00:00Z","content":"[open] next: verify toolchain integration"}
```

Five lines. One strand. Full session record: problem → decision → progress → recovery point.

---

## 6. Projection

A strand is not stored directly. It is projected from events at read time.

For each strand:

```
strand = {
    id:          from strand_created.id
    created_at:  from strand_created.ts
    updated_at:  from max(log_appended.ts)
    log:         [all log_appended for this id, ordered by ts]
}
```

The strand listing is derived by:

1. Group all events by `id`, filter to those with a `strand_created` event
2. For each strand, project as above; extract first and last `log_appended.content`
3. Sort by `updated_at` descending (most recently active first)

---

## 7. Conventions (Non-Normative)

The following conventions are observed in real use but are NOT part of
the protocol. Implementations MAY support them; agents MAY ignore them.

### 7.1 Content Prefixes

Agents often prefix appends with semantic markers:

| Prefix | Meaning |
|--------|---------|
| `[friction]` | A real workflow blockage |
| `[decision]` | A choice made and why |
| `[progress]` | A completed action or verified result |
| `[observation]` | A discovered pattern |
| `[correction]` | Fixing a previous misunderstanding |
| `[done]` | Explicit strand closure |

`[open]` is not a non-normative prefix. It is the normative recovery
primitive defined in §4. It appears inside append content, but its role
is stronger than the optional reading hints above.

### 7.2 First Append Convention

The first append of a strand is its title and positioning — not a log
entry. It answers "why does this strand exist?"

### 7.3 Session Closure

Every active session ends with an `[open]`, even if the open point says
"no immediate next action." A strand closed with `[done]` is complete
and has no recovery point.

---

## 8. Implementation Compliance

This section is for implementers of compatible tools. If you are an
agent using an existing tasktree release, §1-§7 are the operational
path; you do not need this section to start.

### 8.1 Minimum Compliance

A compliant implementation MUST:

1. Read and write `journal.jsonl` in the format defined in §5
2. Support `strand_created` and `log_appended` events
3. Project strands as defined in §6
4. Preserve the append-only invariant (§5.6)
5. Ignore unknown events and fields (§5.5)

### 8.2 Full Compliance

A fully compliant implementation SHOULD additionally:

1. Support all optional events (§5.4)
2. Display strands sorted by most recent activity (§6.2)
3. Support prefix matching for strand identifiers

### 8.3 Interoperability

Two implementations are interoperable if:

- Implementation A can read and project a journal written by
  implementation B without data loss
- Implementation B can read and project a journal written by
  implementation A without data loss
- Both produce equivalent strand listings from the same journal

---

## 9. Versioning

This is version 1.0 of the protocol. Future versions:

- MUST preserve backward compatibility with all prior journal formats
- MUST NOT remove required event types
- MAY add new event types (ignored by older readers)
- MAY add new optional fields (ignored by older readers)

---

## 10. License

MIT License

Copyright (c) 2026 杨镇泽

Permission is hereby granted, free of charge, to any person obtaining
a copy of this specification and associated documentation files, to deal
in the Specification without restriction, including without limitation
the rights to use, copy, modify, merge, publish, distribute, sublicense,
and/or sell copies of the Specification, and to permit persons to whom
the Specification is furnished to do so, subject to the following
conditions:

The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Specification.

THE SPECIFICATION IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED.

Implementations of this protocol may use any license.
