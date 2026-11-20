---
name: rebuild-vault-index
description: Update the semantic embeddings DB for a given vault by triggering a rebuild via the embedding-viewer plugin's local HTTP API.
---

Rebuild the vault's semantic embeddings DB using the embedding-viewer plugin.

## Step 1: Confirm Requirements

- Obsidian must be running.
- The current working directory (`pwd`) must be the root of the Obsidian vault (contains `.obsidian/`).
- The `embedding-viewer` plugin must be installed and active in the vault.

## Step 2: Locate the API Port

Read the file `.obsidian/plugins/embedding-viewer/data.json` from the current working directory.
Parse it as JSON and look for the `apiPort` property. If the file doesn't exist or the property is missing, use the default port `27123`.

## Step 3: Run Rebuild Command

Run an HTTP request (via `curl` or script) to trigger the rebuild via the local API:

```bash
# To trigger a normal rebuild
curl -X POST http://127.0.0.1:<port>/rebuild

# To trigger a forced full rebuild, use:
curl -X POST 'http://127.0.0.1:<port>/rebuild?force=true'
```

*Note: The API returns a 202 Accepted response (`{"status": "started"}`) immediately. The rebuilding process will continue asynchronously inside Obsidian.*

## Step 4: Report

On successful request, report to the user that the index rebuild has been started in Obsidian.
On failure, surface the error and suggest checking that Obsidian is running and the embedding-viewer API is enabled on the correct port.
