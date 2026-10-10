# Evidence: the measurements this design rests on

Two kinds of evidence are kept separate here. The **first** was taken inside the deployment this design changed, to decide what to change. The **second** is the long-form public report, which measures what compaction drops and states its own limitations.

## 1. The criterion behind the split (deployment-internal)

An instrument compares a summary against the exact text it shadowed and reports three deterministic columns:

- file paths present in the original, absent from the summary;
- genuine error codes likewise (env-var names and external URLs are filtered out as noise, not failures);
- identifiers of the config/env kind.

On one real session, with the original instruction that asked the summarizer to preserve this material verbatim **while also** asking for terse prose and for stale material to be dropped:

| column | raw | after noise filtering | still genuinely dropped |
|---|---|---|---|
| local file paths | 390 | 360 | **360** |
| genuine error codes | 79 | 14 | **14** |

The reading is a reading about **one session on one deployment**; what it establishes is not a rate but the existence of the failure and its size when it happens. It is also what makes the boundary checkable afterwards: those three classes are exactly the ones the design stops asking the summary to carry, so the same instrument becomes the regression check on the pointer path.

## 2. Why continuity rather than fidelity — the sibling report

The long-form numbers live in a separate public report, **"Two Walls, Two Rulers: What Context Compaction Actually Drops in a Real Agent Deployment"** (independent report, September 2026; DOI [10.5281/zenodo.22931513](https://doi.org/10.5281/zenodo.22931513), code and aggregate data at [laa1991/two-walls-two-rulers](https://github.com/laa1991/two-walls-two-rulers)). Four of its findings are directly load-bearing for this design:

1. **Failures are three mechanisms, not one** — truncation, a fold that does not shrink, and a measurement mismatch — and the first two need opposite remedies. One needs an output-budget strategy; the other needs a criterion for *refusing* to fold that span.
2. **Summaries do not carry freshness — structurally.** The status section is written at generation time and rewritten only by the next fold; nothing in the summary format carries a timestamp or a validity condition, so an already-resolved state travels as if it were current. (This is why the deployment added an explicit re-arm after every fold: the card that carries live state is re-emitted after each compaction rather than trusted to survive it.)
3. **Two rulers, one phenomenon.** Constraint items survive **13% verbatim** after one compaction round and **0% by round four**, while surviving **91 / 77 / 53% semantically** at rounds 1 / 2 / 4. The same material, the same folds, two instruments: one says "everything is gone", the other says "most of it is still there, reworded". Any single number here is a statement about a ruler, not about the world.
4. **The miss replicates; the substitute does not.** Given material without a route back to the original, probes failed to retrieve the operative constraint; the constraint they substituted varied.

Frame 3 is the argument of this repository in one table: the thing that collapses across generations is **wording**, and the thing that most needs to survive is **what is still open**. A design that optimizes verbatim survival is optimizing the ruler that is already at zero by round four; a design that preserves tension and keeps the original addressable is optimizing the part that still carries.

A unit discipline came out of the same work and is worth carrying into any future table: **state the unit.** In our folds the top-two buckets (assistant text vs tool results) swap order depending on whether shares are counted in bytes or in occurrences, in roughly a third of comparable folds (38.7% of 463 folds in our measurement; a collaborating deployment reported 32.8% on theirs). A table that does not name its unit is not comparable with one that does.

## 3. How to reproduce the report's numbers

The report ships its instruments; they read ordinary session logs and need no third-party packages. The four most relevant: a census of folds (successful and failed), a survival curve for constraint items, a "did the fold actually shrink" check, and a redline scan over the report's own text. Each prints the criterion it applies, and two of them ship with a deliberately broken fixture so a criterion can be watched going red rather than taken on trust.

## 4. Limitations

- **One deployment, one machine, one user.** The 360/14 reading and the section design's own instrument are deployment-internal; the report states the same limitation for its rates.
- **The tension claim is a design argument with a measurement behind it, not an A/B.** We did not run a controlled experiment of "sections with tension semantics" against "sections without" on a frozen corpus. What is measured is the failure (freshness, verbatim collapse, retention drop) and the mechanism change; the claim that the change improves continuity rests on the design argument and on the retrieval-side readings in the report.
- **The retention instrument's three columns are the boundary, not the whole of information loss.** Content that is dropped but is neither a path, an error code nor an identifier is not counted by it.
