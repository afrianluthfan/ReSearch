# Vault conventions

Authoritative reference for frontmatter, naming, tags, and the promotion workflow. `CLAUDE.md` summarises this for Claude; this file is the long form for humans.

---

## Frontmatter schema

### Required on every note

```yaml
type: source | note | moc | question | project | daily
status: inbox | active | reviewed | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [kebab-case-only]
```

### Required when `type: source`

```yaml
source_type: paper | web | code | thinking
source_url: <URL or DOI>
fetched: YYYY-MM-DD
authors: [<name>]
year: <int>
```

### Optional

```yaml
related: ["[[Note A]]", "[[Note B]]"]   # explicit cross-links
aliases: [<string>]                     # for retitling without breaking links
author: claude                          # set only on Claude-authored notes
confidence: low | medium | high         # synthesis notes only
pdf_path: <absolute path>               # papers
```

### Forbidden fields

`priority`, `rating`, `score` — they decay into noise. Don't add them.

### Example: paper source

```yaml
---
type: source
source_type: paper
status: reviewed
created: 2026-05-10
updated: 2026-05-12
tags: [bayesian-inference, neural-networks]
source_url: https://arxiv.org/abs/2024.01234
fetched: 2026-05-10
authors: [Smith, J., Doe, A.]
year: 2024
pdf_path: /Users/bankraya/.../attachments/smith-2024.pdf
related: ["[[bayesian-deep-learning-MOC]]"]
---
```

### Example: atomic note

```yaml
---
type: note
status: active
created: 2026-05-12
updated: 2026-05-12
tags: [epistemic-uncertainty]
related: ["[[bayesian-inference-MOC]]"]
---
```

---

## Tag taxonomy

- **Single level only.** `bayesian-inference`, not `methods/bayesian/inference`.
- **kebab-case.** Lowercase, hyphens between words.
- **Coin sparingly.** Before adding a new tag, check existing ones — the `vault-librarian` subagent can surface them.
- **Forbidden tags:** `important`, `interesting`, `read-later`, `to-read`, `wip`. They tag everything and nothing.
- **Use Tag Wrangler** periodically to merge near-duplicates (e.g., `bayesian` vs `bayesian-inference`).

---

## Filename conventions

| Note type | Filename |
|---|---|
| Source (web, code) | `YYYY-MM-DD-<slug>.md` |
| Source (paper) | `YYYY-MM-DD-<lastname>-<year>-<slug>.md` |
| Daily | `YYYY-MM-DD.md` |
| Atomic / MOC / Question / Project | `<slug>.md` |
| Sandbox proposal for body change | `proposed--<original-name>.md` |
| Sandbox synthesis | `synthesis--<topic>--YYYY-MM-DD.md` |

**Slug rules:** lowercase, hyphens between words, max ~60 chars, no special characters. Strip articles ("a", "the") and stop words where possible.

---

## Promotion checklist (Claude draft → vault)

When promoting a file from `90_Inbox/claude/`:

1. Open the file in Obsidian.
2. Verify frontmatter:
   - `tags` are kebab-case, single level, and existing-where-possible.
   - `related:` `[[links]]` resolve to real notes.
   - Source-specific fields filled (`source_url`, `fetched`, `authors`, etc.).
3. Verify body:
   - Verbatim quotes are accurate (compare to the source).
   - Summary is in your voice. If you rewrote substantially, remove `author: claude`.
4. Change `status:` from `inbox` to `reviewed`.
5. Bump `updated:` to today.
6. Use Cmd-P → "Move file" to relocate. **Obsidian rewrites all `[[links]]` automatically.**

---

## Weekly review

Once a week (e.g. Monday morning):

1. Open `00_Meta/stale-sandbox.base` — surfaces all `90_Inbox/claude/*` drafts older than 7 days.
2. For each: promote, rewrite, or archive. Don't leave stale drafts to rot.
3. Check Tag Wrangler for new tag drift; merge or rename as needed.

---

## Folder rules

- `20_Notes/` stays **flat**. No sub-folders. Sub-folders fragment the Zettelkasten and make grep less effective.
- `10_Sources/` has the three sub-folders shipped (`papers/`, `web/`, `code/`); add no more.
- `99_Archive/` is for things you want to keep but don't want surfacing in everyday searches. Move with intent.
- Claude never writes outside `90_Inbox/claude/` except for the narrow `related:` carve-out.

---

## Anti-patterns (these are real failure modes — read once)

- **Summary as primary record.** Claude writes a summary of a source; you delete the original; later you can't re-interpret. Always keep verbatim capture.
- **Deep tag trees.** Pretty in the tag pane, painful in Bases filters and grep.
- **Sub-folders in `20_Notes/`.** Same problem at folder level.
- **Mega-MOCs.** When a MOC has >30 links, it's a sub-MOC waiting to happen. Split.
- **Daily-notes-as-capture.** Daily notes are for reflection on what you've already captured. New captures go to `90_Inbox/`.
- **Dashboard porn.** `.base` views that look impressive but you never open. If you don't open it weekly, delete it.
- **Pre-built taxonomy.** A `tags.md` file with the "official tag list" ages badly. Let it emerge.
