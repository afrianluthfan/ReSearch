---
name: ingest-pdf
description: Ingest a PDF (research paper, book chapter, white paper) into the vault. Use when the user references a PDF path, attaches a paper, or asks to capture / file / ingest / read / summarize a paper. Phrases that trigger this: "ingest this paper", "read this PDF", "summarize this paper", "file this paper into my vault", or any message that names a .pdf file or arXiv/DOI link. Drafts a paper source note to the sandbox with page-referenced key claims.
---

# Ingest PDF

## When this fires

User has a PDF they want captured as a structured source note. The PDF may be local (a file path) or remote (arXiv, DOI, direct URL).

## Steps

1. **Read the PDF** with the Read tool.
   - For papers >10 pages, start with pages 1–10 (title page, abstract, intro, key results, conclusion).
   - Request more pages only when needed to verify a claim or extract methods detail.
   - The Read tool's `pages` parameter is required for PDFs >10 pages.

2. **Extract metadata**:
   - Title (canonical, from the title page)
   - Authors (full list, in order)
   - Year (publication year)
   - DOI or arXiv ID
   - Venue (journal, conference, preprint server)
   - If any of these are missing from the PDF, WebSearch for the citation before proceeding.

3. **Confirm with user** before moving/copying the PDF into `attachments/`. Default: leave the PDF where it is and record its absolute path in `pdf_path`.

4. **Check for duplicates**:
   - Ripgrep `10_Sources/papers/` for the DOI/arXiv ID.
   - Ripgrep for first-author surname + year as a fallback.

5. **Filename**: `90_Inbox/claude/YYYY-MM-DD-<lastname>-<year>-<slug>.md`
   Example: `2026-05-12-smith-2024-bayesian-deep-learning.md`

6. **Write the note**:

```markdown
---
type: source
source_type: paper
status: inbox
source_url: <DOI URL or arXiv URL>
fetched: YYYY-MM-DD
authors: [<author1>, <author2>]
year: <year>
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [<2-4 kebab-case>]
author: claude
pdf_path: <absolute path>
---

# <Full Title>

## Citation

<formatted citation — APA-style by default unless user specifies>

## Abstract (verbatim)

> <the abstract, copied exactly, in a blockquote>

## Key Claims

- <Claim 1> (p. X)
- <Claim 2> (pp. Y–Z)
- <3-7 bullets total, each with a page reference — non-negotiable>

## Methods

<2-4 sentences: study design, sample / dataset, key techniques, comparisons>

## Evidence Quality

<honest assessment: sample size, replication status, known critiques, conflicts of interest if disclosed, methodological caveats>

## My Notes

<leave EMPTY — this is the user's annotation space>

## Open Questions

- <question the paper raises but doesn't answer>

## Related

- [[<related source or note, if any>]]
```

7. **Reverse-link enrichment** (`related:` frontmatter carve-out — the ONLY exception to the sandbox rule).

   For each older note relevant to this paper (found via tag/lexical search, citation in the paper, or author match) that hits the high-confidence threshold, append `[[<this-new-note-slug>]]` to its `related:` frontmatter.

   **Auto-apply triggers** (any one):
   - ≥2 shared tags between the paper note and the older note, OR
   - ≥1 shared tag PLUS lexical match on ≥2 key nouns, OR
   - Same author appears in the older note's `authors:` list, OR
   - User explicitly requested the link.

   **Operation rules (absolute)**:
   - Append-only to `related:`. Never reorder, never remove.
   - Skip if already present (idempotent).
   - Bump `updated:` to today on the modified old note.
   - Touch ONLY the `related:` field. No other frontmatter changes. No body modification.
   - Medium confidence (single signal) → ask before applying.

8. **Report** the sandbox path, key-claim count, page references, methodological flags, and the list of old notes whose `related:` was enriched.

9. **STOP.** Don't promote. Don't move PDFs without explicit user permission. Don't fill `## My Notes`. Don't edit body content of any existing note.

## Rules (non-negotiable)

- Every key claim has a page reference. No exceptions.
- Abstract is verbatim, in a blockquote.
- `## My Notes` stays empty — that's the user's space, not yours.
- Evidence quality is honest. Flag p-hacking, small N, replication failures, conflicts of interest.
- If the paper is paywalled and you can't read past the abstract, say so and stop — don't fabricate claims.
- Reverse-link enrichment touches ONLY the `related:` frontmatter of high-confidence matches.
