---
name: find-related
description: Find vault notes related to a given note or topic. Use when the user asks what connects to a note, what's related, what other notes touch a subject, or wants link suggestions — phrases like "what's related to this", "what connects to X", "find related notes", "suggest links for this note", "what else do I have on this". Returns a ranked list in chat — does NOT write to disk by default.
---

# Find related notes

## When this fires

User has opened a note (or named a topic) and wants to know what else in the vault relates to it. This is a discovery / linking aid — not a synthesis.

## Steps

1. **Read the seed** (if a note path is given):
   - Extract `tags` and `related` from frontmatter.
   - Collect existing `[[wikilinks]]` (forward links).
   - Identify key nouns / concepts in the body (3–8 of them).

2. **Search the vault**:
   - Ripgrep `20_Notes/`, `10_Sources/`, `30_MOCs/`, `50_Questions/` for the key nouns.
   - Find notes that link TO the seed (backlinks): `rg "[[<seed-filename-base>]]" --type md`.
   - Find notes sharing ≥1 tag with the seed (read frontmatter of candidates).

3. **Rank** each candidate by:
   - Number of shared tags (high signal)
   - Existing link presence (already linked = relevant)
   - Lexical overlap on key nouns (medium signal)
   - Recency (recent notes more likely current)

4. **Return in chat** (no file writes by default). Max 15 notes, grouped:

```
## Closely related (tag overlap + lexical match)
- [[Note A]] — shares tags [x, y]; mentions <concept>
- [[Note B]] — shares tag [z]; already linked from seed (backlink)

## Possibly related (lexical only)
- [[Note C]] — mentions <concept> but no tag overlap
- [[Note D]] — same author / source

## Already linked (FYI)
- [[Note E]] — forward link exists in seed
- [[Note F]] — backlink exists

## Suggested MOC connections (if seed has <2 outgoing links)
- [[<Topic> MOC]] — covers <scope>
```

5. **Apply mode** (`related:` frontmatter carve-out — the ONLY direct-edit exception to the sandbox rule).

   If the user says "apply these" / "add these to related" / "link them" — OR if a candidate hits the auto-apply threshold and the user has approved auto-apply for this session — append `[[<seed>]]` to the candidate's `related:` frontmatter directly.

   **Auto-apply thresholds** (any one, when user opts into auto-apply):
   - ≥2 shared tags between seed and candidate, OR
   - ≥1 shared tag PLUS lexical match on ≥2 key nouns, OR
   - Existing bidirectional link already present and `related:` is just missing the entry.

   **Operation rules (absolute)**:
   - Append-only to `related:`. Never reorder, never remove existing entries.
   - Skip if `[[<seed>]]` is already in `related:` (idempotent).
   - Bump `updated:` to today's date on the modified note.
   - Touch ONLY the `related:` field. No other frontmatter changes. No body content modification.
   - Medium confidence (single signal): list candidates in chat and ask before applying. Don't auto-apply.

   **Direction**: this mode writes `[[<seed>]]` into each CANDIDATE's `related:` (so the candidates point back to the seed). To also enrich the SEED's `related:` with candidate links, the user must explicitly say "add these to <seed>'s related" — that direction is a body-equivalent action on the seed and goes through `proposed--<seed-name>.md` to the sandbox.

## Rules (non-negotiable)

- Chat-only output by default. File writes only when user opts into apply mode.
- Distinguish "actually related" from "shares a stopword."
- If the seed note has fewer than 2 outgoing `[[links]]`, surface 1–2 candidate MOCs to link to.
- Direct edits to old notes are allowed ONLY for `related:` frontmatter appends under apply mode. Body content is NEVER edited in place — that always goes through `proposed--<name>.md`.
