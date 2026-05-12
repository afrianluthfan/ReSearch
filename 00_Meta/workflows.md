# Research workflows

These are the recipes Claude follows when you make common research moves. Each is encoded as a skill in `.claude/skills/<name>/SKILL.md` for auto-invocation — Claude triggers them when your prompt matches, no need to type `/`.

This file is the human-readable narrative. The skills are the executable form.

---

## 1. Ingest a URL

**Trigger:** "ingest this URL", "capture this article", paste of a URL.

**What happens:**

1. Claude calls `vault-librarian` to check for duplicates (same `source_url` already captured?).
2. WebFetch the URL. Use `defuddle` skill on cluttered pages to strip chrome.
3. Extract title, author, publication date, canonical URL.
4. Draft a source note in `90_Inbox/claude/YYYY-MM-DD-<slug>.md` using the `source-web.md` template.
5. `## Capture`: 3–8 verbatim quotes, varying lengths. **Capture goes BEFORE summary.**
6. `## Summary`: 1–3 paragraphs in Claude's own words.
7. Reverse-link enrichment: append `[[<new-note>]]` to `related:` of any older notes that hit the high-confidence threshold.
8. Stop. Report sandbox path, proposed forward links, and enriched old notes.

**You then:** review, edit, change `status: inbox → reviewed`, Cmd-P → "Move file" to `10_Sources/web/`.

---

## 2. Ingest a PDF

**Trigger:** "ingest this paper", "read this PDF", reference to a `.pdf` path.

**What happens:**

1. Claude calls `vault-librarian` to check for duplicates (same DOI / arXiv ID).
2. Read the PDF. For >10 pages, start with pages 1–10 and request more as needed.
3. Extract metadata: title, authors, year, DOI / arXiv ID, venue.
4. Confirm with you before moving the PDF into `attachments/`.
5. Draft a paper note in `90_Inbox/claude/YYYY-MM-DD-<lastname>-<year>-<slug>.md` using `source-paper.md`.
6. `## Abstract`: copied verbatim, in a blockquote.
7. `## Key Claims`: 3–7 bullets, each with a page reference.
8. `## Evidence Quality`: honest — flag p-hacking, small N, replication issues, conflicts of interest.
9. `## My Notes`: **left empty.** That's your annotation space.
10. Reverse-link enrichment: append to `related:` of relevant old notes (tag match, author match, or explicit request).

**You then:** annotate `## My Notes`, promote.

---

## 3. Synthesise across notes

**Trigger:** "what do my notes say about X", "synthesize my thinking on X".

**What happens:**

1. Claude calls `vault-librarian` first to check for existing syntheses on the topic.
2. If a recent synthesis exists (<90 days), Claude surfaces it and asks: extend, or fresh?
3. Ripgrep `20_Notes/` and `10_Sources/` for the topic + 2–4 obvious synonyms.
4. Read top ~15 hits in full. Follow `[[wikilinks]]` one hop.
5. Draft a new note: `90_Inbox/claude/synthesis--<topic>--YYYY-MM-DD.md` with `type:note, author:claude, confidence:medium`.
6. Every claim is bullet-pointed and cites ≥1 source note.
7. Contradictions surface explicitly. Don't paper them over.
8. **Reverse-link enrichment** (citation is strong signal — auto-applied): append `[[synthesis--<topic>--<date>]]` to `related:` of EVERY cited source note.
9. Stop. Report path, claim count, contradiction count.

**You then:** read, verify against sources, bump `confidence: medium → high` only if it holds up.

---

## 4. Find related

**Trigger:** "what's related to this", "find related notes for X", "suggest links".

**What happens:**

1. Read the seed note (or topic). Extract tags, existing wikilinks, key nouns.
2. Ripgrep the vault for key nouns. Check tag overlap. Find existing backlinks.
3. Rank candidates by signal strength (tag overlap > existing link > lexical match).
4. Return a ranked list **in chat** (max 15).
5. If you say "apply these" or candidates are high-confidence: append `[[<seed>]]` to each candidate's `related:` frontmatter (per Bidirectional Linking policy).

**You then:** decide which suggestions are real. Optionally ask "apply these" to write them to `related:`.

---

## 5. Ask a question

**Trigger:** "log this as a question", "I'm wondering about X", or when a question surfaces during synthesis.

**What happens:**

1. Apply `question.md` template → `90_Inbox/claude/<slug>.md`.
2. Pre-fill `## Sources Consulted` with anything Claude has already read on the topic.
3. Stop.

**You then:** flesh out, promote to `50_Questions/`.

---

## 6. Promote from sandbox (you do this, not Claude)

1. Open `90_Inbox/claude/<note>.md` in Obsidian.
2. Review per `00_Meta/conventions.md` → "Promotion checklist".
3. Edit `status: inbox → reviewed`.
4. Cmd-P → "Move file" to the right folder. Obsidian auto-rewrites `[[links]]`.
5. Optionally drop `author: claude` if the note is now substantially your own.

**Why this is manual:** the sandbox-then-promote pattern works precisely because promotion is a deliberate, friction-light human act. Don't automate this.

---

## On the carve-out (reverse-link enrichment)

Workflows 1, 2, 3, and 4 may modify the `related:` frontmatter of existing notes. This is the ONLY direct-edit exception to the sandbox policy. The rules:

- Append-only to `related:`. No reorders, no removes, no replaces.
- Idempotent (skip if `[[link]]` already present).
- Bump `updated:` on the modified note.
- Touch ONLY the `related:` field. NO other frontmatter changes. NO body content modification, ever.
- Medium confidence (single signal) → ask in chat, don't auto-apply.

See `CLAUDE.md` → "Bidirectional linking" for the full policy.
