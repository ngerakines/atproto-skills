# atproto-skills

A Claude Code plugin that packages ATProtocol-specific skills — lexicon design, DID resolution, XRPC, DAG-CBOR, MST/repo inspection, OAuth flows, record parsing — so agents can load specialized knowledge on demand instead of relying on generic recall.

## Status

Active. Seven skills shipped:

- `atproto-attestation` — badge.blue record attestations (inline + remote), ECDSA low-S, two-CID model. Rust · TypeScript · Go.
- `atproto-cid` — CID parsing, construction, and validation (DASL / BDASL). Rust · TypeScript · Go.
- `atproto-identity-resolution` — handle ↔ DID resolution, DID documents, bidirectional verification. Rust · TypeScript · Go.
- `atproto-lexicon` — lexicon authoring, schema validation, XRPC invocation, record parsing. Rust · TypeScript · Go.
- `atproto-oauth` — OAuth 2.1 + AT Proto profile (PAR, DPoP, PKCE, `private_key_jwt`). Rust · TypeScript (Node + Browser) · Go.
- `atproto-publish-lexicon` — publish lexicons as `com.atproto.lexicon.schema` records; NSID authority binding.
- `atproto-repository` — CAR v1, MST, DRISL canonical CBOR, commit signing/verification. Rust · TypeScript · Go.

Six are polyglot (language-neutral spec with per-language guides); `atproto-publish-lexicon` is a single-file skill. Skills load automatically when their trigger terms appear in conversation — see each skill's `description` for the full trigger surface.

Bluesky-domain record idioms (`app.bsky.*` facets, richtext, embeds, threadgates, labels) are out of scope for this plugin.

## Install

The repo ships a marketplace manifest at `.claude-plugin/marketplace.json`. Add it to your Claude Code config:

```json
{
  "extraKnownMarketplaces": {
    "atproto-skills": {
      "source": {
        "source": "github",
        "repo": "ngerakines/atproto-skills"
      }
    }
  },
  "enabledPlugins": {
    "atproto-skills@atproto-skills": true
  }
}
```

Prefer to work from a local clone? Swap the source:

```json
"source": { "source": "filesystem", "path": "/absolute/path/to/atproto-skills" }
```

## Use

Skills are loaded by Claude Code on demand based on their `description` field. You don't invoke them directly — just work on ATProto code and talk about what you're doing in natural language ("my `did:plc` lookup is returning 404", "help me design a lexicon for…"). When a skill matches, the agent pulls it into context automatically.

## Skills

Each skill lives at [`skills/<name>/`](./skills/) with a polyglot structure (`shared/` + `rust/` / `typescript/` / `go/`) unless noted otherwise. The scenarios below are the kinds of prompts that will load the skill.

### `atproto-attestation`

