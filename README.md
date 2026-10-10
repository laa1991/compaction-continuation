# Compaction as Continuation, Not Restoration

> What we changed in a real, continuously used agent deployment after measuring what compaction actually drops — and the design that followed it: **facts leave the summary, the original stays addressable, and unresolved tension is what the summary is for.**

Long-horizon agents survive their context window by *compacting*: a model summarizes the older part of a conversation, the summary replaces what it shadowed, and work continues. Almost all discussion of this step is about **fidelity** — did the summary keep enough? In our measurement work we found the more useful question is **continuity**: after the fold, can the next turn still tell what is unsettled, what was tried and rejected, and where the exact values live?

This repository is the design side of that answer. It is not a plugin and not a patch: it is the design record of replacing *summary as lossy copy* with *summary as navigation + tension, plus pointers to a lossless original*.

## What the design says, in four lines

1. **A summary cannot be both a compressor and a verbatim store.** Our own instruction asked it to preserve exact paths, commands, error strings and values — while the same call also demanded terse bullets and dropping stale material. Those are opposing forces in one LLM call, and the model resolves them the same way every time. Measured on one real session: **360 local file paths and 14 genuine error codes** were present in the shadowed original and absent from the summary prose.
2. **Split by what survives summarization, not by topic.** What survives — judgments, reasons, signals, tension — belongs in the summary. What does not — paths, error codes, commands, values, quotations — belongs in a place that is not summarized, with the summary carrying a pointer to it.
3. **The most valuable thing in a summary is tension, not information.** Closed work can be reconstructed from its results; what is unclosed — pending items, blockers, open questions, commitments — is gone the moment it is dropped.
4. **In an event-sourced harness the original is already on disk.** The problem is not that verbatim was lost; it is that verbatim is no longer *visible to the model*. The fix is a readable route back to it, not a new store.

## What is in here

| File | What it is |
|---|---|
| [`DESIGN.md`](DESIGN.md) | The design: diagnosis, the split, the section structure (navigation layer + tension layer), the boundary with the original-text pointer, and the pass criteria |
| [`COMPONENTS.md`](COMPONENTS.md) | The components that came out of it — what each one does and which half of the design it serves |
| [`EVIDENCE.md`](EVIDENCE.md) | The measurements this rests on, with the instruments named, and the report where the long-form numbers live |
| [`DATA-POLICY.md`](DATA-POLICY.md) | What is publishable and what is not, from the same deployment |

## What we do not claim

- **One deployment.** Every rate here is a rate about one continuously used single-machine agent deployment. What we argue transfers is the **design and the instruments**, not the rates.
- **Not all of it is shipped.** The section split and the reference pointer are implemented and live; the later layering pass (moving slow-changing material out of the summary altogether) is a design with measurements behind it, still open. `COMPONENTS.md` marks which is which.
- **The adjacent measurements are separate.** What compaction drops quantitatively — survival curves, two rulers, failure modes — lives in the report linked from `EVIDENCE.md`, which also states its own limitations.

## License

Design and text (the `*.md` files here): CC BY 4.0. The red-line scanner in `tools/`: Apache-2.0 — see `LICENSE`.
