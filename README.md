# ReSearch Vault — Architecture Guide

A guide to how this Obsidian + Claude Code research workspace is structured, why it's built this way, and how to use it.

This is the architecture explainer. For day-to-day rules Claude must follow, see `CLAUDE.md` (the contract Claude reads on every session).

---

## 1. What this vault is

A personal research workspace pairing **Obsidian** (a local markdown editor with bidirectional links) with **Claude Code** (an AI assistant running on your filesystem). The vault is a corpus of plain markdown files — no database, no proprietary format — that Claude can navigate, augment, and synthesize, while you keep curatorial control.

**The core problem it solves:** capturing research material (papers, articles, code refs, your own thoughts) in a way that:

- Doesn't lose the source material — verbatim capture before summary
- Doesn't pollute your hand-written notes with AI drafts — sandbox isolation
- Surfaces connections automatically — Obsidian's graph + Claude's reading
- Stays portable — plain markdown, no lock-in, grep-able forever

---

## 2. The Obsidian + Claude Code pairing

| Tool | Role |
|---|---|
| **Obsidian** | UI, link graph, backlinks pane, tag pane, Bases (queryable views), Templater, manual curation |
| **Claude Code** | Reads anywhere, writes to a sandbox folder, captures sources verbatim, suggests links, drafts synthesis notes |

Obsidian provides the **graph database substitute** — wikilinks (`[[Note Name]]`) form a directed graph that Obsidian indexes and traverses natively. Claude doesn't need a separate graph DB; it reads files and follows links.

Obsidian provides the **structured query layer** — the **Bases** feature (built in, no plugin needed) gives you Notion-style table views over YAML frontmatter. No SQLite needed for "show me all papers from 2024 with status: reviewed".

Together they cover what you'd otherwise need 4–5 tools for, on flat files.

---

## 3. Folder structure: why the numbers

The top-level folders are prefixed with two-digit numbers (`00_`, `10_`, `20_`, …). This is a **Johnny.Decimal-style** organization, chosen for three benefits:

1. **Deterministic sort order in any file browser.** `00_Meta/` always appears first; `99_Archive/` always last. No `_underscore_` hacks, no surprises.
2. **Gaps allow insertion** — you can add `15_Drafts/` between `10_Sources/` and `20_Notes/` without renumbering everything.
3. **Stable references** — when you point a teammate to "the `10_Sources/papers/` folder," that path is predictable and unambiguous.