Inline and remote record attestations per the [badge.blue](https://badge.blue/) specification. Rust · TypeScript · Go.

- "Sign this record with an inline ECDSA attestation and append it to `signatures[]`."
- "Write a remote attestation: put a proof record in my repo, reference it from the subject via `strongRef`."
- "Verify the attestations on this record I just fetched — inline and remote both."
- "I'm getting `RemoteAttestationCidMismatch` / `SignatureValidationFailed` — walk me through what could be off."
- "Port my Rust `atproto-attestation` signing code to `@noble/curves` in TypeScript."

### `atproto-cid`

CID parsing, construction, and validation under the DASL / BDASL profile. Rust · TypeScript · Go.

- "Parse `bafyreib...` and tell me whether it's a valid DASL CID."
- "Compute the CID for this DAG-CBOR record / this raw blob."
- "Decode the `$link` form in this JSON and the CBOR tag-42 form in this binary."
- "My encoder and my PDS disagree on the CID — what's wrong with my canonicalization?"
- "Add BLAKE3 (BDASL) support for hashing large video blobs."

### `atproto-identity-resolution`

Handle ↔ DID resolution, DID document handling, and bidirectional verification. Rust · TypeScript · Go.

- "Resolve `alice.bsky.social` to a DID — try DNS TXT and `/.well-known/atproto-did` concurrently."
- "Resolve `did:plc:…` to a DID document, extract the PDS endpoint and signing key."
- "Run bidirectional verification: handle → DID → `alsoKnownAs` → handle."
- "My user's handle is showing `handle.invalid` — what's the recovery flow?"
- "Why does this `did:web` resolve but `did:webvh` rejects at fetch time in my library?"

### `atproto-lexicon`

Lexicon authoring, schema validation, XRPC invocation, and protocol-level record parsing. Rust · TypeScript · Go.

- "Author a lexicon for a `com.example.notes.entry` record with an open union and a `strongRef`."
- "Validate this record against its lexicon — strict on write, lenient on read."
- "Invoke `com.atproto.repo.listRecords` with typed inputs/outputs from my Go client."
- "Consume the firehose and dispatch on `$type` without a codegen step."
- "I want to add a field to an existing lexicon — is this change backward-compatible?"

### `atproto-oauth`

OAuth 2.1 + AT Protocol profile: PAR, DPoP, PKCE, `private_key_jwt`, client metadata, session storage. Rust · TypeScript (Node + Browser) · Go.

- "Stand up a confidential BFF client: `/oauth/authorize`, `/oauth/callback`, `/oauth/refresh`, `/oauth/logout`."
- "Publish my `/oauth-client-metadata.json` with the correct keys, scopes, and `dpop_bound_access_tokens`."
- "Build a public SPA client that authenticates with DPoP alone — no client secret."
- "I'm getting `use_dpop_nonce` on resource requests — how do I handle the nonce cycle?"
- "Two nodes both tried to refresh the same token and one got `invalid_grant` — mitigate the race."
- "Design a permission set with `repo:*`, `rpc:*`, and `include:` clauses."

### `atproto-publish-lexicon`

Publishing lexicons as `com.atproto.lexicon.schema` records and resolving them back by NSID. Single-file skill.

- "Publish my new `com.example.foo.getBar` lexicon to my PDS — full pipeline."
- "Bump the revision on an existing lexicon and gate the publish on a compatibility check."
- "Resolve an unknown NSID back to its lexicon document: DNS `_lexicon.` → DID → PDS → record."
- "My lexicon published fine but consumers can't resolve it — check authority, rkey, and the `_lexicon.` TXT binding."
- "What's the social contract for `revision` bumps vs. minting a new NSID?"

### `atproto-repository`

CAR v1, MST, DRISL canonical CBOR, commit signing and verification. Rust · TypeScript · Go.

- "Parse a CAR v1 export from `com.atproto.sync.getRepo` and walk the MST."
- "Verify the commit signature on this CAR against the account's signing key."
- "Insert a record, recompute the MST root, produce a signed commit."
- "Diff two MSTs and emit firehose-style create/update/delete ops."
- "I'm hitting `prev: null` vs. omitted behavior mismatch between two implementations — how do I align?"
- "My block store says `CID mismatch` on one block — which canonical-CBOR rule did I break?"

## Add a skill

1. Pick the right template:
   - `templates/SKILL.md.template` for a single-file skill (narrow topic, or a topic that doesn't diverge meaningfully across languages).
   - `templates/POLYGLOT-SKILL.md.template` + the scaffolding under `templates/polyglot/` when the topic is language-neutral but the idioms differ across Rust, TypeScript, or Go. See [`templates/polyglot/README.md`](./templates/polyglot/) for the layout it produces.
2. Copy it into `skills/<your-skill-name>/` and fill in the frontmatter (`name` must match the directory) plus the body.
3. Read [`CLAUDE.md`](./CLAUDE.md) — the authoring guide covering skill anatomy, `description` field conventions, and the local testing loop.
4. Open a PR; see [`CONTRIBUTING.md`](./CONTRIBUTING.md) for the checklist.

## License

MIT. See [`LICENSE`](./LICENSE).
