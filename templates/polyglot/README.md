# Polyglot skill templates

Starter files for building a skill that spans Rust, TypeScript, and Go (or any mix). Copy these into `skills/<your-skill-name>/` and edit in place — names are placeholders.

## When to use this pattern

Use the polyglot shape when the skill's **core topic is language-neutral** (a wire format, a spec, a primitive) but the **idioms diverge meaningfully** across ecosystems — enough that a single SKILL.md would either bloat past ~500 lines or paper over real interop hazards.

Landed examples in this repo:

- `skills/atproto-cid/` — CID parsing/construction/validation across Rust, TS, Go.
- `skills/atproto-identity-resolution/` — handle ↔ DID resolution across Rust, TS, Go.
- `skills/atproto-repository/` — CAR + MST + commit signing across Rust, TS, Go.
- `skills/atproto-oauth/` — OAuth 2.1 + AT Proto profile across Rust, TS (Node + Browser), Go.
- `skills/atproto-lexicon/` — lexicon + XRPC + record parsing across Rust, TS, Go.

If your topic doesn't fit that shape, use `../SKILL.md.template` instead.

## The files

| File                                     | Copy to                                         | Purpose                               |
| ---------------------------------------- | ----------------------------------------------- | ------------------------------------- |
| `../POLYGLOT-SKILL.md.template`          | `skills/<name>/SKILL.md`                        | Router. Defaults, detection, reading. |
| `shared/spec.md.template`                | `skills/<name>/shared/<spec-file>.md`           | Normative language-neutral spec.      |
| `shared/divergence-matrix.md.template`   | `skills/<name>/shared/divergence-matrix.md`     | Cross-language differences.           |
| `shared/test-vectors.md.template`        | `skills/<name>/shared/test-vectors.md`          | Fixtures used by every language.      |
| `LANG/README.md.template`                | `skills/<name>/{rust,typescript,go}/README.md`  | Per-language setup and idioms.        |
| `LANG/topic.md.template`                 | `skills/<name>/{rust,typescript,go}/<task>.md`  | Per-(language, task) guide. Match filenames across languages. |

## Rules the final skill must honor

From `CLAUDE.md` §1 and the research note at `.context/attachments/pasted_text_2026-04-21_12-28-08.txt`:

1. **SKILL.md is a router.** Defaults + language detection + reading guide + cross-language hazards. Keep it under ~150 lines. The procedure lives in `{lang}/*.md`, not here.
2. **`shared/` is a peer.** Not under `references/`. Language-neutral material only — spec, byte layouts, test vectors, divergence matrix.
3. **Every supported language gets a directory.** At minimum `README.md` plus the topical task files. Match filenames across languages (`rust/parsing.md` ↔ `typescript/parsing.md` ↔ `go/parsing.md`) so the router's reading guide works without branching per language.
4. **Divergence matrix is the interop pivot.** Per-language files link back to `shared/divergence-matrix.md` instead of restating differences. This is where someone debugging "why do bytes from language A fail in language B" lands first.
5. **Unsupported languages get an escape hatch.** The router should name `shared/spec.md` and the most complete reference implementation so callers can transliterate rather than hit a dead end.
6. **Description field drives triggering.** Name the supported languages, library and import names for each, common error strings, and what the skill does NOT cover. The description is the only signal Claude uses to load the skill — optimize for keyword density, not brevity.

## Authoring order

Roughly what works (from building the landed skills):

1. Draft `shared/spec.md` first — it anchors everything else.
2. Collect `shared/test-vectors.md` from upstream or by computing with one trusted implementation.
3. For each language: write `README.md`, then each `<task>.md`. Do the languages sequentially — it surfaces divergences you'll want in the matrix.
4. Write `shared/divergence-matrix.md` AFTER the per-language files. Trying to write it first leads to fiction; writing it last lets the real ergonomic gaps fall out.
5. Write `SKILL.md` last. You know the defaults, the hazards, and the filenames only after the rest exists.
6. Keep the router under ~150 lines. If it's longer, content belongs in `shared/` or `{lang}/`.
