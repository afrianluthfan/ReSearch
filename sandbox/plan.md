# ReSearch Vault — Architecture & Setup Plan

## Context

You're standing up a personal research workspace at `/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/` that pairs Obsidian with Claude Code. The directory is currently empty.

**Usage profile (confirmed):**
- All four source types: academic PDFs, web articles, personal synthesis, code/technical refs.
- Claude reads anywhere but writes ONLY to a sandbox folder; you review and promote drafts.
- Heavy use of WebFetch/WebSearch — Claude actively scouts and files findings.
- Medium scale (a few hundred notes in 6 months), with a clean upgrade path if it grows.

**Goal:** A markdown-first vault with explicit conventions so Claude can navigate it predictably, capture sources without polluting your hand-written notes, and never lose the original source material. No databases initially; defer that decision until a concrete trigger fires.

---

## Architecture decisions (with justifications)

### 1. No database layer — Obsidian's primitives suffice at this scale

| Question | Decision | Why |
|---|---|---|
| Graph DB? | **No.** | `[[wikilinks]]` ARE a directed graph. Claude traverses by reading a file and following links. Real graph DBs only earn their keep above ~10K nodes. |
| SQLite? | **No.** | YAML frontmatter + Obsidian Bases gives structured views and filters over your metadata, natively. A SQLite mirror would drift from the vault and double the sync surface. |
| Vector DB? | **No, not yet.** | At a few hundred notes, ripgrep beats embeddings — your corpus is small enough that recall isn't the bottleneck. Concrete upgrade triggers below. |

**Upgrade path when needed:** expose a vector store via an MCP server. The vault's markdown content does NOT change — only retrieval gets a new layer.

### 2. Sandbox-folder write policy

Claude writes ONLY to `90_Inbox/claude/`. Filesystem-level isolation beats:
- **Filename prefixes** (`_draft_*.md`) — couples policy to naming, easy to forget.
- **Frontmatter flags** (`author: claude`) — requires every tool to parse YAML to trust provenance.
- **Git branching** — overkill for a personal vault, Obsidian doesn't surface branch state.

Keep `author: claude` as a *secondary* frontmatter field — lets a Base view filter to "all Claude drafts older than 7 days" — but the folder is the gate.

**One narrow carve-out** (see "Bidirectional linking policy" below): Claude MAY append `[[new-note]]` to the `related:` frontmatter field of older notes when confidence is high. This is the ONLY exception — append-only, `related:` field only, no body edits, no other field changes, no removals.

### 3. Flat Zettelkasten, deep sources

- `20_Notes/` stays **flat** — sub-folders fragment the Zettelkasten and make grep miss things. Tags + MOCs do the categorising.
- `10_Sources/` has 3 sub-folders (`papers/`, `web/`, `code/`) because the templates and ingestion workflows genuinely differ.

### 4. Capture verbatim before summarising

This is the single biggest research-vault failure mode: letting Claude write summaries as the *primary* record. You lose the ability to re-interpret the source. Every source note has a `## Capture` section with verbatim quotes; the `## Summary` is secondary.

---

## Folder structure

```
ReSearch/
├── CLAUDE.md                    # Vault navigation + write policy + workflows
├── README.md                    # Human-facing overview
├── 00_Meta/
│   ├── conventions.md           # Frontmatter schema, naming, tag rules
│   ├── workflows.md             # Detailed workflow recipes for Claude
│   ├── upgrade-triggers.md      # When to add vector DB / SQLite
│   └── templates/               # 8 templates, consumed by Templater AND Claude
│       ├── source-paper.md
│       ├── source-web.md
│       ├── source-code.md
│       ├── atomic-note.md
│       ├── moc.md
│       ├── question.md
│       ├── project.md
│       └── daily.md
├── 10_Sources/
│   ├── papers/                  # One note per PDF (PDF in attachments/)
│   ├── web/                     # Article/blog captures
│   └── code/                    # Repo/snippet refs
├── 20_Notes/                    # Atomic notes — FLAT, no sub-folders
├── 30_MOCs/                     # Maps of Content
├── 40_Projects/                 # Active research threads
├── 50_Questions/                # Open questions
├── 60_Daily/                    # YYYY-MM-DD journal entries
├── 90_Inbox/
│   ├── claude/                  # Claude's sandbox — ONLY writable location
│   └── human/                   # Your raw captures before triage
├── 99_Archive/                  # Stale content
└── attachments/                 # PDFs, images (Obsidian's default attachment dir)
```

---

## Frontmatter schema

