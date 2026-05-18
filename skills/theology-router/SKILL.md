---
name: theology-router
description: Activate when the user asks about Orthodox theology, patristics, Church Fathers, Scripture commentary, sacred tradition, councils, ascetic teaching, dogmatics, hesychasm, Palamism, Christology, Trinity, energies vs essence, or any topic where citing actual patristic sources matters. Routes search to the `teo-search` subagent (search-only, returns candidate citations) and then quotes via `mcp__patristic__read_passage` in the main loop.
---

This is a theological / patristic question. Follow this two-step pattern:

1. **Delegate the search to subagent `teo-search`.** Use the standard Task
   tool. **Reformulate the user's question into Russian before passing
   it** — the corpus is Russian (azbyka.ru translations), Postgres
   tsvector uses a Russian stemmer, and bge-m3 cross-lingual similarity
   is weaker than same-language. Translate the *question*, but keep
   Greek / Slavonic technical terms (ὁμοούσιος, ипостась, энергия,
   обожение, исихия) verbatim.

   It returns 3–8 candidates: `[{citation, snippet, author, work, relevance_hint}]`
   — these will be in Russian (source language). It does NOT include
   full passage text, only slugs and short snippets.

2. **Read the passages you want to quote, in the main loop.** For each
   citation you plan to use in the answer, call the MCP tool
   `mcp__patristic__read_passage` directly with the citation slug. It
   returns `{text, author, work_title, chapter_num, source_url, ...}`
   — the verbatim full paragraph.

3. **Quote only from `read_passage.text`.** Never quote from the `snippet`
   field of `teo-search`'s output — snippets are truncated and may have
   been reformulated by the search subagent. If a quoted phrase doesn't
   appear verbatim in `read_passage.text`, that's a hallucination.

4. **Citation slug is a TOOL ARGUMENT, not a label for the user.** The
   long form `author_slug/work_slug/NNNN/pX[-Y]` exists so `read_passage`
   can find the paragraph — copy it verbatim into the tool call, never
   simplify it. **But do NOT show this raw slug to the user as the
   attribution line.** It's unreadable. Build the attribution from the
   human-readable fields that `read_passage` returns:

   - `author` — display name (e.g. «Прп. Исаак Сирин Ниневийский»)
   - `work_title` — display title (e.g. «Избранник»)
   - `chapter_num` + optional `chapter_title`
   - `source_url` — the azbyka.ru link to the original page

   ✅ GOOD (human-readable attribution):

   > «Милосердие и правосудие в одной душе — то же, что человек,
   > который в одном доме поклоняется Богу и идолам…»
   >
   > — Прп. Исаак Сирин, *Избранник*, гл. 54 ([azbyka.ru](https://azbyka.ru/otechnik/Isaak_Sirin/izbornik/54))

   ❌ BAD (raw slug dumped on the user):

   > «Милосердие и правосудие в одной душе…»
   > (isaak_sirin_ninevijskij_prepodobnyj/isaak_sirin_ninevijskij_prepodobnyj_izbornik/0054/p2)

   If `chapter_title` is non-empty, include it: `гл. 54, «О милосердии»`.
   The `source_url` is the user's path back to the original — always
   include it as a clickable link.

5. **Negative results**: if `teo-search` returns `[]`, say so honestly
   ("в корпусе ничего не нашлось по этой теме") rather than answering
   from general knowledge.

6. **Reply in the user's language.** Search runs in Russian, but the
   final answer to the user goes in whatever language they asked
   (English, Russian, etc.). Patristic quotations stay in their
   original Russian form — if the user asked in English, give the
   Russian quote AND a short English gloss / translation after it. Do
   not silently translate the quote itself, the verbatim Russian is
   the citation's value.
