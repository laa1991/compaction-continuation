# Publication red lines (DATA-POLICY)

> Written before any artifact is produced. Every clause is checkable, and the scan prints what it scanned as well as what it hit.

## Never published

1. **Session transcripts.** Raw session logs, the text of model-written summaries, and verbatim spill files. They contain a person's private work and personal context; they are not ours to publish regardless of research value.
2. **Fields that locate a person.** Session identifiers, session titles, workspace or home paths of the deployment, personal names, and credential-like strings.
3. **Third-party material that is not already public.** Public sources may be quoted with a link; they are not re-hosted here.

## May be published

1. **Design text, criteria and devices we wrote**, with thresholds in the device header rather than in prose.
2. **Aggregate readings**, each with its denominator, and anonymized identifiers only (any mapping stays local).
3. **Quotations** from public sources, or from our own writing.

## Unit discipline (so that two rulers are not read as one)

- Every retention or survival number states **instrument, denominator and unit**. A row missing any of the three is not admissible.
- Counts of failures state their dedup rule.
- Where two instruments measure the same material (verbatim against semantic survival), both readings are shown side by side, never merged into one number.

## Self-check before anything leaves this machine

```sh
node tools/check-redline.mjs      # any non-exempt hit ⇒ exit 3
```

- **Line-level exemption**: the line ends with `redline-ok: <reason>`; an empty reason counts as a hit.
- **File-level exemption** applies to exactly two files: the scanner itself (its patterns are assembled there) and this policy.
- The scan prints **its surface** (files, lines) next to its hits. "Scanned little" and "scanned clean" are different readings, and the scan is written so the difference is visible.
- A run that only reports an exit code is not evidence: the passing clause prints its own success line.
