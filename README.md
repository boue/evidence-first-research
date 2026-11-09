# evidence-first-research

A public [Agent Skill](https://skills.sh) for evidence-first research gates.

It tells an agent how to preserve original bytes, admit sources before generation, require exact same-run passages, keep model / citation / human stamps separate, and append human review only after a run is closed. Promotion stays a separate host write.

This repo is the **gate contract**. Hosts supply a rejecting store. Batcave’s fetch, SQLite evidence store, and claim-grounding modules are one adapter; HN, YouTube, product dates, and Insights UI stay in that host.

## Install

```bash
npx skills add boue/evidence-first-research --global
```

## Not this skill

- Not a bot (no schedule, no chat channel).
- Not a research agent that fetches, drafts, reviews, and promotes in one loop.
- Not a Batcave-only feature dump.

## Layout

- `SKILL.md` — procedure, hard stops, and store port.

## License

MIT
