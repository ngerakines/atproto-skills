# atproto-skills

A Claude Code plugin that packages ATProtocol-specific skills — lexicon design, DID resolution, XRPC, DAG-CBOR, MST/repo inspection, OAuth flows, record parsing — so agents can load specialized knowledge on demand instead of relying on generic recall.

## Status

Active. Five polyglot skills shipped (Rust + TypeScript + Go), covering:

- `atproto-cid` — CID parsing, construction, and validation (DASL / BDASL).
- `atproto-identity-resolution` — handle ↔ DID resolution and DID document handling.
- `atproto-repository` — CAR v1, MST, DRISL canonical CBOR, commit signing.
- `atproto-oauth` — OAuth 2.1 + AT Proto profile: PAR, DPoP, PKCE, private_key_jwt.
- `atproto-lexicon` — lexicon authoring, validation, XRPC invocation, record parsing.

See [`CLAUDE.md`](./CLAUDE.md) §6 for status and scope boundaries.

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

## Add a skill

1. Copy `templates/SKILL.md.template` into `skills/<your-skill-name>/SKILL.md`.
2. Read [`CLAUDE.md`](./CLAUDE.md) — it is the authoring guide.
3. Open a PR; see [`CONTRIBUTING.md`](./CONTRIBUTING.md) for the checklist.

## Repo layout

```
atproto-skills/
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest
│   └── marketplace.json    # marketplace entry for local install
├── skills/
│   ├── atproto-cid/
│   ├── atproto-identity-resolution/
│   ├── atproto-lexicon/
│   ├── atproto-oauth/
│   └── atproto-repository/
├── templates/
│   ├── SKILL.md.template           # starter for single-language skills
│   ├── POLYGLOT-SKILL.md.template  # router for polyglot skills
│   └── polyglot/                   # per-language and shared template files
├── CLAUDE.md               # authoring guide (read this)
├── CONTRIBUTING.md         # PR workflow
├── LICENSE                 # MIT
└── README.md               # you are here
```

## License

MIT. See [`LICENSE`](./LICENSE).
