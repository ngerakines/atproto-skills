# Contributing

`atproto-skills` is a Claude Code plugin: a collection of ATProtocol-specific skills that agents load on demand. If you work on ATProto — lexicons, DIDs, XRPC, DAG-CBOR, OAuth, feeds — and keep re-explaining the same context to agents, that's a skill waiting to be written.

## Where the real guide lives

The authoring conventions — frontmatter schema, how to write a `description` that actually triggers, progressive disclosure between `SKILL.md` / `references/` / `scripts/`, the MCP tools skills can call — are in [`CLAUDE.md`](./CLAUDE.md). Read that first. This file is the short, human-facing companion.

## Adding a new skill

1. Pick a name. Kebab-case, under 30 characters, specific (`atproto-firehose`, not `events`). Landed skills prefix with `atproto-`; follow suit unless there's a good reason not to.
2. `cp templates/SKILL.md.template skills/<your-skill-name>/SKILL.md`
3. Edit the frontmatter and body per `CLAUDE.md` §1–§3.
4. If the skill has deep content, add `skills/<your-skill-name>/references/` and link from `SKILL.md`.
5. If it has deterministic helpers, add `skills/<your-skill-name>/scripts/`.
6. Test locally (`CLAUDE.md` §8) and iterate on the `description` until the skill fires reliably.
7. Open a PR.

## PR checklist

- [ ] Skill directory name matches `name:` in frontmatter (kebab-case).
- [ ] `description:` is third-person, names concrete ATProto terms, and distinguishes the skill from adjacent ones.
- [ ] `SKILL.md` body is under 500 lines; long content lives in `references/`.
- [ ] Any MCP tools mentioned exist on `lexicon-garden` or `atpmcp` (see `CLAUDE.md` §5).
- [ ] Manually verified the skill triggers on at least two realistic user phrases.
- [ ] Backlog entry in `CLAUDE.md` §6 updated or removed if your skill fills one.

## Proposing a skill before writing it

If you're not sure whether a skill belongs here, or it overlaps with one in the backlog (`CLAUDE.md` §6), open an issue first. A one-paragraph pitch is enough: what it covers, what triggers it, and how it differs from neighbors.
