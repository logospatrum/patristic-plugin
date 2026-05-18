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

4. **Citation format**: `author_slug/work_slug/NNNN/pX[-Y]` exactly as
   returned by the tool. Don't simplify, don't kebab-case, don't drop
   redundant prefixes — the long form is correct.

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
