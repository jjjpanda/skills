---
name: query-vault
description: Semantic vault search using the embedding-viewer Obsidian plugin. Given query strings, returns a deduplicated list of confirmed relevant vault notes. Use to surface backlink candidates from bare entity names or note content.
---

Semantic search across the Obsidian vault using the embedding-viewer plugin's local HTTP API.

## Requirements

1. Obsidian must be running.
2. The current working directory (`pwd`) must be the root of the Obsidian vault (contains `.obsidian/`).
3. The `embedding-viewer` plugin must be installed and active in the vault.

## Locate the API Port

Read the file `.obsidian/plugins/embedding-viewer/data.json` from the current working directory.
Parse it as JSON and look for the `apiPort` property. If the file doesn't exist or the property is missing, use the default port `27123`.

## Run Queries

For each query string provided by the caller, run an HTTP request (via `curl` or script) to hit the local API:

```bash
# Example using curl (replace <port> and <query_string> appropriately)
curl -X POST http://127.0.0.1:<port>/search \
  -H "Content-Type: application/json" \
  -d '{"query": "<query_string>", "limit": 5}'
```

Parse the JSON output, which takes the form:
`{ "results": [ { "path": "...", "content": "...", "similarity": 0.85 }, ... ] }`

## Filter Results

Discard results where:
- `similarity` < 0.4
- `path` is already read in the calling skill's context

## Read and Confirm

Deduplicate remaining paths across all queries. Read each note and confirm its content is clearly relevant to what the query references. Discard any that aren't a strong match.

## Return

Return the confirmed notes as a candidate list. The calling skill determines how to use them (as context, as wikilink candidates, or both).

## Graceful Degradation

If the API call fails (plugin not running, API disabled, any error), fall back to lexical search: search for files matching `**/*{entity}*.md` from the vault root for each query's apparent named entity. For hits that aren't periodic notes, read and confirm relevance, then include in the candidate list.