```yaml
# REQUIRED on every note
type: source | note | moc | question | project | daily
status: inbox | active | reviewed | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [kebab-case-only]

# REQUIRED for type: source
source_type: paper | web | code | thinking
source_url: <string>          # canonical URL or DOI
fetched: YYYY-MM-DD
authors: [<string>]
year: <int>

# OPTIONAL
related: ["[[Note A]]"]       # explicit cross-links beyond inline
aliases: [<string>]           # retitling without breaking links
author: claude                # set ONLY by Claude in sandbox
confidence: low | medium | high
```

**Rules:**
- Tags are kebab-case, **single level**. `bayesian-inference` not `methods/bayesian/inference`. Nesting kills the tag pane and Bases filters.
- `status: inbox` is the default for sandbox writes; you change it on review.
- No `priority`/`rating`/`score` fields — they decay into noise.

---

## Filename conventions

| Note type | Filename |
|---|---|
| Source (paper/web/code) | `YYYY-MM-DD-<slug>.md` |
| Daily | `YYYY-MM-DD.md` |
| Atomic note / MOC / Question / Project | `<slug>.md` (no date) |
| Sandbox proposal to edit existing note | `proposed--<original-name>.md` |
| Sandbox synthesis | `synthesis--<topic>--YYYY-MM-DD.md` |

---

## Templates (8 files in `00_Meta/templates/`)

Each carries its frontmatter block plus skeleton headings.

| Template | Body sections |
|---|---|
| `source-paper.md` | Citation / Abstract (verbatim) / Key Claims / Methods / Evidence Quality / My Notes / Open Questions / Related |
| `source-web.md` | Capture (verbatim quotes) / Summary / Why It Matters / Related |
| `source-code.md` | What It Is / How It Works / When To Use / Caveats / Related |
| `atomic-note.md` | Single idea in 1–3 paragraphs / Sources / Related |
| `moc.md` | Scope / Core Notes / Sources / Open Questions (all bullet lists of links) |
| `question.md` | Question / Why It Matters / What I've Tried / Candidate Answers / Sources Consulted |
| `project.md` | Goal / Current State / Next Steps / Notes Touched / Sources |
| `daily.md` | What I Read / What I Thought / Questions Raised / Tomorrow |

---

## CLAUDE.md sections (top-down, < 250 lines)

1. **Write Policy** (top, unmissable) — read anywhere, write only `90_Inbox/claude/`; to modify an existing note, write `proposed--<name>.md` to the sandbox instead.
2. **Folder Map** — one line per top-level folder.
3. **Frontmatter Schema** — the spec from above, one example per type.
4. **Filename Conventions** — the table above.
5. **Workflows** — six numbered recipes (see next section).
6. **Linking Discipline** — prefer `[[Note Title]]` over aliased links; new notes should link to ≥1 existing MOC.
7. **What NOT to do** — anti-patterns list.
8. **Upgrade Path** — pointer to `00_Meta/upgrade-triggers.md`; do not propose new infra unless a trigger fires.
9. **Tool Guidance** — when to WebFetch vs WebSearch; always set `source_url` + `fetched`; download free PDFs to `attachments/`.

---

## Workflows (Claude's exact step sequences)

**Ingest URL:**
1. WebFetch → extract title, author, date.
2. Apply `source-web.md` template → `90_Inbox/claude/YYYY-MM-DD-<slug>.md`.
3. Quote 3–8 passages verbatim in `## Capture`; write `## Summary` in own words.
4. Grep `20_Notes/` for existing tags before proposing new ones. Add 1–3 `[[links]]` if matches exist.
5. **Reverse-link enrichment** (per Bidirectional Linking policy): for any older note that triggers the high-confidence rule, append `[[<new-note>]]` to its `related:` frontmatter.
6. Stop; report sandbox path, proposed forward links, and which old notes were enriched.

**Ingest PDF:**
1. Read PDF; extract title, authors, year, DOI/arXiv ID.
2. Confirm with user before moving PDF into `attachments/`.
3. Apply `source-paper.md` → sandbox.
4. `## Key Claims`: 3–7 bullets, each with page reference.
5. Flag methodological caveats in `## Evidence Quality`.
6. **Reverse-link enrichment** (per Bidirectional Linking policy): for high-confidence old-note matches, append `[[<new-note>]]` to their `related:` frontmatter.

**Synthesise across notes:**
1. Ripgrep `20_Notes/` and `10_Sources/` for the topic + obvious synonyms.
2. Read top ~15 hits; follow `[[wikilinks]]` one hop.
3. Write new atomic note: `synthesis--<topic>--YYYY-MM-DD.md`, `type:note, author:claude, confidence:medium`.
4. Cite `[[source-note]]` per claim. Flag contradictions explicitly. Do NOT modify source-note BODIES.
5. **Reverse-link enrichment** (per Bidirectional Linking policy): for each cited source note, append `[[synthesis--<topic>--<date>]]` to its `related:` frontmatter (single-tag-match is enough here, since citation is itself a strong signal).

