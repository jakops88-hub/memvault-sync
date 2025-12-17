# MemVault Sync Action

Automatically index your repository's documentation, code, and knowledge into MemVault's GraphRAG-powered long-term memory system.

## What It Does

This GitHub Action:
- Scans specified files in your repository (Markdown, code, docs)
- Extracts knowledge, entities, and relationships using GraphRAG
- Syncs everything to your MemVault vault
- Enables semantic search and AI-powered retrieval across your codebase

## Quick Start

Add this to your workflow file (e.g., `.github/workflows/memvault-sync.yml`):

```yaml
name: Sync to MemVault

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Sync to MemVault
        uses: jakops88-hub/memvault-sync@v1
        with:
          memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `memvault_api_key` | ✅ Yes | - | Your MemVault API key (create at [memvault.com](https://memvault.com)) |
| `vault_id` | ❌ No | (default vault) | Specific vault ID to sync to |
| `file_paths` | ❌ No | `**/*.md` | Glob pattern for files to sync |

## Examples

### Basic Usage (Markdown Only)

```yaml
- uses: jakops88-hub/memvault-sync@v1
  with:
    memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
```

This syncs all `.md` files in your repository.

### Sync Specific Files

```yaml
- uses: jakops88-hub/memvault-sync@v1
  with:
    memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
    file_paths: |
      docs/**/*.md
      README.md
      ARCHITECTURE.md
```

### Sync Multiple File Types

```yaml
- uses: jakops88-hub/memvault-sync@v1
  with:
    memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
    file_paths: |
      **/*.md
      **/*.ts
      **/*.py
      **/*.js
```

### Sync to Specific Vault

```yaml
- uses: jakops88-hub/memvault-sync@v1
  with:
    memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
    vault_id: vault_production_123
    file_paths: docs/**/*.md
```

### Run on Pull Requests

```yaml
name: Sync PR Context to MemVault

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Sync PR to MemVault
        uses: jakops88-hub/memvault-sync@v1
        with:
          memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
          vault_id: vault_pr_review_${{ github.event.pull_request.number }}
```

### Scheduled Sync (Daily)

```yaml
name: Daily Documentation Sync

on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM UTC daily

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Sync to MemVault
        uses: jakops88-hub/memvault-sync@v1
        with:
          memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
```

## Setup Instructions

### 1. Get Your MemVault API Key

1. Sign up at [memvault.com](https://memvault.com)
2. Go to Dashboard → API Keys
3. Create a new API key
4. Copy the key (starts with `sk_`)

### 2. Add Secret to GitHub

1. Go to your repository → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: `MEMVAULT_API_KEY`
4. Value: Your API key (e.g., `sk_live_abc123...`)
5. Click "Add secret"

### 3. Create Workflow File

Create `.github/workflows/memvault-sync.yml`:

```yaml
name: Sync to MemVault

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: jakops88-hub/memvault-sync@v1
        with:
          memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
```

### 4. Trigger Sync

- **Automatic:** Push to main branch
- **Manual:** Go to Actions tab → Select workflow → Run workflow

## File Pattern Syntax

Uses [glob patterns](https://github.com/isaacs/minimatch):

- `**/*.md` - All Markdown files recursively
- `docs/**/*.md` - Markdown files in docs folder
- `*.{md,txt}` - Markdown and text files in root
- `src/**/*.{ts,js}` - TypeScript and JavaScript files in src
- `!node_modules/**` - Exclude node_modules (automatic)

## What Gets Indexed

MemVault's GraphRAG extracts:

- **Entities:** Functions, classes, concepts, technologies
- **Relationships:** How entities connect and relate
- **Context:** File structure, commit info, branch name
- **Metadata:** File paths, last modified, repository info

## Use Cases

### Documentation Search
```yaml
# Sync all docs → Ask MemVault "How do I deploy this project?"
file_paths: docs/**/*.md
```

### Code Understanding
```yaml
# Sync codebase → Ask "Which functions handle authentication?"
file_paths: src/**/*.{ts,js,py}
```

### Onboarding Bot
```yaml
# Sync everything → New devs ask questions, get instant answers
file_paths: |
  **/*.md
  src/**/*.ts
  README.md
