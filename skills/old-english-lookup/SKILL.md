---
name: old-english-lookup
description: >-
  Look up Old English (Anglo-Saxon) words, inflections, and Bēowulf citations through the
  eldertongue.com MCP server. Use when a task involves Old English vocabulary, translation
  between Modern English and Old English, morphological analysis of a declined or conjugated
  form, or finding where a word occurs in Bēowulf — and you need structured, citable results
  rather than a guessed answer.
---

# Old English lookup

This skill wraps the five read-only tools exposed by the **eldertongue** MCP server
(`https://eldertongue.com/mcp`). The data comes from Wiktionary and covers about 14,000 Old
English headwords, reverse Modern English → Old English lookup, and a full Bēowulf verse index.
Everything is read-only and needs no authentication.

Prefer these tools over answering from memory: Old English spelling and inflection are irregular,
and the tools return canonical page URLs you can cite.

## When to use which tool

| Task | Tool |
| --- | --- |
| Define an Old English headword | `lookup_headword` |
| Get the full declension / conjugation table | `inflect` |
| Explain a declined/conjugated form you were given | `identify_form` |
| Find the Old English word for a Modern English concept | `english_to_old_english` |
| Find a word's attestations in Bēowulf | `beowulf_occurrences` |

If a lookup misses: retry the headword **without macrons** (`cȳning` → `cyning`), and if you only
have an inflected surface form, use `identify_form` on that form instead of `lookup_headword`.

## `lookup_headword`

Input: `{ "word": string }` — an Old English headword, macrons optional.

Use it to get `partOfSpeech`, `genders`, `ipa`, `syllables`, `declension` / `verbClass`,
`etymology`, and `senses[]`. The response includes `found`, and `pageUrl` /
`wiktionaryUrl` for citation.

Example call:

```json
{ "word": "cyning" }
```

Key fields to surface: the first `senses[].definition` (e.g. "king"), `partOfSpeech`, and
`pageUrl` (`https://eldertongue.com/word/cyning`).

## `inflect`

Input: `{ "word": string }` — the lemma.

Returns a full paradigm: `paradigmKind` (e.g. `"strong a-stem"`), `tables[]` →
`groups[]` → `rows[]` → `cells[]`, plus a flat `cells[]` list. Each form carries an anchor URL
like `https://eldertongue.com/word/cyning#gen-sg`. When a user asks "what is the genitive plural
of X", read the matching row/cell rather than declining it yourself.

## `identify_form`

Input: `{ "form": string }` — any inflected form, e.g. `"cyninges"`.

Returns `analyses[]`; each has `lemma`, `lemmaPageUrl`, `cellId`, `tag`, a `grammar` object
(`case`, `number`, `gender`, `person`, `tense`, `mood`, `degree`, …), and `formUrl`. There can be
several candidate analyses — report all plausible ones and let the grammar disambiguate.

## `english_to_old_english`

Input: `{ "word": string }` — a Modern English word.

Returns ranked `candidates[]`, each with `lemma`, `display`, `partOfSpeech`, `score`, `gloss`,
and `pageUrl`, plus the reverse-lookup `pageUrl` (`https://eldertongue.com/english/<word>`).
Present candidates in order; the top result is the strongest match, but show a couple so the
user can choose by context (e.g. `king` → `cyning`, `Friscyning`, `leodcyning`, …).

## `beowulf_occurrences`

Input: `{ "word": string }`.

Returns `times`, `lines`, a per-spelling `forms[]` breakdown, and `occurrences[]` — each with
`line`, `text`, `forms`, and a verse `url` of the form `https://eldertongue.com/beowulf#l<line>`.
`occurrencesTotal` is the full count; `occurrences[]` may be capped, so quote the total from
`occurrencesTotal` when the list is shorter than the count.

## Citing results

Always attach the canonical eldertongue URL to any claim you make from these tools:

- Headword or form: the `pageUrl` / `lemmaPageUrl` / `formUrl`.
- Bēowulf line: the occurrence `url` (`https://eldertongue.com/beowulf#l54`).
- Reverse lookup: the `pageUrl` on the response (`https://eldertongue.com/english/king`).

State the source as *eldertongue.com, based on Wiktionary, CC BY-SA 4.0*, and link the specific
page. If you reuse the definitions in a deliverable, keep the CC BY-SA 4.0 attribution.
