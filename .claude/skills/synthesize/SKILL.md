---
name: synthesize
description: Synthesize what the vault says about a topic across multiple notes. Use when the user asks for a synthesis, summary, or roundup that spans more than one note — phrases like "what do my notes say about X", "synthesize my thinking on X", "pull together what I have on X", "summarize across my notes on X", "give me a roundup of X". Writes a new atomic synthesis note to the sandbox, citing every claim. Auto-trigger on these phrases.
---

# Synthesize across notes

## When this fires

User wants Claude to pull together claims across multiple existing notes into a single new synthesis note. This is NOT for summarizing a single source — that's `ingest-url` / `ingest-pdf` territory.

## Steps

1. **Delegate inventory to vault-librarian first**:
   - Use the Agent tool with `subagent_type=vault-librarian`.
   - Ask: "What notes exist on <topic>? Are there existing syntheses? Any duplicates I should know about?"
   - This protects against re-synthesizing what already exists.

2. **If a recent synthesis exists** (<90 days old) on the same topic, surface it and ask the user:
   - Extend the existing synthesis, or
   - Draft a fresh one (with reason), or
   - Skip.

3. **Search the vault** for the topic + 2–4 obvious synonyms:
   - Ripgrep `20_Notes/`, `10_Sources/`, `30_MOCs/` for keywords.
   - Read top ~15 hits in full.
   - Follow `[[wikilinks]]` one hop from each relevant hit.

4. **Filename**: `90_Inbox/claude/synthesis--<topic-slug>--YYYY-MM-DD.md`

5. **Write the synthesis**:

```markdown
---
type: note
status: inbox
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [<topic tags>]
author: claude
confidence: medium
---

# Synthesis: <Topic>

## Scope

<1-2 sentences: what this synthesis covers and what it deliberately excludes>

## Claims

- **<Claim 1>**: <expanded explanation in 1-3 sentences>. Sources: [[<source-note-1>]], [[<source-note-2>]].
- **<Claim 2>**: <explanation>. Sources: [[<source>]].
- <continue — every claim must cite ≥1 source>

## Contradictions

- [[<source-A>]] argues <X> while [[<source-B>]] argues <Y>. The disagreement is about <nature>.
- <If no contradictions found, write: "No direct contradictions found among consulted sources.">

## Open Questions

- <question not resolved by current sources>
- <gap in the evidence base>

## Sources Consulted

- [[<every source note used>]]
- [[<...>]]
```

6. **Cite discipline**:
   - EVERY claim has ≥1 `[[source-note]]` citation.
   - When exact wording matters, quote the source note verbatim.
   - Contradictions are surfaced explicitly. Never paper them over.

7. **Confidence**:
   - Default: `medium`.
   - `low` if sources are thin (<3) or conflict heavily.
   - `high` only after the user explicitly verifies — never self-assigned.

8. **Reverse-link enrichment** (`related:` frontmatter carve-out).

   For EVERY source note cited in this synthesis, append `[[synthesis--<topic>--<date>]]` to that source's `related:` frontmatter. Citation is itself a strong signal — single-tag-match is enough to trigger auto-apply for this skill.

   **Operation rules (absolute)**:
   - Append-only to `related:`. Never reorder, never remove.
   - Skip if already present (idempotent).
   - Bump `updated:` to today on the modified source note.
   - Touch ONLY the `related:` field. No other frontmatter changes. NEVER touch source-note BODY content.

9. **Report** the path, claim count, contradiction count, source notes consulted, and the list of source notes whose `related:` was enriched.

10. **STOP.** Source-note BODY content is NEVER modified. Do NOT merge atomic notes. The synthesis is a separate, additive note. The `related:` carve-out is your only exception.

## Rules (non-negotiable)

- Never make a claim without a citation.
- Never edit body content of source notes during synthesis.
- Never merge or consolidate atomic notes.
- Contradictions surface explicitly with both sides cited.
- Default confidence is `medium`. Don't self-promote.
- If the vault is too sparse to synthesize meaningfully (<3 source notes on the topic), say so and stop — don't pad.
- Reverse-link enrichment is auto-applied to every cited source. Frontmatter `related:` only — never body content, never other fields.