```

### Architecture Queries
```yaml
# Sync architecture docs → Ask "What's the database schema?"
file_paths: |
  ARCHITECTURE.md
  docs/architecture/**
  prisma/schema.prisma
```

## Advanced Configuration

### Ignore Specific Paths

Action automatically ignores:
- `node_modules/`
- `.git/`
- `dist/`
- `build/`

To add custom ignores, use negative patterns:

```yaml
file_paths: |
  **/*.md
  !**/private/**
  !SECRETS.md
```

### Multiple Vaults

Sync different content to different vaults:

```yaml
jobs:
  sync-docs:
    steps:
      - uses: jakops88-hub/memvault-sync@v1
        with:
          memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
          vault_id: vault_documentation
          file_paths: docs/**/*.md

  sync-code:
    steps:
      - uses: jakops88-hub/memvault-sync@v1
        with:
          memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
          vault_id: vault_codebase
          file_paths: src/**/*.ts
```

## Troubleshooting

### "No files matched the provided pattern"

**Cause:** Glob pattern didn't match any files.

**Fix:** Check your pattern and file structure:
```bash
# Test pattern locally
ls -la docs/**/*.md
```

### "401 Unauthorized"

**Cause:** Invalid or missing API key.

**Fix:**
1. Verify secret name is exactly `MEMVAULT_API_KEY`
2. Check API key is valid (starts with `sk_`)
3. Ensure key has write permissions

### "API Error: 402 Payment Required"

**Cause:** Insufficient credits on your MemVault account.

**Fix:** Upgrade your plan or add credits at [memvault.com/billing](https://memvault.com/billing)

### "Action takes too long"

**Cause:** Syncing too many/large files.

**Fix:** Be more specific with `file_paths`:
```yaml
# Instead of all files
file_paths: '**/*'

# Sync only essential docs
file_paths: |
  README.md
  docs/**/*.md
```

## How It Works

1. **Checkout:** Action accesses your repository files
2. **Scan:** Matches files based on `file_paths` glob pattern
3. **Read:** Extracts content from matched files
4. **Send:** POSTs to MemVault API with metadata (repo, branch, commit)
5. **Process:** MemVault's GraphRAG extracts entities and relationships
6. **Index:** Knowledge stored in your vault, ready for semantic search

## API Endpoint

Action calls:
```
POST https://moderate-krystal-memvault-af80fe26.koyeb.app/api/memory/async
```

Payload:
```json
{
  "vaultId": "optional_vault_id",
  "repo": {
    "owner": "jakops88-hub",
    "name": "my-repo",
    "branch": "main",
    "commit": "abc123..."
  },
  "files": [
    {
      "path": "README.md",
      "content": "# My Project\n..."
    }
  ]
}
```

## Pricing

Syncing uses MemVault API credits:

| Plan | Monthly Credits | Cost |
|------|----------------|------|
| **Hobby** | 100,000 tokens | $29/mo |
| **Pro** | 1,000,000 tokens | $99/mo |

**Estimate:** ~1000 tokens per Markdown file

## Support

- **Documentation:** [memvault.com/docs](https://memvault.com/docs)
- **API Issues:** [github.com/jakops88-hub/Long-Term-Memory-API/issues](https://github.com/jakops88-hub/Long-Term-Memory-API/issues)
- **Action Issues:** [github.com/jakops88-hub/memvault-sync/issues](https://github.com/jakops88-hub/memvault-sync/issues)

## License

MIT

## Related Projects

- [MemVault API](https://github.com/jakops88-hub/Long-Term-Memory-API) - Backend API
- [MemVault SDK](https://www.npmjs.com/package/@memvault/client) - Official TypeScript/JavaScript SDK
- [MemVault Docs](https://memvault.com/docs) - Full documentation

---

**Made with 🧠 by MemVault**