| Folder          | Purpose                                                            | Why this number                                      |
| --------------- | ------------------------------------------------------------------ | ---------------------------------------------------- |
| `00_Meta/`      | Conventions, templates, workflows — the vault's instruction manual | `00` sorts to top; readers see context first         |
| `10_Sources/`   | External material (papers, web, code)                              | `10` — first "real" content folder                   |
| `20_Notes/`     | Atomic notes — your own writing, one idea per note (Zettelkasten)  | `20` — second tier; depends on sources               |
| `30_MOCs/`      | Maps of Content — topic indexes built from `[[links]]`             | `30` — emerges from atomic notes                     |
| `40_Projects/`  | Active research threads with goal and next steps                   | `40` — depends on notes + sources                    |
| `50_Questions/` | Open research questions awaiting answers                           | `50` — orthogonal axis to projects                   |
| `60_Daily/`     | Date-stamped journal entries                                       | `60` — temporal log                                  |
| `90_Inbox/`     | Triage zone (Claude's sandbox + your raw captures)                 | `90` — at the bottom because it's churn, not curated |
| `99_Archive/`   | Stale content kept for reference                                   | `99` — last by design                                |
| `attachments/`  | PDFs, images (Obsidian's default attachment folder)                | no number — Obsidian convention                      |

**Sub-folders only appear where templates genuinely differ:**

- `10_Sources/{papers,web,code}/` — different ingestion templates per source type
- `90_Inbox/{claude,human}/` — different write policies per actor

**`20_Notes/` is intentionally flat.** Sub-folders fragment the Zettelkasten — you end up with the same idea filed under three different paths because you weren't sure where it belonged. Tags and MOCs do the categorization. The cost of "no sub-folders" is occasionally scrolling a long list; the benefit is that every atomic note is discoverable by grep, by tag, or via any MOC that points to it.

---

## 4. The sandbox-folder write policy

**Claude can read anywhere in the vault. Claude can write ONLY to `90_Inbox/claude/`.**

This is enforced by the contract in `CLAUDE.md` (Claude reads it at the start of every session). The policy exists because:

- Claude drafts should be **reviewable before they enter the curated knowledge base.** The sandbox is a staging area.
- **Filesystem-level isolation** beats alternatives:
  - Filename prefixes (`_draft_*.md`) couple policy to naming → easy to forget
  - Frontmatter flags (`author: claude`) require every tool to parse YAML
  - Git branching is overkill for a personal vault and Obsidian doesn't surface branch state

**The promotion workflow:**

1. Open the draft in `90_Inbox/claude/`
2. Review, edit, fix errors
3. Change `status: inbox → reviewed`
4. **Use Obsidian's "Move file" (Cmd-P)** — this auto-rewrites every `[[link]]` pointing at the moved file
5. Drop `author: claude` if it's now substantially your own work

**The one narrow exception:** Claude MAY append `[[links]]` to the `related:` frontmatter field of existing notes when confidence is high (see §10 below). This is `related:`-only, append-only, idempotent, with no body edits.

To propose a change to an existing note's body content, Claude writes `90_Inbox/claude/proposed--<original-name>.md` with the suggested replacement, and stops.

---

## 5. Frontmatter schema

Every note has YAML frontmatter at the top:

```yaml
type: source | note | moc | question | project | daily
status: inbox | active | reviewed | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [kebab-case-only]
```

Source notes add:

```yaml
source_type: paper | web | code | thinking
source_url: <URL or DOI>
fetched: YYYY-MM-DD
authors: [<name>]
year: <int>
```

Optional fields:

```yaml
related: ["[[Note A]]", "[[Note B]]"]
aliases: [<string>]
author: claude              # set ONLY when Claude wrote the note
confidence: low | medium | high
pdf_path: <absolute path>   # for papers
```

**Why these fields specifically:**

- `type` and `status` drive Bases views (e.g., "all sources I haven't reviewed yet")
- `created` and `updated` enable stale-draft detection
- `tags` are kebab-case **single-level** because nested tags (`methods/bayesian`) break Obsidian's tag pane and Bases filters
- `source_url`/`fetched` mean you can re-acquire the original if the local copy is corrupted
- `confidence` flags AI drafts as not-yet-verified
- `author: claude` is a *secondary* signal — the folder is the primary gate

**Forbidden fields:** `priority`, `rating`, `score`. They decay into noise. If you need urgency, use `status: active`.

---

## 6. Filename conventions

| Note type | Filename | Why |
|---|---|---|
| Source (web / code) | `YYYY-MM-DD-<slug>.md` | Date prefix sorts chronologically; you remember "the article from March" |
| Source (paper) | `YYYY-MM-DD-<lastname>-<year>-<slug>.md` | Author lastname + paper year makes it citation-grep-able |
| Daily | `YYYY-MM-DD.md` | Obvious temporal sort |
| Atomic / MOC / Question / Project | `<slug>.md` | Evergreen — no temporal meaning; clean slug is the title |
| Sandbox proposal | `proposed--<original-name>.md` | Double-dash separator makes the intent unambiguous |
| Sandbox synthesis | `synthesis--<topic>--YYYY-MM-DD.md` | Topic + date so multiple syntheses on one topic coexist |

The double-dash in sandbox files is intentional — it visually separates the "what kind of draft" prefix from the actual subject.

---

## 7. The linking model: wikilinks as a graph

Obsidian's `[[wikilink]]` syntax creates a directed edge between two notes. The collection of all wikilinks forms a graph that Obsidian indexes and renders in:

- **Graph view** — visual node/edge diagram
- **Backlinks pane** — for any note, every note that links *to* it
- **Outgoing links pane** — every note this one links *from*

This replaces what you'd otherwise put in a graph database. Claude navigates the graph by reading a file's contents and following the link targets. At sub-10,000 notes, this beats a real graph DB on every axis: simpler, faster to set up, no sync, no second source of truth.

**Linking discipline:**

- Prefer `[[Note Title]]` over aliased `[[Note Title|alias]]` — aliases break grep
- New atomic notes should link to ≥1 existing MOC where possible
- Don't artificially inflate links — quality over quantity

---

## 8. Verbatim capture before summary

**The single biggest failure mode** for AI-assisted research vaults is letting the AI write a summary as the *primary* record. You lose:

- The ability to re-interpret the source later
- Direct quotes you can use in your own writing
- A check against hallucination

Every source template has this structure:

```markdown
## Capture
> "Verbatim quote 1"
> — page / paragraph ref

> "Verbatim quote 2"
> — page / paragraph ref

## Summary
[Claude's interpretation, in its own words, after the capture]
```

Capture comes first. Summary is secondary. This rule is enforced by both the templates and the skill prompts.

---

## 9. Workflows and skills

Claude has four custom skills installed in `.claude/skills/` that auto-trigger on matching phrases — you don't type a slash command.

| Skill | Auto-fires on phrases like... | What it does |
|---|---|---|
| `ingest-url` | "ingest this URL", "save this article", "capture this blog", pasting a URL | WebFetch + verbatim capture + sandbox draft |
| `ingest-pdf` | "ingest this paper", "read this PDF", reference a `.pdf` path | PDF read + paper template + sandbox draft |
| `synthesize` | "what do my notes say about X", "synthesize my thinking on X" | Search vault + draft synthesis in sandbox |
| `find-related` | "what's related to X", "suggest links for this note" | Return ranked candidates; optionally apply links |

Three vendor skills from [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) (the Obsidian CEO's pack):

- `obsidian-markdown` — teaches Claude native Obsidian syntax (callouts, embeds, properties)
- `defuddle` — strips chrome from web pages before WebFetch (reduces tokens dramatically)
- `obsidian-bases` — teaches Claude `.base` file syntax (filters, formulas, views)

One read-only subagent at `.claude/agents/vault-librarian.md`:

- Invoke with "what do I have on X?" for a quick inventory
- Used internally by `synthesize` to find duplicates before drafting
- Strictly read-only — has no Write tool access

---

## 10. Bidirectional linking

Links flow two ways in this vault:

**Forward** (new note → old notes): every ingestion / synthesis skill inserts `[[wikilinks]]` to existing related notes in the new note's body and `related:` frontmatter at creation time. Always automatic.

**Reverse** (old note → new note): two layers.

**Layer 1 — Obsidian's automatic backlinks pane.** For any note, Obsidian's sidebar shows everything that links to it. Zero file modification needed. This handles most "what's related" questions and is the default.

**Layer 2 — `related:` frontmatter carve-out** (high-confidence cases only). Claude MAY append `[[<new-note>]]` to an older note's `related:` field when ANY trigger fires:

- ≥2 shared tags between the new and old note, OR
- ≥1 shared tag PLUS lexical match on ≥2 key nouns, OR
- User explicitly asked, OR
- For synthesis only: the old note is cited (citation is strong signal; single-tag-match suffices)

**Absolute operation rules:**

- Append-only. Never reorder, remove, or replace existing entries.
- Idempotent — skip if the link is already present.
- Bump `updated:` to today's date on the modified note.
- Touch ONLY the `related:` field. NO other frontmatter changes. NO body content modification, ever.
- Medium confidence (single signal) → surface in chat and ask. Don't auto-apply.

**Why this is safe:** `related:` is metadata, not prose. Appending to it doesn't change what the note *says* — only how it surfaces in queries and link views. The body of the note (where meaning lives) stays under the absolute sandbox rule.

---

## 11. Upgrade triggers: when to add infrastructure

The vault deliberately starts with **no database layer.** Plain markdown + Obsidian + Claude is sufficient at this scale (a few hundred notes).

**Add a vector layer (via MCP server) when ANY of these fires:**

1. Ripgrep for a typical concept returns >50 hits and you can't pick the 5 to read
2. You ask Claude the same synthesis question twice in a month because you forgot the earlier answer existed
3. Corpus crosses ~800 notes OR ~2M tokens of markdown, whichever first
4. You search for ideas using vocabulary not present in your notes (semantic gap) more than once a week

**Add SQLite only if** Obsidian Bases can't aggregate the cross-vault time-series questions you're actually asking. Almost never necessary.

**Do NOT upgrade for:** "it would be cool", "I read about RAG", or note count alone.

Full checklist: `00_Meta/upgrade-triggers.md`.

---

## 12. What's NOT in the architecture (and why)

| Not included | Why |
|---|---|
| Vector database | At <1K notes, ripgrep beats embeddings. Defer behind triggers. |
| SQLite mirror | Obsidian Bases handles structured queries. Mirror = drift. |
| Graph database | Wikilinks ARE a directed graph. A real graph DB only earns its keep above ~10K nodes. |
| MCP servers | Token-intensive at session start; nothing requires them yet. |
| Custom Templater JS scripts | Maintenance liability. Plain template files suffice. |
| Git versioning | Obsidian Sync / iCloud handles versioning for personal vaults. Add only if collaborating. |
| Fixed `tags.md` taxonomy | Tags emerge from real use; designed taxonomies rot. |
| Pre-filled placeholder MOCs | MOCs are *emergent*. Build them when you have 5+ notes on a topic, not before. |
| Plugin auto-installation | I recommend; you install. Plugin settings need GUI judgment. |
| Slash command for promotion | Promotion is a 5-second manual act in Obsidian. Building a command for it is over-engineering. |

---

## 13. How to use this vault day-to-day

**Capturing a web article:**

1. Open Claude Code in this directory
2. Say: *"ingest https://example.com/some-article"*
3. The `ingest-url` skill auto-fires
4. Claude writes `90_Inbox/claude/YYYY-MM-DD-<slug>.md` with verbatim capture, summary, and forward links
5. Review, change `status: inbox → reviewed`, Cmd-P → Move to `10_Sources/web/`

**Ingesting a PDF:**

1. Drop the PDF in `attachments/`
2. Say: *"ingest this paper /absolute/path/to/file.pdf"*
3. The `ingest-pdf` skill fires
4. Claude reads the PDF, drafts a paper note in `90_Inbox/claude/` with `pdf_path:` pointing at the file
5. Review, promote to `10_Sources/papers/`

**Writing your own atomic notes:**

- Use Templater (Cmd-P → "Templater: Insert template") to scaffold an atomic note
- Write the idea in 1–3 paragraphs
- Link to ≥1 MOC and to any source notes that support it
- Save directly in `20_Notes/` — no sandbox needed for your own writing

**Asking Claude to synthesize:**

1. Say: *"what do my notes say about [topic]?"*
2. The `synthesize` skill fires; calls `vault-librarian` to inventory existing notes
3. Claude drafts `90_Inbox/claude/synthesis--<topic>--YYYY-MM-DD.md` with citations
4. Cited source notes may get `[[<synthesis-note>]]` appended to their `related:` field (high-confidence triggers only)
5. Review and promote to `20_Notes/`

**Daily journal:**

- Use Templater to create `60_Daily/YYYY-MM-DD.md` each day
- Capture what you read, what you thought, questions raised, plans for tomorrow

**Weekly review:**

- Open `00_Meta/stale-sandbox.base` in Obsidian → table view of Claude drafts older than 7 days in `90_Inbox/claude/`
- Decide for each: promote, refine, or delete

---

## 14. For collaborators and new readers

If you've just been pointed at this vault and want to understand what it is:

1. **Read `CLAUDE.md`** first if you're going to run Claude Code in this directory. It's the contract.
2. **Read this file (`README.md`)** for the architecture rationale.
3. Skim `00_Meta/conventions.md` for the long-form schema rules.
4. Skim `00_Meta/workflows.md` for narrative descriptions of each workflow.
5. Check `00_Meta/upgrade-triggers.md` to see when to suggest new infrastructure.

**Key constraints to know:**

- Claude writes ONLY to `90_Inbox/claude/` — don't ask it to edit `20_Notes/<existing>.md` directly; ask for a `proposed--<name>.md` draft instead
- Tags are kebab-case, single-level (`bayesian-inference`, not `methods/bayesian/inference`)
- `20_Notes/` is flat — no sub-folders
- Source notes must have verbatim quotes in `## Capture` before any `## Summary`
- Root-level files (`CLAUDE.md`, `README.md`) are setup-time documentation, not research content — don't put research notes there

---

## 15. Reference index

| File | What it is |
|---|---|
| `CLAUDE.md` | Contract Claude reads on every session — write policy, schema, workflows |
| `README.md` | This file — full architecture explainer + quick start |
| `00_Meta/conventions.md` | Long-form frontmatter / tag / filename rules |
| `00_Meta/workflows.md` | Narrative descriptions of the six workflows |
| `00_Meta/upgrade-triggers.md` | Checklist for when to add infrastructure |
| `00_Meta/stale-sandbox.base` | Bases view of Claude drafts >7 days old |
| `00_Meta/templates/*.md` | 8 templates Claude and Templater both consume |
| `.claude/skills/*/SKILL.md` | Custom and vendor skills |
| `.claude/agents/vault-librarian.md` | Read-only inventory subagent |

---

## 16. Design philosophy in one paragraph

This vault is built on the bet that **plain markdown plus a strong contract beats sophisticated infrastructure** for personal research at sub-1000-note scale. Everything you need — a graph (wikilinks), structured queries (Bases), full-text search (ripgrep), AI assistance (Claude Code) — works on flat markdown files. Heavier infrastructure (vector DBs, SQLite, MCPs) is deferred behind concrete triggers so it gets added when there's an actual problem to solve, not speculatively. The sandbox-folder write policy means Claude's contributions are always reviewable before they enter the curated knowledge base. Verbatim capture before summary means you never lose the source material. The structure is designed to age gracefully: rename a note and Obsidian rewrites every link; promote a sandbox draft and the same thing happens. The whole vault is portable, grep-able, and outlives any single tool that touches it.
