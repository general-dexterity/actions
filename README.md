# General Dexterity GitHub Actions

Reusable GitHub Actions for General Dexterity projects.

## Versioning

Use the `@v2` tag for the latest stable release:

```yaml
uses: general-dexterity/actions/pnpm-install@v2
```

For maximum stability, pin to a specific version (e.g., `@v2.0.0`).

## Actions

### `cloudflare-deploy-production`
Deploy to Cloudflare Workers production traffic using versions.

```yaml
- uses: general-dexterity/actions/cloudflare-deploy-production@v2
  with:
    cloudflare-api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    environment-url: https://example.com
    version-message: "abc1234 commit message"
    github-token: ${{ github.token }}
```

### `cloudflare-deploy-preview`
Deploy preview version with alias to Cloudflare Workers. Automatically comments the preview URL on the PR.

```yaml
- uses: general-dexterity/actions/cloudflare-deploy-preview@v2
  with:
    cloudflare-api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    preview-alias: pr-123
    version-message: "abc1234 commit message"
    github-token: ${{ github.token }}
    pr-number: ${{ github.event.pull_request.number }}
```

The `pr-number` input is optional. If omitted, the action tries to resolve it from the pull request event context. Pass it explicitly when running in `workflow_run` triggers where the PR context isn't directly available (e.g., `${{ github.event.workflow_run.pull_requests[0].number }}`).

### `wrangler-run`
Run a wrangler command with optional pre/post commands. Use this instead of `cloudflare/wrangler-action` for tasks like D1 migrations.

```yaml
- uses: general-dexterity/actions/wrangler-run@v2
  with:
    command: "d1 migrations apply MY_DB --remote"
    api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    account-id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

### `check-dead-links`
Check for dead links in built site using hyperlink.

```yaml
- uses: general-dexterity/actions/check-dead-links@v2
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
    types: [opened, reopened]

jobs:
  label:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read
    steps:
      - uses: general-dexterity/actions/claude-tag@v2
```

### `pnpm-install`
Install pnpm and dependencies with caching.

```yaml
- uses: general-dexterity/actions/pnpm-install@v2
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
      - uses: general-dexterity/actions/pr-label-check@v2
        # with:
        #   blocking-labels: blocked,do-not-merge  # optional, defaults to "blocked"
```
