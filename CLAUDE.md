# Authoring Guide: `atproto-skills`

This repository is a Claude Code plugin that packages ATProtocol-specific skills. The goal is that any agent doing ATProto work — designing lexicons, resolving DIDs, writing XRPC clients, decoding DAG-CBOR, auditing OAuth flows — can load the right specialized knowledge on demand instead of relying on generic training-time recall.

This file is the authoring guide. It is also loaded into the context of any agent working inside this repo, so its conventions apply to every skill added here.

## 1. Skill anatomy

Every skill is a single directory under `skills/`. The directory name is the skill name.

```
skills/<skill-name>/
├── SKILL.md          ← required. Frontmatter + body.
├── references/       ← optional. Long-form material loaded on demand.
├── scripts/          ← optional. Deterministic helpers (validators, parsers).
└── assets/           ← optional. Templates copied into user outputs. Not loaded into context.
```

`SKILL.md` frontmatter:

| Field         | Required | Notes                                                        |
| ------------- | -------- | ------------------------------------------------------------ |
| `name`        | yes      | Must match the directory. kebab-case. <30 chars.             |
| `description` | yes      | The trigger mechanism — see §3.                              |
| `version`     | no       | Semver. Conventional but not enforced.                       |

Body conventions:
- Keep `SKILL.md` under **500 lines**. Spill anything longer into `references/` and link to it.
- Lead with *When to use* and *What this skill provides*. Put the *Procedure* next. Everything else is supporting material.
- Reference MCP tools by name (see §6) rather than duplicating what they already do.

The canonical starter is `templates/SKILL.md.template`. Copy it, don't retype it.

### Polyglot skills (optional shape)

When a skill's core topic is language-neutral but its idioms differ meaningfully across ecosystems (CIDs, DAG-CBOR, XRPC client design, OAuth flows), use the polyglot layout instead of `references/`:

```
skills/<skill-name>/
├── SKILL.md          ← thin router. Defaults + language detection + reading guide.
├── shared/           ← language-neutral material. Spec, binary layouts, test vectors, divergence matrix.
│   ├── spec.md
│   ├── test-vectors.md
│   └── divergence-matrix.md
├── rust/             ← one sibling per supported language.
│   ├── README.md     ← crate setup, library choice, idioms.
│   ├── parsing.md    ← input → in-memory value.
│   ├── construction.md ← in-memory value → output.
│   └── codecs.md     ← or other topical files as needed.
├── typescript/
│   └── …
└── go/
    └── …
```

Rules:

- `SKILL.md` is a **router**, not a tutorial. It lists defaults, tells Claude how to detect the target language (project files, file extension, import paths), and points at `{lang}/*.md`. Keep it under ~150 lines.
- `shared/` is peer to the language directories (not under them). It holds the normative spec and anything language-neutral.
- Every supported language gets its own subdirectory with at least a `README.md` (setup + idioms) and the task files the topic needs. Match filenames across languages so cross-references work (`rust/parsing.md`, `typescript/parsing.md`, `go/parsing.md`).
- `shared/divergence-matrix.md` is the pivot point for porting and interop review. Each per-language file links back to it instead of restating the matrix.
- If a language can't be supported yet, say so in `SKILL.md` and point at `shared/spec.md` + a reference implementation.

The canonical example is `skills/atproto-cid/` — mirror its layout when adding a polyglot skill.

## 2. Progressive disclosure

Agents pay a context cost for everything they load, so each skill should be shaped like a funnel:

| Layer        | Loaded when           | What belongs here                                                   |
| ------------ | --------------------- | ------------------------------------------------------------------- |
| `description`| always, as metadata   | A few sentences. The trigger surface. Loaded even when skill is idle. |
| `SKILL.md`   | skill is triggered    | Procedure, decision rules, short examples. ≤500 lines.              |
| `references/`| link-followed         | Spec excerpts, long examples, reference tables.                     |
| `scripts/`   | invoked deterministically | Executable helpers. Claude calls them instead of reasoning.     |
| `assets/`    | copied into outputs   | Boilerplate, templates. Never read into context.                    |

If a piece of content isn't needed to *trigger* or *execute* the main flow, push it out of `SKILL.md` and into `references/`.

## 3. Writing effective `description` fields

