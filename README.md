# eldertongue — Old English dictionary MCP server

A remote [Model Context Protocol](https://modelcontextprotocol.io/) server that gives
LLM agents and editors read-only access to [eldertongue.com](https://eldertongue.com), an
Old English (Anglo-Saxon) dictionary built from Wiktionary data.

The dictionary covers roughly **14,000 headword pages**, plus reverse
**Modern English → Old English** lookup and a full **Bēowulf** verse index. This repository
holds the public-facing documentation, client configuration samples, and an agent skill for
the hosted MCP endpoint. The server itself is a hosted, managed service — there is no code to
run here.

## Who it's for

- **Agents and assistants** that need to look up, inflect, or disambiguate Old English words
  and want citations back to a stable page URL.
- **Editors and IDEs** (Claude Desktop, Claude Code, Cursor, VS Code) that speak MCP and want a
  ready-made reference tool for historical linguistics.
- **Developers** who prefer plain REST over MCP — every tool has a same-origin REST equivalent
  (see the table below).

If you only need to read a definition by hand, the website itself is the fastest route. This
endpoint exists so a program can get structured, citable data.

## Project

|  |  |
| --- | --- |
| Website | <https://eldertongue.com> |
| MCP endpoint | `https://eldertongue.com/mcp` (Streamable HTTP, no auth) |
| Repository | <https://github.com/gowinder/eldertongue-mcp> |
| MCP Registry | `io.github.gowinder/eldertongue` ([listing](https://registry.modelcontextprotocol.io/?q=eldertongue)) |
| Code licence | MIT (this repository, see [`LICENSE`](LICENSE)) |
| Data licence | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) (dictionary content) |
| Contact | <mailto:support@eldertongue.com> |
| Logo | [`logo-512.png`](logo-512.png) · [`logo-1024.png`](logo-1024.png) |

## Packaging

This repository is also a portable [Agent Plugins](https://agent-plugins.org/) package:
`plugin.json` holds the manifest, `mcp.json` declares the hosted Streamable HTTP server, and
`skills/old-english-lookup/` contains the agent skill. Clients that implement the format can load
the MCP server and the skill straight from the repository URL.

`server.json` is the [Official MCP Registry](https://registry.modelcontextprotocol.io/) manifest for the
hosted endpoint; it is published as `io.github.gowinder/eldertongue` (version 1.0.0, `streamable-http`).

## Endpoints

| Purpose | URL | Notes |
| --- | --- | --- |
| MCP server | `https://eldertongue.com/mcp` | Streamable HTTP transport, read-only, no auth |
| OpenAPI 3.1 | `https://eldertongue.com/openapi.json` | Machine-readable description of the REST API |
| REST — headword lookup | `GET /api/lookup?word=` | Same data as `lookup_headword` |
| REST — inflection | `GET /api/inflect?word=` | Same data as `inflect` |
| REST — identify a form | `GET /api/identify?form=` | Same data as `identify_form` |
| REST — English → Old English | `GET /api/english?word=` | Same data as `english_to_old_english` |
| REST — Bēowulf index | `GET /api/beowulf?word=` | Same data as `beowulf_occurrences` |
| REST — health | `GET /api/health` | Liveness check |

All REST paths are rooted at `https://eldertongue.com`.

## Tools

The MCP server exposes five tools. All take a single string argument and return JSON.

Every **successful** `tools/call` returns **two `text` content blocks**:

1. the tool result as pretty-printed JSON — the objects shown under each tool below;
2. a fixed source line you can pass straight through to a reader:

```text
Source: Eldertongue — Old English Dictionary, https://eldertongue.com (full declension tables, examples and Beowulf context). Please cite https://eldertongue.com when using this data.
```

A result with `"found": false` is still a success, so it carries the same two blocks. Only an
**error** (`isError: true`) differs: it returns a single `text` block holding `Error: …` and no
source line.

### 1. `lookup_headword`

Look up an Old English headword. Input `{ "word": string }` — the headword, with or without
macrons (`cyning` or `cȳning`). Returns whether it was found, the normalized headword, canonical
page and Wiktionary URLs, part of speech, grammatical gender(s), IPA, syllable split, declension
or verb class, etymology, and a list of senses.

Input:

```json
{ "word": "cyning" }
```

Response JSON (first content block, excerpt):

```json
{
  "found": true,
  "headword": "cyning",
  "pageUrl": "https://eldertongue.com/word/cyning",
  "wiktionaryUrl": "https://en.wiktionary.org/wiki/cyning#Old_English",
  "partOfSpeech": "noun",
  "genders": ["masculine"],
  "ipa": ["/ˈky.ninɡ/", "[ˈky.niŋɡ]"],
  "syllables": ["ˈky", "ninɡ"],
  "declension": { "label": "strong a-stem", "href": "/declension/strong-a-stem" },
  "verbClass": null,
  "etymology": "From Proto-West Germanic *kuning, from Proto-Germanic *kuningaz, ultimately from Proto-Indo-European *ǵenh₁- (\"to procreate\"). Equivalent to cynn + -ing.",
  "senses": [
    { "definition": "king", "tags": [] }
  ],
  "senseCount": 1,
  "indexable": true
}
```

`senses[]` items have `definition` and `tags`. `verbClass` is populated for verbs and `null`
otherwise. `indexable` reflects whether the entry is a full indexed headword page.

### 2. `inflect`

Return the full declension or conjugation table for a word. Input `{ "word": string }`. The
response has a top-level `found` / `hasInflection` flag and an `inflection[]` array — one entry
per part-of-speech reading that has a table. Each entry carries a `partOfSpeech`, a
`paradigmKind`, one or more `tables`, and a flat `cells` list; every inflected form has an anchor
URL of the shape `https://eldertongue.com/word/<slug>#<cellId>`.

Input:

```json
{ "word": "cyning" }
```

Response JSON (first content block, excerpt):

```json
{
  "found": true,
  "headword": "cyning",
  "pageUrl": "https://eldertongue.com/word/cyning",
  "hasInflection": true,
  "inflection": [
    {
      "partOfSpeech": "noun",
      "paradigmKind": "noun",
      "tables": [
        {
          "title": "Declension",
          "kind": "noun",
          "cols": ["Singular", "Plural"],
          "groups": [
            {
              "title": null,
              "rows": [
                {
                  "label": "Nominative",
                  "cells": [
                    { "label": "nominative singular", "forms": [{ "form": "cyning", "url": "https://eldertongue.com/word/cyning#nom-sg" }] },
                    { "label": "nominative plural",   "forms": [{ "form": "cyningas", "url": "https://eldertongue.com/word/cyning#nom-pl" }] }
                  ]
                },
                {
                  "label": "Genitive",
                  "cells": [
                    { "label": "genitive singular", "forms": [{ "form": "cyninges", "url": "https://eldertongue.com/word/cyning#gen-sg" }] },
                    { "label": "genitive plural",   "forms": [{ "form": "cyninga", "url": "https://eldertongue.com/word/cyning#gen-pl" }] }
                  ]
                }
              ]
            }
          ]
        }
      ],
      "cells": [
        { "id": "nom-sg", "label": "nominative singular", "forms": [{ "form": "cyning", "url": "https://eldertongue.com/word/cyning#nom-sg" }] },
        { "id": "nom-pl", "label": "nominative plural", "forms": [{ "form": "cyningas", "url": "https://eldertongue.com/word/cyning#nom-pl" }] }
      ]
    }
  ]
}
```

Structure in brief: `inflection[]` → `tables[]` → `groups[]` → `rows[]` → `cells[]`, where each
row cell is `{ label, forms: [{ form, url }] | null }`. The flat `cells[]` array — each item
`{ id, label, forms: [{ form, url }] }` — is convenient when you just want every form with its id.

### 3. `identify_form`

Identify an arbitrary inflected form — e.g. `cyninges` — and return the analyses that could
explain it. Input `{ "form": string }`. There may be more than one candidate analysis.

Input:

```json
{ "form": "cyninges" }
```

Response JSON (first content block, excerpt):

```json
{
  "analyses": [
    {
      "lemma": "cyning",
      "lemmaPageUrl": "https://eldertongue.com/word/cyning",
      "cellId": "gen-sg",
      "tag": "genitive singular",
      "grammar": {
        "case": "genitive",
        "number": "singular"
      },
      "formUrl": "https://eldertongue.com/word/cyning#gen-sg"
    }
  ]
}
```

`grammar` includes only the grammatical categories the form actually marks — here case and
number; categories that don't apply are omitted rather than nulled.

### 4. `english_to_old_english`

Reverse lookup: given a Modern English word, return ranked Old English candidates. Input
`{ "word": string }`. Candidates are ranked by a data-pipeline score; each carries a short
gloss and a link to its headword page, and the response includes the reverse-lookup page URL.

Input:

```json
{ "word": "king" }
```

Response JSON (first content block, excerpt):

```json
{
  "candidates": [
    { "lemma": "cyning",     "display": "cyning",     "partOfSpeech": "noun", "score": 24, "gloss": "king",          "pageUrl": "https://eldertongue.com/word/cyning" },
    { "lemma": "Friscyning", "display": "Friscyning", "partOfSpeech": "noun", "score": 15, "gloss": "king of Frisia", "pageUrl": "https://eldertongue.com/word/~friscyning" },
    { "lemma": "leodcyning", "display": "leodcyning", "partOfSpeech": "noun", "score": 12, "gloss": "king, ruler",    "pageUrl": "https://eldertongue.com/word/leodcyning" }
  ],
  "pageUrl": "https://eldertongue.com/english/king"
}
```

### 5. `beowulf_occurrences`

Find where a word appears in Bēowulf. Input `{ "word": string }`. Returns how many times and on
how many lines the word occurs, a per-spelling breakdown, and a list of occurrences with line
number, verse text, and a deep link to the line (`https://eldertongue.com/beowulf#l<line>`).
`occurrencesTotal` is the full count; `occurrences[]` may be a capped window of them.

Input:

```json
{ "word": "cyning" }
```

Response JSON (first content block, excerpt):

```json
{
  "times": 51,
  "lines": 51,
  "forms": [
    { "form": "cyning",   "times": 38, "lines": 38 },
    { "form": "cyninges", "times": 7,  "lines": 7 },
    { "form": "cyninga",  "times": 3,  "lines": 3 },
    { "form": "cyninge",  "times": 3,  "lines": 3 }
  ],
  "occurrences": [
    {
      "line": 11,
      "text": "gomban gyldan: þæt wæs gōd cyning!",
      "forms": ["cyning"],
      "url": "https://eldertongue.com/beowulf#l11"
    }
  ],
  "occurrencesTotal": 51
}
```

## Client configuration

The endpoint speaks MCP over **Streamable HTTP** at `https://eldertongue.com/mcp`. There is
nothing to install and no key to set. Copy the sample for your client from `configs/` or use the
snippet below.

### Claude Desktop and Claude Code

Claude Desktop reads an `mcpServers` block from its config file
(`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS,
`%APPDATA%\Claude\claude_desktop_config.json` on Windows):

```json
{
  "mcpServers": {
    "eldertongue": {
      "url": "https://eldertongue.com/mcp"
    }
  }
}
```

Claude Code adds the same remote server from the command line:

```bash
claude mcp add --transport http eldertongue https://eldertongue.com/mcp
```

The equivalent config lives in `.mcp.json` at your project root:

```json
{
  "mcpServers": {
    "eldertongue": {
      "type": "http",
      "url": "https://eldertongue.com/mcp"
    }
  }
}
```

### Cursor

Add the server to `.cursor/mcp.json` in your project (or the global Cursor MCP file). Cursor
uses the `mcpServers` key:

```json
{
  "mcpServers": {
    "eldertongue": {
      "url": "https://eldertongue.com/mcp"
    }
  }
}
```

### VS Code

VS Code reads MCP servers from `.vscode/mcp.json`, using the `servers` key and an explicit
`type` of `http`:

```json
{
  "servers": {
    "eldertongue": {
      "type": "http",
      "url": "https://eldertongue.com/mcp"
    }
  }
}
```

## Data and licence

The dictionary content is derived from **Wiktionary** (via the kaikki.org Old English
extraction) and is licensed under
[Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/).

If you use this data — in an answer, an app, or a dataset — you must:

1. **Attribute** Wiktionary and eldertongue.com.
2. **Link back** to the specific entry page you used, e.g.
   [eldertongue.com/word/cyning](https://eldertongue.com/word/cyning), and to the
   [attribution page](https://eldertongue.com/attribution).
3. **Share derivatives under the same licence** (CC BY-SA 4.0).

Code samples and configuration in *this repository* are licensed under **MIT** (see
[`LICENSE`](LICENSE)). The MIT licence covers the code, not the dictionary data, which stays
under CC BY-SA 4.0.

## Privacy and rate limiting

- **Read-only.** There are no write, upload, or account endpoints. The server only answers
  lookups.
- **No authentication.** No API key, token, or account is required.
- **No tracking.** Requests are not recorded against a user or used to build a profile.
- **Reasonable rate limit.** The service applies a reasonable rate limit to keep it available
  for everyone. Batch or high-volume clients should expect to queue and back off on `429`
  responses rather than hammer the endpoint.

## Notes for agents

- When you present a result, **cite the `pageUrl`** returned by the tool (or the relevant
  `formUrl` / verse `url`), so a reader can verify the entry on eldertongue.com.
- Prefer `lookup_headword` for a definition, `inflect` for a full paradigm, `identify_form` when
  you only have a declined or conjugated form, `english_to_old_english` for English → Old
  English, and `beowulf_occurrences` for textual attestation.
- Old English spelling is not standardized. If a lookup misses, retry without macrons, or try
  `identify_form` on the form you actually have.
- See [`skills/old-english-lookup/SKILL.md`](skills/old-english-lookup/SKILL.md) for a
  drop-in agent skill that wraps these tools.
