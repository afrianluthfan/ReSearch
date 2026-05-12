# Upgrade triggers

Don't add infrastructure speculatively. This file is the gate: review it when tempted, and only proceed if at least one box is ticked.

---

## Vector DB / semantic search (via MCP server)

Add when ANY trigger fires:

- [ ] Ripgrep for a typical concept returns **>50 hits** AND I can't pick the 5 to read first.
- [ ] I asked Claude the same synthesis question **twice in a month** because I forgot the earlier answer existed.
- [ ] Corpus crosses **~800 notes** OR **~2M tokens** of markdown (whichever first).
- [ ] I search for ideas using vocabulary not present in my notes (semantic gap) **more than once a week**.

**Implementation when triggered:** stand up an MCP server (e.g., a local Qdrant or LanceDB instance) that exposes vault content as a semantic-search tool. The vault's markdown content does NOT change — only retrieval gets a new layer.

---

## SQLite (cross-vault aggregation)

Add when ALL hold:

- [ ] Bases can't aggregate the cross-vault, time-series questions I'm actually asking.
- [ ] The same query is run >5 times in a month and Bases is genuinely insufficient.
- [ ] (Almost never necessary. Most users don't reach this.)

---

## Custom MCP server (Zotero, Notion, etc.)

Add when ALL hold:

- [ ] I'm actively using the external service (e.g., Zotero) as part of my research workflow.
- [ ] Manual data shuttling between the service and the vault costs >30 min/week.
- [ ] An off-the-shelf MCP for that service exists and is maintained.

---

## Do NOT upgrade for

- "It would be cool."
- "I read about RAG."
- "Someone on r/Obsidian recommended it."
- Note count alone, without a recall problem.
- Aesthetic appeal of dashboards.

---

## When you do upgrade

Document the trigger that fired, the date, and the configuration chosen — in this file, as a new section at the bottom. Future-you will want to know why.

---

## Review log

- **2026-05-12** — Vault initialised. No triggers fired. Plan to revisit at 200 notes or 3 months, whichever first.
