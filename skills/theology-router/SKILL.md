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

4. **Citation slug NEVER appears in the user-visible output.** The slug
   (`author_slug/work_slug/NNNN/pX[-Y]`) is a tool argument and nothing
   else. Copy it verbatim into `read_passage` calls; never paste it into
   your reply, not in parentheses, not as a "source line", not anywhere.
   This includes Bible slugs like `bible_sobornoe_poslanie_…/0002/p12` —
   the user wants «James 2:12–13», not the internal slug.

   Build the attribution line from the human-readable fields
   `read_passage` returns:

   - `author` — display name
   - `work_title` — display title
   - `chapter_num` and `chapter_title` if non-empty
   - `source_url` — the azbyka.ru link

   For Bible passages, format the attribution as the usual scriptural
   reference (e.g. «Иак. 2:12–13», «Jas. 2:12–13», «James 2:12–13»),
   *not* the slug.

5. **The entire reply — including quotations — goes in the user's language.**

   Whatever language the user asked in, that's the language of: your
   prose, the patristic quotation itself, the author/work attribution,
   and the chapter label. The Russian source text from `read_passage`
   is for you, the assistant, to read and translate — it is not the
   user's final view.

   If the user asked in Russian → keep everything in Russian (the
   corpus is already in Russian, no translation needed).

   If the user asked in English (or any other non-Russian language) →
   translate the quotation into idiomatic theological English. Mark
   the translation honestly as a working translation when accuracy
   matters (e.g. `(working translation)` after the first long quote).
   Render the author's name in the user's language conventions ("St.
   Isaac the Syrian", not «Прп. Исаак Сирин»). Always include the
   `source_url` as a clickable link — the reader who wants the
   verbatim Russian can click through to azbyka.ru.

   ✅ GOOD (English question, English reply, translated quote, NO slug):

   > "Mercy and justice in one soul are like a man who in one house
   > worships God and idols. Mercy is opposed to justice…"
   > *(working translation)*
   >
   > — St. Isaac the Syrian, *Ascetical Homilies*, ch. 54 ([azbyka.ru](https://azbyka.ru/otechnik/Isaak_Sirin/izbornik/54))

   ❌ BAD (English question, but quote left in Russian + raw slug):

   > «Милосердие и правосудие в одной душе…»
   > (isaak_sirin_ninevijskij_prepodobnyj/.../0054/p2)

6. **Negative results**: if `teo-search` returns `[]`, say so honestly
   in the user's language ("nothing in the corpus on this topic" /
   "в корпусе ничего не нашлось по этой теме") rather than answering
   from general knowledge.
