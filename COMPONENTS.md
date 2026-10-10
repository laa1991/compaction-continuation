# Components: what the design produced

Each row is a piece that exists in the deployment this design ran on, followed by the half of the design it serves. **[shipped]** means it runs in live sessions; **[design]** means it is argued and measured but not implemented.

Scope note: this document is about **the design**, not about attribution. Which of these pieces are deployment-local and which travelled upstream is deliberately out of scope here.

## Fact half — get the unsummarizable out of the summary

**1. Verbatim spill with a pointer — [shipped]**
A deterministic extractor (no model in the loop, so no loss by construction) pulls the exact material a summary must not paraphrase — file paths, genuine error codes, identifiers — writes it to one file per fold, and the summary ends with a single pointer line naming that file. Before this, the same class of content was asked for *inside* the summary prose, with an instruction; the measurement in `EVIDENCE.md` is what showed that does not hold.

**2. Self-addressing pruning marker — [shipped]**
Over-budget tool results are rewritten to a bounded head, a marker, and a bounded tail, while the original event stays in the append-only log. The marker carries the address: *"N chars omitted; full copy retained as session event #S"*. An elided middle is therefore **deferred, not destroyed** — which is what makes it safe to set the pruning threshold to what the budget actually earns instead of what survival allows.

## Tension half — make the summary carry what is unsettled

**3. Navigation layer + tension layer — [shipped]**
The section list was reorganized and every section's hint rewritten to state both what it carries and what deliberately stays out (`DESIGN.md` §4–§5). The summarizer rules gained an explicit tension-first clause: keep unresolved tension verbatim, do not let completed actions crowd it out.

**4. Session-compaction entry points — [shipped]**
A human `/compact` command and a model-side compact tool, with per-call options (a focus string; keeping user messages verbatim). The model-side entry point is what makes compaction an *autonomous* act — triggered when a topic closes rather than only when the window overflows — which matters because the moment of compaction is the only moment when the previous and the current text are in one place.

**5. Folding that fits — [shipped]**
A cap on the region a single fold may select, with staged folding of the oldest span when the summary request itself would not fit. This removes a failure mode where the fold fails or truncates precisely on the longest sessions.

## Measurement half — keep the boundary honest

**6. Retention instrument — [shipped]**
Three deterministic columns — dropped file paths, dropped genuine error codes, dropped identifiers — measured by comparing a summary against the exact text it shadowed. It is the criterion behind the split, and it stays in place as a regression check: the summary no longer claims to carry those three classes as prose, so the instrument now measures the boundary (does the pointer path actually cover what left the summary) rather than a hope.

**7. Pricing the shadowed span with the meter that measured it — [shipped]**
A unit-mismatch fix: the shadowed span's token count is computed with the policy that owns that measurement, so cost readings and selection decisions stop disagreeing.

## Still design

**8. Layering: move slow-changing material out, declare what must stay — [design]**
The next step this design pointed at: sections that are stable across generations (aiming, boundaries, user state) do not need to be re-written by every fold; they want a home that compaction cannot touch, with the summary holding only what changes.

The measured part of this is worth recording because it removed the obvious implementation: **"slow-changing" is not mechanically detectable.** Over long-range sessions the longest run of consecutive generations with an unchanged section was 0–2, and line-level verbatim inheritance was 4.5%. A heuristic that looks for stability finds nothing to move. What replaced it is a *declaration*: the material that must remain verbatim is named explicitly rather than inferred.

**9. Configuration surface — [shipped, unexercised]**
A default focus for compaction and a per-call switch to keep every user message verbatim. Both are wired and unit-tested; neither is exercised by default. Listed here because an unexercised switch is a claim about intent, not a measurement.