**Find related:**
1. Read the seed note; extract tags, links, key nouns.
2. Ripgrep vault for key nouns; check tag overlap.
3. Return ranked list **in chat**.
4. **Apply mode** (per Bidirectional Linking policy): if user says "apply these" or candidates are high-confidence, append `[[<seed>]]` to each candidate's `related:` frontmatter. Frontmatter-only; never touch body content.

**Promote from sandbox** (user action — see next section).

**Ask a question:**
1. Create `question.md`-based draft in sandbox.
2. Pre-fill `## Sources Consulted` with anything already read on the topic.

---

## Promotion / review workflow

1. Open `90_Inbox/claude/<note>.md` in Obsidian.
2. Review; change `status: inbox → reviewed`; fix errors.
3. Use Obsidian's "Move file" (Cmd-P) — **this is the killer feature**: Obsidian auto-rewrites all `[[links]]` pointing at the file.
4. Drop `author: claude` if the note is now substantially your own; keep it if still mostly Claude's.

**A weekly check** (a Base view at `00_Meta/stale-sandbox.base`): lists everything in `90_Inbox/claude/` older than 7 days so stale drafts surface.

**Don't build now**: a `/promote` slash command, an MCP server, a git hook. Promotion is a 5-second manual act and that's the point.

---

## Bidirectional linking policy

**Forward direction** (new note → old note): every ingestion / synthesis skill inserts `[[wikilinks]]` to existing related notes during creation. Always automated. No carve-out needed since the new note is in the sandbox.

**Reverse direction** (old note → new note): two layers.

**Layer 1 — Obsidian backlinks (default, automatic, zero modification):**
Obsidian's backlinks pane on any note shows everything that links to it. No file modification needed. This handles the vast majority of "what's related" questions and is the default — Claude does NOT modify old notes just to make connections visible.

**Layer 2 — `related:` frontmatter carve-out (high-confidence cases):**
Claude MAY append `[[new-note]]` to the `related:` frontmatter of an older note when ANY of these triggers fires:

- ≥2 shared tags between the new and old note, OR
- ≥1 shared tag PLUS lexical match on ≥2 key nouns, OR
- Explicit user instruction ("link these notes", "add to related").

**Operation rules** (absolute):
- Append-only. Never remove or reorder existing entries.
- Idempotent: skip if `[[new-note]]` is already in `related:`.
- Bump `updated:` to today's date on the modified note.
- Touch ONLY the `related:` field — no other frontmatter changes.
- Touch NO body content.

**Confidence handling:**
- **High** (multiple signals): auto-apply.
- **Medium** (single signal): surface in chat, ask before applying.
- **Low** (no clear signal): do nothing; rely on Layer 1 backlinks.

**Why this is safe:** `related:` is metadata, not prose. Appending to it doesn't change what the note says — only how it surfaces in queries and link views. Body content (where meaning lives) stays under the absolute sandbox rule.

---

## Upgrade triggers — when to add infrastructure

Add a **vector layer (via MCP server)** when ANY one fires:
1. Ripgrep for a typical concept returns **>50 hits** and you can't pick the 5 to read.
2. You ask Claude the same synthesis question twice in a month because you forgot the earlier answer existed.
3. Corpus crosses **~800 notes** OR **~2M tokens of markdown**, whichever first.
4. You search for ideas using vocabulary not present in your notes (semantic gap) more than once a week.

Add **SQLite** only if Bases can't aggregate cross-vault time-series questions you're actually asking. Almost never necessary.

Do NOT upgrade for: "it would be cool", "I read about RAG", or note count alone.

---

## Anti-patterns (encoded into CLAUDE.md)

- Deep tag hierarchies — flat kebab-case only.
- Sub-folders inside `20_Notes/`.
- Letting Claude edit existing notes' BODY content "just this once" — the body-edit rule is absolute. The only carve-out is appending to `related:` frontmatter under the conditions in the Bidirectional Linking section.
- Auto-summarising sources without verbatim capture — irreversible information loss.
- Mega-MOCs (>30 links) — split.
- Base views / dashboards that look impressive but nobody reads.
- Treating Claude drafts as ground truth — `confidence: medium` default; verify before promoting.
- Storing PDFs inline in notes — kills grep and sync.
- Daily notes as primary capture — they become a swamp. Capture goes to `90_Inbox/`.

