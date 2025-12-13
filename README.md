# General Dexterity GitHub Actions

Reusable GitHub Actions for General Dexterity projects.

## Versioning

Use the `@v1` tag for the latest stable v1.x.x release:

```yaml
uses: general-dexterity/actions/pnpm-install@v1
```

For maximum stability, pin to a specific version (e.g., `@v1.0.0`).

## Actions

### `cloudflare-deploy-production`
Deploy to Cloudflare Workers production traffic using versions.

```yaml
- uses: general-dexterity/actions/cloudflare-deploy-production@v1
  with:
    cloudflare-api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    environment-url: https://example.com
    version-message: "abc1234 commit message"
    github-token: ${{ github.token }}
```

### `cloudflare-deploy-preview`
Deploy preview version with alias to Cloudflare Workers.

```yaml
- uses: general-dexterity/actions/cloudflare-deploy-preview@v1
  with:
    cloudflare-api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    preview-alias: pr-123
    version-message: "abc1234 commit message"
    github-token: ${{ github.token }}
```

### `check-dead-links`
Check for dead links in built site using hyperlink.

```yaml
- uses: general-dexterity/actions/check-dead-links@v1
  with:
    directory: dist
```

### `claude-tag`
Add a "Claude" label (orange) to PRs when the branch starts with `claude/`.

```yaml
# .github/workflows/claude-tag.yml
name: Claude Tag
on:
  pull_request:
    types: [opened]

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: general-dexterity/actions/claude-tag@v1
```

### `delete-deployment-environment`
Delete a GitHub deployment environment.

```yaml
- uses: general-dexterity/actions/delete-deployment-environment@v1
  with:
    github-token: ${{ github.token }}
    environment-name: pr-123
```

### `pnpm-install`
Install pnpm and dependencies with caching.

```yaml
- uses: general-dexterity/actions/pnpm-install@v1
```

### `pr-label-check`
Fail CI if specified labels are present on a PR. Useful for blocking PRs with labels like "blocked", "do-not-merge", or "wip".

Create a dedicated workflow file to trigger on label changes:

```yaml
# .github/workflows/pr-label-check.yml
name: PR Label Check
on:
  pull_request:
    types: [opened, synchronize, reopened, labeled, unlabeled]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: general-dexterity/actions/pr-label-check@v1
        # with:
        #   blocking-labels: blocked,do-not-merge  # optional, defaults to "blocked"
```
