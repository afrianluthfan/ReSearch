---
name: ingest-url
description: Ingest a web URL into the research vault. Use when the user asks to capture, save, file, ingest, or "pull in" a webpage / article / blog post / docs page, or when they paste a URL with apparent intent to file it (e.g. "ingest https://...", "save this article", "capture this blog post", "file this link", "add this to my notes"). Drafts a source note to the sandbox with verbatim quotes captured BEFORE summary. Auto-trigger on these phrases.
---

# Ingest URL

## When this fires

User wants a web source captured into the vault. They've either pasted a URL or named one.

## Steps

1. **Fetch** the URL with WebFetch. Extract:
   - Title
   - Author(s)
   - Publication date
   - Canonical URL (use the page's canonical link if different from what user pasted)

2. **Check for duplicates** before drafting:
   - Ripgrep `10_Sources/web/` for the URL.
   - If a matching `source_url` already exists, surface it and ask the user before proceeding.

3. **Slugify** the title (kebab-case, max 60 chars). Filename:
   `90_Inbox/claude/YYYY-MM-DD-<slug>.md`

4. **Search for existing tags** before coining new ones:
   - `rg -h "^tags:" "/Users/bankraya/Documents/Obsidian Vault/Personal/ReSearch/20_Notes/" 2>/dev/null | sort -u`
   - Reuse existing tags where they fit. Coin new only when nothing matches.

5. **Write the note** using this exact template:

```markdown
---
type: source
source_type: web
status: inbox
source_url: <canonical-url>
fetched: YYYY-MM-DD
authors: [<author1>]
year: <year-if-known>
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [<2-4 kebab-case>]
author: claude
---

# <Title>

## Capture

> "<verbatim quote 1>"

> "<verbatim quote 2>"

> "<verbatim quote 3-8 varying lengths — the substance of the piece in the author's own words>"

## Summary

<1-3 paragraphs in your own words, synthesizing the piece>

## Why It Matters

<1-2 sentences on relevance to ongoing research threads, if any apparent>

## Related

- [[<existing note found via tag/topic match, if any>]]
```

6. **Reverse-link enrichment** (`related:` frontmatter carve-out — the ONLY exception to the sandbox rule).

   For each older note found during the tag/lexical search that hits the high-confidence threshold, append `[[<this-new-note-slug>]]` to its `related:` frontmatter.

   **Auto-apply triggers** (any one):
   - ≥2 shared tags between the new note and the older note, OR
   - ≥1 shared tag PLUS lexical match on ≥2 key nouns, OR
   - User explicitly requested the link.

   **Operation rules (absolute)**:
   - Append-only to `related:`. Never reorder, never remove, never replace.
   - Skip if `[[<this-new-note-slug>]]` is already present (idempotent).
   - Bump `updated:` to today's date on the modified old note.
   - Touch ONLY the `related:` field of the old note. No other frontmatter changes. No body modification.
   - If only ONE signal is present (medium confidence), surface in chat and ask before applying — do NOT auto-apply.
   - If `related:` does not exist on the old note, add it as a new top-level frontmatter key with `[[<this-new-note-slug>]]` as the only entry.

7. **Report** the sandbox path, forward links inserted in the new note, and the list of old notes whose `related:` was enriched.

8. **STOP.** Body content of existing notes is NEVER edited. The `related:` carve-out is your only exception to the sandbox rule. Don't promote. Don't modify any other frontmatter field.

## Rules (non-negotiable)

- Verbatim quotes go BEFORE the summary. Always. Summary-as-primary-record is a banned anti-pattern.
- 3–8 quoted passages, varying lengths. Aim for the substantive content, not boilerplate.
- Tags are kebab-case, single-level. Never `methods/bayesian/mcmc`-style nesting.
- `confidence` field is omitted for sources (only used for synthesis notes).
- Use today's date for `created`, `updated`, and `fetched`.
- Reverse-link enrichment touches ONLY the `related:` frontmatter of high-confidence matches. Never body content. Never other frontmatter fields.
