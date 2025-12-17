# MemVault Sync Action

![Marketplace](https://img.shields.io/badge/GitHub_Marketplace-Available-purple?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)

> **Automatically sync your repository documentation and code into your MemVault Knowledge Graph.**

This GitHub Action connects your repository to your **MemVault** account. It automatically pushes changes in your documentation (Markdown files) or source code to your long-term memory graph, ensuring your AI agents always have the latest context.

---

## Usage

Add this to your `.github/workflows/memvault.yml`:

```yaml
name: Sync to MemVault
on:
  push:
    branches: [ main ]
    paths:
      - 'docs/**' # Only sync when docs change
      - 'src/**/*.ts' # Or specific code files

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: MemVault Sync
        uses: jakops88-hub/memvault-sync@v1
        with:
          memvault_api_key: ${{ secrets.MEMVAULT_API_KEY }}
          # Optional: Filter specific files
          file_paths: "docs/**/*.md" 
          # Optional: Set this to your MemVault API instance
          api_url: "[https://memvault-demo-g38n.vercel.app/api](https://memvault-demo-g38n.vercel.app/api)"
```

---

## Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `memvault_api_key` | Your API Key from the MemVault Dashboard. | **Yes** | N/A |
| `file_paths` | Glob pattern for files to sync. | No | `**/*.md` |
| `vault_id` | Specific Vault ID (if using multiple vaults). | No | `default` |
| `api_url` | Base URL for the MemVault API. | No | `https://api.memvault.com` |

---

## Setup Instructions

1.  **Get your API Key:** Log in to [MemVault Dashboard](https://memvault-demo-g38n.vercel.app/dashboard) and generate a key.
2.  **Add Secret:** Go to your Repo Settings -> Secrets and Variables -> Actions -> New Repository Secret. Name it `MEMVAULT_API_KEY`.
3.  **Create Workflow:** Copy the YAML above into `.github/workflows/memvault.yml`.

---

## License

MIT