`description` is the only signal Claude uses to decide whether to load this skill. A bad description means the skill sits idle forever. Rules:

- **Third person.** "This skill should be used when the user…". Not "I…" and not "you…".
- **Name the concrete terms.** List the ATProto vocabulary this skill handles (see §4). Agents often trigger on surface-level keyword match; give them keywords to match.
- **Describe the trigger situations, not the implementation.** "debugging a CBOR decode error in a `com.atproto.repo.getRecord` response" is a trigger. "Implements a CBOR decoder" is an implementation note.
- **Call out adjacent skills.** If a sibling skill covers repo internals and yours covers record shapes, say so — "covers record structure; for raw CAR/MST traversal use `repo-inspector`".
- **Err on the side of over-triggering.** An idle skill is worse than a slightly over-eager one. The agent can always ignore the guidance.

**Before:**
> Helps with DIDs.

**After:**
> This skill should be used when the user is resolving, validating, or debugging an AT Protocol DID — including `did:plc`, `did:web`, and `did:webvh`. Triggers on phrases like "resolve handle", "DID document", "signing key rotation", "PDS endpoint", or errors from `com.atproto.identity.resolveHandle`. Does NOT cover lexicon design (see `atproto-lexicon`) or OAuth token flows (see `atproto-oauth`).

## 4. ATProto vocabulary checklist

Skill descriptions should draw trigger terms from this vocabulary. If a user mentions something on this list, *some* skill here should fire.

- Identity: `DID`, `did:plc`, `did:web`, `did:webvh`, handle, handle resolution, signing key, rotation key
- Network: PDS, AppView, relay, feed generator, labeler
- Addressing: `at://` URI, NSID, TID, CID
- Data: record, commit, MST (Merkle Search Tree), CAR, DAG-CBOR, block store
- API surface: XRPC, `com.atproto.*`, `app.bsky.*`, lexicon, query vs procedure
- Streaming: firehose, jetstream, subscribeRepos, event cursor
- Auth: OAuth, DPoP, PAR, scopes, client metadata, service auth (JWT), app passwords
- Moderation: labels, label value definitions, takedowns, thread gate

## 5. MCP tools already available

Skills should call existing MCP infrastructure instead of reimplementing it. When writing procedures, name the tool and its parameters directly.

**`lexicon-garden`** (remote, HTTP)
- `describe_me` — authorize the session and return the caller's identity.
- `describe_lexicon` — fetch and explain a lexicon by NSID.
- `validate_lexicon` — check a lexicon document for correctness.
- `invoke_xrpc` — call an XRPC method with typed inputs/outputs.
- `transmogrify_record` — transform records between representations.
- `create_record_cid` — compute a record CID per spec.
- `facet_text` — parse/produce Bluesky facets (mentions, links, tags).
- `check_compatibility` / `discover_permission_sets` — OAuth scope tooling.

**`atpmcp`** (local)
- `resolve_handle_to_did`, `resolve_identity` — identity lookups.
- `get_record`, `invoke_xrpc`, `validate_xrpc` — record and XRPC utilities.
- `generate_tid`, `create_record_cid` — spec-conformant generators.
- `parse_facets`, `transmogrify_record`, `get_lexicon`, `validate_lexicon_schema` — record and lexicon tooling.

When in doubt, prefer `lexicon-garden` for spec-conformant behavior and `atpmcp` for local development against a PDS.

## 6. Candidate skill backlog

These are the skills this repo is intended to grow into. They are deliberately *not* implemented yet — they're listed so skill authors know the neighborhood and can describe their own skills against a stable map.

