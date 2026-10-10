# Design: what the summary is for

This is the design record behind the change: **a compaction summary stops trying to be a smaller copy of the conversation and becomes a navigation layer plus a tension layer, with facts moved out to something that is not summarized and stays addressable.**

It was written before the code, against measurements taken on a live deployment, and it is kept here in the form it was argued in — including the parts that were overturned later.

## 1. Diagnosis: one call, two opposing forces

The instruction that produced summaries in this deployment carried, in the same call:

- a rule to preserve verbatim material — *"Preserve exact file paths, commands, error strings, identifiers, numeric values, function signatures, and syntax fragments"*; and
- rules that pull the other way — write terse bullets rather than prose, compress pasted logs to a one-line gist, and, when folding an existing checkpoint, *merge it and drop what is stale*.

Verbatim retention and prose compression are **opposing forces**. Asking one summarization call to do both does not produce a compromise: the model resolves toward compression, which is what the surrounding rules ask for. The failure is not model quality; it is **task mismatch** — a prose summarizer cannot be made to do "do not summarize this part", and the instruction that tries is a hope, not a mechanism.

The measurement that settled it was deterministic and cheap: an instrument that compares a summary against the exact text it shadowed and reports three columns — **file paths, genuine error codes, and identifiers present in the original but absent from the summary**. On one real session: **360 local file paths and 14 genuine error codes** were present in the original and gone from the summary. Before filtering (env-var names and external URLs are noise, not failures) the raw numbers were 390 paths and 79 error codes; after filtering, 360 and 14 were still genuinely dropped.

That instrument is what turns "the summaries feel thin" into a number, and it is also what defines the boundary the design uses: it names exactly the class of content that a summary must not be asked to carry.

## 2. Split by what survives summarization, not by topic

The original structure was organized by *topic* — sections for files, errors, context, pending work. Topic-organized sections put "must survive verbatim" and "survives paraphrase fine" material in the same buckets, so one pass of summarization takes the facts with it.

The split the design uses instead:

- **What survives summarization** — judgments, reasons, signals, tension — goes in the summary. It is abstract; summarizing it a hundred times still yields the same sentence.
- **What does not survive** — paths, error codes, commands, numeric values, quotations — goes somewhere that is not summarized, and the summary carries a pointer to it. It is concrete; one pass of prose turns it into something else.

## 3. Three rules

1. **The summary carries what cannot be damaged by summarizing.**
2. **Notes carry what is damaged by summarizing.**
3. **The most valuable thing in a summary is tension, not information.** Closed work can be reconstructed from its results; unclosed work — pending items, blockers, open questions, commitments — is gone the moment it is dropped.

A counter-intuitive consequence, argued in the design and worth stating plainly: **"what was done" should not occupy the summary.** It is banked — it exists in code, files and commits — and a summary that spends its budget restating it is spending it on the reconstructable. What deserves the space is what is still *hanging*: the next step, the commitment, the thing that was tried and rejected.

A second line from the same argument, kept because it is the whole idea in six words: **"remembering that there is a pitfall is a good state."** A pitfall is a *signal*, not content. You rarely need to remember what it looked like; you need to remember "there is a pitfall here, do not walk straight into it". The signal stays resident; the scene is read on demand.

## 4. The structure: navigation layer + tension layer

| # | Section | Carries | Origin |
|---|---|---|---|
| 1 | `Goal and Intent` | what the user wants (the goal anchor) | from the original request section, minus the verbatim requirement |
| 2 | `Current State` | where we are, plus technical context (frameworks, patterns, conventions) | current work + technical concepts |
| 3 | `Next Step` | the single next action | kept as-is |
| 4 | `Decisions and Rationale` | decisions with the reason at the time, compressed to one line | split out of the catch-all context section |
| 5 | `Pitfalls` | the *signal* of a pitfall — "X leads into Y", "careful here" | new; the error codes move out to notes |
| 6 | `Pending and Open` | everything unclosed: to-dos, blockers, open questions, commitments | the original pending section, widened |
| 7 | `Boundaries` | constraints and what is explicitly out of scope | split out of the catch-all context section |
| 8 | `User State` | user state and preferences (the relationship layer) | split out of the catch-all context section |

The deployed section list keeps this split and is a superset of the table; the point is not the count, it is that **every section now has a written answer to two questions**: what goes in, and what deliberately stays out.

## 5. Per-section: in / out

The hints for every section were rewritten to state both halves, because a section that only says what to include will happily accept facts that belong in notes.

| Section | In (resident, survives summarizing) | Out (not summarized, addressable) |
|---|---|---|
| `Goal and Intent` | goal, intent, direction | the verbatim wording of the request |
| `Current State` | progress, what is in hand, the stack | exact file paths, code fragments |
| `Next Step` | one next action | the full step list (that is a plan) |
| `Decisions and Rationale` | decision + reason, **one line**: "chose A because B has a pitfall" | the twenty lines of argument behind it |
| `Pitfalls` | the signal: "X will hit Y", "careful" | the scene: error codes, stack traces, reproduction steps |
| `Pending and Open` | the unclosed list, each item one line | the full context of each item (that is its own document) |
| `Boundaries` | constraints, explicitly-out-of-scope | — |
| `User State` | state and preferences | — |

**The granularity is deliberate.** A reason is compressed to the point where one sentence carries it: the summary produces *the clue*, the notes hold *the answer*, and you fetch the answer only when you actually walk that way.

## 6. Where the facts go: pointers to a lossless original

In an event-sourced harness the original text was never lost — the log keeps it. What compaction removes is the model's *visibility* of it: the shadowed span is replaced by a summary, and everything the summary does not restate is, from the model's side, gone.

So the second half of the design is not a new store. It is **a readable route back**:

- the verbatim material that must not be paraphrased is written to a file by a deterministic extractor (no model involved, so no loss), and
- the summary carries a pointer line naming that file, so a later turn can read it when it needs exact values.

This is the same shape as the file-reference approach used by other harnesses, with one difference that matters: because the log is append-only and the original event is still there, the pointer can address **the original itself**, not a copy. The pruning marker in the same system was later given the same treatment: an elided middle is described as *deferred* — "full copy retained as session event #N" — rather than as destroyed.

## 7. What it touched, and the pass criteria

- the section table (titles and hints) — the summarizer instruction is rendered from it, so the change propagates by construction;
- the summarizer's rules, which gained an explicit **tension-first** rule: keep unresolved tension verbatim, keep completed actions out of the way;
- the retention instrument's reading, which had to be re-aligned with the new boundary (it still reports paths, error codes and identifiers — the class the summary no longer claims to carry — so the number stays honest and keeps working as a regression check).

Pass criteria used at the time: the structural check still passes section by section on the new list (the sections still exist, at 100%), the extractor criteria agree with the new boundary, and the compaction package's full test suite is green.

## 8. What this design does not do

- It does not rebuild compaction: thresholds, retention budget and selection are untouched.
- It does not add retrieval-by-similarity; the route back is explicit and addressable, not guessed.
- It does not assume a model will recover the facts on its own. The pointer is a route, not a promise: whether a later turn walks it is a separate question, and it is measured separately.
- The later layering pass — moving slow-changing material (aiming, boundaries, user state) out of the summary entirely and leaving a declaration of what must stay verbatim — is **design, not shipped**; see `COMPONENTS.md`.