---

## Implementation order (when plan is approved)

1. **Folder skeleton** — create the 12 directories (one-shot `mkdir -p`).
2. **`CLAUDE.md`** at vault root — the load-bearing file.
3. **`00_Meta/conventions.md`** — frontmatter schema, naming, tag rules. Reference to `00_Meta/stale-sandbox.base` for the weekly check.
4. **`00_Meta/workflows.md`** — full prose of the six workflows.
5. **`00_Meta/upgrade-triggers.md`** — the trigger list as a checklist.
6. **`00_Meta/templates/*.md`** — 8 template files.
7. **`00_Meta/stale-sandbox.base`** — Base view listing `90_Inbox/claude/*` files older than 7 days; pinned in the sidebar for the weekly review.
8. **`README.md`** — human-facing overview, plugin recommendations (Templater, Tag Wrangler — Bases is built into Obsidian).
9. **`.gitkeep` in each otherwise-empty folder** so the structure survives sync tools.

**Critical files to create (absolute paths):**
- `/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/CLAUDE.md`
- `/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/README.md`
- `/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/00_Meta/conventions.md`
- `/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/00_Meta/workflows.md`
- `/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/00_Meta/upgrade-triggers.md`
- `/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/00_Meta/stale-sandbox.base`
- `/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/00_Meta/templates/{source-paper,source-web,source-code,atomic-note,moc,question,project,daily}.md`

---

## Installed Claude Code automation

Already in place under `.claude/`. Skills auto-trigger on matching prompts; no need to type `/`.

**Custom workflow skills** (`.claude/skills/`):

| Skill | Triggers when you say... |
|---|---|
| `ingest-url` | "ingest this URL", "capture this article", "save this link", "file this", paste a URL |
| `ingest-pdf` | "ingest this paper", "read this PDF", "summarize this paper", reference a .pdf path |
| `synthesize` | "what do my notes say about X", "synthesize my thinking on X", "pull together what I have on X" |
| `find-related` | "what's related to this", "what connects to X", "suggest links for this note" |

**Vendor skills** (from [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) — Obsidian CEO's official pack):

| Skill | What it does |
|---|---|
| `obsidian-markdown` | Teaches native Obsidian syntax — callouts, embeds, properties, wikilinks. Auto-loads when writing `.md` files. |
| `defuddle` | Strips chrome from web pages before WebFetch returns content. Reduces tokens on URL ingestion. Pairs with `ingest-url`. |
| `obsidian-bases` | Teaches `.base` file syntax — filters, formulas, views. Auto-loads when writing `.base` files. |

**Read-only subagent** (`.claude/agents/vault-librarian.md`):

Inventory agent. `synthesize` calls it before drafting to check for existing notes / duplicates / tag drift. Strictly read-only; never writes. Invoke directly with "what do I have on X?" for a quick inventory.

**Sandbox** (`90_Inbox/claude/`): created. All Claude writes land here.

---

## What I will explicitly NOT do

- Install Obsidian plugins (you do that in the GUI; I'd be guessing at settings).
- Seed placeholder/example notes — they rot and never get deleted.
- Build a custom MCP server — no trigger met.
- Set up SQLite mirroring "for later" — speculative infra.
- Write Templater JS user scripts — maintenance liability.
- Pre-fill MOCs for topics you haven't researched — MOCs are emergent.
- Create a fixed `tags.md` taxonomy — let it grow organically.
- Add git versioning unless you ask — Obsidian Sync / iCloud handles versioning fine for a personal vault.

---

## Verification (after implementation)

1. **Structure check** — `tree -L 3 "<vault>/ReSearch"` shows the 12 directories and all listed files.
2. **CLAUDE.md round-trip** — open a fresh Claude Code session in `ReSearch/`, ask "where do you write?" — answer must be `90_Inbox/claude/` only.
3. **Template smoke test** — ask Claude to "ingest https://<some-url>" — verify it lands in `90_Inbox/claude/`, has correct frontmatter, has `## Capture` with verbatim quotes BEFORE summary.
4. **Promotion drill** — manually move a sandbox file to `10_Sources/web/`; confirm in Obsidian that any `[[links]]` were rewritten and the file opens cleanly.
5. **Anti-pattern check** — ask Claude to "fix a typo in `<some existing note>`" — Claude must refuse direct edit and write `proposed--<name>.md` to sandbox instead.
6. **Upgrade-trigger awareness** — ask Claude "should we add a vector DB?" — answer must reference `00_Meta/upgrade-triggers.md` and report which (if any) triggers are met.

If all six pass, the vault is ready to use.
