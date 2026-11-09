# evidence-first-research

A public [Agent Skill](https://skills.sh) that stops research agents from treating titles, summaries, or model confidence as proof.

## Why it exists

Most agentic research collapses three different things into one answer:

1. what the model *thinks*
2. whether a citation actually resolves to original source text
3. whether a human accepted the claim

That is how you get fluent briefs that cite paraphrases, promote unverifiable claims, and hide fetch failures. This skill forces the opposite order: **keep original bytes, admit sources before generation, require exact same-run passages, separate the three stamps, and only then allow a human review.** Promotion, if any, is a later explicit host write—not a side effect of labeling.

## Value

- **Auditability** — every kept claim points at an exact passage in an immutable snapshot.
- **Honest failure** — missing, blocked, or thin sources stay excluded instead of being papered over with titles or summaries.
- **Stamp separation** — model support, citation validity, and human review cannot overwrite each other.
- **Safer reuse** — any host that implements the rejecting store port can run the same gate without copying a product UI or scheduler.

## Install

```bash
npx skills add boue/evidence-first-research --global
```

## How it helps

The skill is a **procedure plus a store contract**. Your host supplies a rejecting store (`saveSnapshot`, `openRun`, `recordAdmission`, `saveClaimProposals`, `recordReview`, `readFrame`). If that port is missing, the skill stops instead of inventing soft storage.

Agents following it will:

- preserve original source bytes
- admit or exclude before generation
- freeze admission once claims exist
- accept only exact passages from the same run
- append human review only after the run is closed
- refuse to treat a model label as promotion

## Not this skill

- Not a bot (no schedule, no chat channel).
- Not a research agent that fetches, drafts, reviews, and promotes in one loop.
- Not a product-specific dashboard or source connector pack. Discovery, UI, dates, and delivery stay in the host.

## Layout

- `SKILL.md` — procedure, hard stops, and store port.

## License

MIT
