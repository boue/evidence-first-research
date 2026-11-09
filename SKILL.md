---
name: evidence-first-research
description: |
  Run research through an evidence-first gate: immutable snapshots, admit-or-exclude
  before generation, exact same-run passages, separate model/citation/human stamps,
  and append-only human review after close. Use when building or reviewing agentic
  research that must refuse titles-as-evidence, paraphrased citations, collapsed
  stamps, or silent promotion. Requires a host rejecting store; stop if none exists.
license: MIT
metadata:
  version: "1.0.0"
---

# Evidence-first research

Preserve original bytes. Connect every claim to an exact same-run passage. Keep model support, citation validity, and human review as three separate stamps. Do not promote from a model label.

This skill is a **procedure plus a store contract**. It is not a bot (no clock, no channel) and not an agent that fetches, drafts, reviews, and files in one worker.

## Hard stop

If the host has no rejecting store that implements the port below, **stop**. Do not invent tables, soft-fail writes, or treat chat memory as evidence.

## Store port

The host must expose a rejecting store with at least:

| Method | Duty |
| --- | --- |
| `saveSnapshot` | Persist original bytes (or an immutable content-addressed snapshot). Refuse mutation of stored bytes. |
| `openRun` | Open a research run with a frozen input cutoff and pipeline version. |
| `recordAdmission` | Admit or exclude a source **before** generation. Record exclusion reasons. |
| `saveClaimProposals` | Accept model proposals of evidence ids + exact passages. Quarantine invalid proposals. |
| `recordReview` | Append a human review **after** the run is closed. Idempotent on operation id. |
| `readFrame` | Read claims with separate model, citation, and human statuses. Passages only; no full snapshot dump required for review UI. |

The store **throws** (does not soft-fail) when:

- an immutable snapshot or completed-run frame would be updated in place
- an evidence id is unknown or from another run
- a passage is absent from the admitted snapshot (exact match required)
- a review is attempted before the run is closed (`completed`)
- a trusted-file write, schedule registration, or promotion path is requested through this port

No `trustedFile`, `schedule`, `promote`, `embed`, or `deliver` methods belong on this port.

## Procedure

Run these steps in order. Do not skip ahead to claims or review.

### 1. Preserve original bytes

- Fetch or load the primary source as original content (HTML article text, transcript text, or other raw body the host defines).
- Save via `saveSnapshot`. Deduplicate by source identity and content hash when the host supports it.
- **Titles, blurbs, and generated summaries are discovery context, not evidence.**

### 2. Open a run

- Call `openRun` with an input cutoff and pipeline version.
- Every later admission and claim belongs to this run id.

### 3. Admit or exclude before generation

- For each candidate source, `recordAdmission` with `admitted` or an exclusion (`ineligible`, `excluded`, `evidence_pending`, `quarantined`, etc.).
- Exclusion reasons stay on the run frame (fetch failure, too short, missing source, policy stop).
- **Do not generate claims against a source that was not admitted.**

### 4. Freeze admission before claims

- Once any claim exists for the run (valid or invalid), refuse further admission writes.
- Close the run to further source-frame changes when the host marks it `completed` or `failed`.

### 5. Model proposes; store accepts or quarantines

- Give the model **only** admitted evidence ids and excerpted snapshot text the host chose to send.
- The model may propose `{ evidenceId, passage, relation }` with an exact consecutive passage copied from that snapshot.
- Call `saveClaimProposals`. The store:
  - accepts proposals whose passage exists at a concrete offset in the same-run admitted snapshot and that include at least one `supports` link when required by the host
  - quarantines the rest with the original proposal JSON and a validation error
  - **does not** write partial link sets for a mixed valid/invalid proposal batch when the host uses all-or-nothing link rules
- Store **model support** on the claim if the model offered it. Never copy it into the human review field.

### 6. Human appends review after close

- Only after the run is `completed`, call `recordReview` with `{ reviewStatus, operationId }`.
- Allowed labels (typical): `supported`, `partly_supported`, `unsupported`, `contradicted`, `unverifiable`, `rejected`.
- `supported` / `partly_supported` require a resolved same-run supporting passage that still matches the snapshot bytes.
- The same `operationId` must not insert a second event.
- Review updates **only** human review status (and reviewed-at). It must not change evidence bytes, claim text, model support, or citation status.

### 7. Promotion is a separate explicit write

- A human `supported` label is not promotion.
- `partly_supported` stays visible and is not a promotion path.
- Promotion, if the host has it at all, is a later explicit write outside this skill’s port, with its own eligibility rules (typically citation-valid, prompt-seen, human-supported, passage still matches).

## Hard stops during fetch and generation

Stop the fetch path on **401, 403, 429, and CAPTCHA / challenge** responses. Do not bypass with proxies unless the host has a separately approved pilot outside this skill.

A missing source stays missing. Do not backfill from titles or summaries.

The agent following this skill **does not review** and **does not promote**. Model support never becomes the human label.

## Three stamps

Keep these separate in storage and in any UI:

1. **Model support** — what the generator claimed.
2. **Citation status** — whether the store resolved an exact same-run passage (`valid` / `invalid` / …).
3. **Human review** — append-only owner label after close.

Collapsing them into one “confidence” score violates this skill.

## What this skill leaves to the host

Source discovery, rankings, product dates/timezones, UI routes, schedulers, delivery channels, embeddings, and domain-specific adapters stay in the host. One known adapter shape is a fetch module + evidence store + claim-grounding module behind the port above.

## Install

```bash
npx skills add boue/evidence-first-research --global
```

Do not install into `~/.cursor/skills-cursor/` by hand unless your toolchain requires it; prefer the skills CLI.
