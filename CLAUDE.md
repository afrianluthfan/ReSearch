# Claude Code instructions for the ReSearch vault

This vault is a personal research workspace pairing Obsidian with Claude Code. You read across it freely; you write under a strict policy. **Read this entire file before doing any vault work.**

---

## Write policy

**You may READ any file in the vault.**

**You may WRITE files ONLY inside `90_Inbox/claude/`.** This is your sandbox. The user reviews drafts there and promotes them to their proper home via Obsidian's "Move file" (Cmd-P).

**You may NOT edit the body content of any existing note outside the sandbox.** Ever. To propose a change to an existing note, write `90_Inbox/claude/proposed--<original-name>.md` containing the proposed replacement, then stop.

**One narrow carve-out** — see Bidirectional Linking below. You MAY append `[[link]]` entries to the `related:` frontmatter field of existing notes when confidence is high. This is the ONLY direct-edit exception. No body edits. No other frontmatter fields. No removals.

---

## Folder map

| Folder | What's in it |
|---|---|
| `00_Meta/` | Conventions, workflows, upgrade triggers, templates. Read for context; don't write here. |
| `10_Sources/papers/` | Paper notes (one per PDF). PDFs live in `attachments/`. |
| `10_Sources/web/` | Web article / blog captures. |
| `10_Sources/code/` | Repo and code reference notes. |
| `20_Notes/` | Atomic notes (Zettelkasten). **Flat — no sub-folders.** |
| `30_MOCs/` | Maps of Content. Topic organisers via links. |
| `40_Projects/` | Active research threads. |
| `50_Questions/` | Open research questions. |
| `60_Daily/` | YYYY-MM-DD journal entries. |
| `90_Inbox/claude/` | **Your sandbox.** All your writes land here. |
| `90_Inbox/human/` | User's raw captures awaiting triage. Read-only for you. |
| `99_Archive/` | Stale content. Read-only for you. |
| `attachments/` | PDFs, images. |
| `sandbox/` | Planning artifacts — exclude from vault searches. |
| `.claude/` | Skills, subagents, settings — exclude from vault searches. |

---

## Frontmatter schema

**Required on every note:**

```yaml
type: source | note | moc | question | project | daily
status: inbox | active | reviewed | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [kebab-case-only]
```

**Required when `type: source`:**

```yaml
source_type: paper | web | code | thinking
source_url: <URL or DOI>
fetched: YYYY-MM-DD
authors: [<name>]
year: <int>
```

**Optional anywhere:**

```yaml
related: ["[[Note A]]", "[[Note B]]"]
aliases: [<string>]
author: claude              # set ONLY when YOU wrote the note
confidence: low | medium | high   # for synthesis notes only
pdf_path: <absolute path>   # for papers
```

**Tag rules:** kebab-case, **single level**. `bayesian-inference`, never `methods/bayesian/inference`. Nesting kills the tag pane and Bases filters.

**Forbidden fields:** `priority`, `rating`, `score`. They decay into noise.

---

## Filename conventions

| Note type | Filename |
|---|---|
| Source (web / code) | `YYYY-MM-DD-<slug>.md` |
| Source (paper) | `YYYY-MM-DD-<lastname>-<year>-<slug>.md` |
| Daily | `YYYY-MM-DD.md` |
| Atomic / MOC / Question / Project | `<slug>.md` |
| Sandbox proposal for body change | `proposed--<original-name>.md` |
| Sandbox synthesis | `synthesis--<topic>--YYYY-MM-DD.md` |

---

## Workflows (skills auto-trigger on natural language)

| Skill | Auto-fires on phrases like... |
|---|---|
| `ingest-url` | "ingest this URL", "save this article", "capture this blog", paste of a URL |
| `ingest-pdf` | "ingest this paper", "read this PDF", reference to a `.pdf` path |
| `synthesize` | "what do my notes say about X", "synthesize my thinking on X" |
| `find-related` | "what's related to this", "find related notes for X" |

Detailed step recipes live in `.claude/skills/<name>/SKILL.md` and in `00_Meta/workflows.md`.

**Always call the `vault-librarian` subagent BEFORE drafting** any synthesis or ingesting a source, to surface duplicates / existing notes / tag drift. It is read-only.

---

## Bidirectional linking

**Forward direction** (new note → old notes): every ingestion / synthesis skill inserts `[[wikilinks]]` to existing related notes in the new note's body and `related:` frontmatter. Always automatic.

**Reverse direction** (old note → new note): two layers.

**Layer 1 — Obsidian's automatic backlinks pane.** No file modification needed. Use this for casual discovery. This is the default.

**Layer 2 — `related:` frontmatter carve-out** (high-confidence cases only). Append `[[<new-note>]]` to an older note's `related:` field when ANY trigger fires:

- ≥2 shared tags between the new and old note, OR
- ≥1 shared tag PLUS lexical match on ≥2 key nouns, OR
- User explicitly asked, OR
- (For synthesis only) the old note is cited in the new synthesis — citation is itself strong signal; single-tag-match suffices.

**Operation rules (absolute):**

- Append-only to `related:`. Never reorder, never remove, never replace.
- Idempotent — skip if `[[link]]` already present.
- Bump `updated:` to today on the modified note.
- Touch ONLY the `related:` field. NO other frontmatter changes. NO body content modification, ever.
- Medium confidence (single signal) → surface in chat and ask. Don't auto-apply.

---

## Linking discipline

- Prefer `[[Note Title]]` over `[[Note Title|alias]]`.
- New atomic notes should link to ≥1 existing MOC where possible.
- Don't artificially inflate links. Quality over quantity.

---

## What NOT to do

- Don't edit body content of existing notes. Use `proposed--<name>.md` in the sandbox.
- Don't modify any frontmatter field other than `related:` on existing notes.
- Don't use deep tag hierarchies. Flat kebab-case only.
- Don't create sub-folders inside `20_Notes/`.
- Don't auto-summarise sources without verbatim capture. Capture comes BEFORE summary, always.
- Don't fill the `## My Notes` section of paper templates — that's the user's annotation space.
- Don't store PDFs inline in notes. Use `attachments/` and reference via `pdf_path`.
- Don't treat your own drafts as ground truth. Default `confidence: medium`; bump only after user verification.
- Don't propose vector DB / SQLite / MCP infra without a trigger from `00_Meta/upgrade-triggers.md`.
- Don't install Obsidian plugins for the user — recommend only.
- Don't create placeholder example notes. MOCs and tags are emergent, not designed.

---

## Upgrade path

See `00_Meta/upgrade-triggers.md`. Do NOT propose new infrastructure unless a trigger fires.

---

## Tool guidance

- **WebFetch** for known URLs. **WebSearch** for discovery.
- Use the `defuddle` skill before WebFetch on cluttered pages — strips chrome, saves tokens.
- Always set `source_url` and `fetched` on source notes.
- Download free PDFs to `attachments/` only after user permission. Record path in `pdf_path`.
- Use the `obsidian-markdown` skill for native Obsidian syntax (callouts, embeds, properties).
- Use the `obsidian-bases` skill when authoring `.base` files.
- Exclude `sandbox/`, `.claude/`, and `attachments/` from vault content searches.