| Skill                  | Status  | Triggers on                                                              |
| ---------------------- | ------- | ------------------------------------------------------------------------ |
| `atproto-cid`                   | landed  | CID parsing/construction/validation, `$link`, tag 42, `bafyrei`/`bafkrei`. |
| `atproto-identity-resolution`   | landed  | Handle ↔ DID resolution, DNS TXT `_atproto.`, `/.well-known/atproto-did`, DID document shape, `handle.invalid`, bidirectional verification. Supersedes the original `did-resolver` backlog entry. |
| `atproto-repository`            | landed  | CAR v1 parse/emit, MST node/tree/diff, DRISL canonical CBOR, commit signing & verification, `at://<did>/<collection>/<rkey>`, TID rkeys. Supersedes the `repo-inspector` and `mst-debugger` backlog entries. |
| `atproto-oauth`                 | landed  | OAuth 2.1 + AT Proto profile: PAR, DPoP, PKCE, `private_key_jwt`, client metadata documents, scope/permission-set design, session storage, refresh-race handling. Rust / TypeScript (Node + Browser) / Go. Supersedes the `oauth-at-flow` backlog entry. |
| `atproto-lexicon`               | landed  | Lexicon authoring, schema validation, backward-compat checks, XRPC method invocation (`query`/`procedure`/`subscription`), and protocol-level record parsing (`$type` dispatch, `strongRef`, blob refs, `at://` refs inside record bodies). Binds lexicons + XRPC + record parsing into one skill because they share the schema model. Supersedes the `lexicon-designer`, `xrpc-client-builder`, and `record-parser` backlog entries. Does NOT cover Bluesky-domain record idioms (facets, richtext, embeds, threadgates) — those are out of scope for this plugin. |
| `atproto-publish-lexicon`       | landed  | Publishing lexicons as `com.atproto.lexicon.schema` records and resolving them back by NSID. Covers the record shape (`id == rkey`, `lexicon: 1`, `revision`), DNS `_lexicon.<authority>` TXT authority binding, pre-publish compatibility checks, and `com.atproto.repo.putRecord` mechanics. Paired with `atproto-lexicon`: that sibling owns the JSON document (authoring, validation, invocation); this skill owns the record on the network (publish + resolve). Descriptions explicitly disclaim each boundary so they don't shadow each other. |
| `atproto-attestation`           | landed  | Creating, parsing, and verifying inline (ECDSA-signed) and remote (content-addressed strongRef) record attestations per the badge.blue specification. Polyglot: Rust, TypeScript, Go. Covers the CID-first signing pipeline (`record + $sig(repository)` → DAG-CBOR → SHA-256 → CIDv1 → ECDSA → low-S → base64), the two-CID model for remote proofs, P-256/K-256 support with the P-384 normalization gap in the reference crate, and replay protection via `repository` binding. Anchored on the `atproto-attestation` Rust crate at ngerakines.me/atproto-crates. Does NOT cover generic record parsing/XRPC (→ `atproto-lexicon`), DID resolution (→ `atproto-identity-resolution`), CAR/MST/commit signing (→ `atproto-repository`), or CID construction outside attestation contexts (→ `atproto-cid`). |

The previously-listed `dag-cbor-explain` entry has been dropped: DRISL has one canonical home in `atproto-repository` §drisl, and CID tag-42 encoding lives in `atproto-cid`. A third skill would duplicate content and split trigger surface.

Bluesky-domain record idioms (`app.bsky.*` facets, richtext, embeds, threadgates, labels, feed/thread view hydration) are **out of scope** for this plugin. They have a distinct library surface (`@atproto/api`) and belong in a separate downstream plugin if one ever exists. The `atproto-lexicon` skill's `description` explicitly disclaims them and points users at the Bluesky appview docs.

Adding one is an explicit goal, not a surprise — just make sure the `description` differentiates it from its neighbors (see §3).

## 7. How to add a new skill

1. `cp templates/SKILL.md.template skills/<your-skill-name>/SKILL.md`
2. Fill in `name` (must match the directory), `description` (read §3 first), and the body sections.
3. If you need deep content, create `skills/<your-skill-name>/references/` and link from `SKILL.md`.
4. If you need deterministic helpers, create `skills/<your-skill-name>/scripts/` and call them from the *Procedure* section.
5. Test locally (§8).
6. Open a PR. See `CONTRIBUTING.md` for the checklist.

## 8. Local testing loop

The shortest path to verify a skill actually triggers:

1. Enable the plugin locally by adding this repo path to your Claude Code plugins config (or installing the local marketplace entry).
2. Start a fresh Claude Code session and prompt using language a real user would — e.g. "my `did:plc` lookup is returning 404, help me debug" for `did-resolver`.
3. Confirm the skill loads. If it doesn't, the `description` is too narrow or not keyword-dense enough. Edit it, restart the session, retry.
4. Once triggering is reliable, check the procedure actually runs end-to-end against the MCP tools it names.

Iterating on `description` is the main loop. Treat it like a spec for the trigger, not a marketing blurb.
