# General Dexterity GitHub Actions

Reusable GitHub Actions for General Dexterity projects.

## Actions

### `cloudflare-deploy-production`
Deploy to Cloudflare Workers production traffic using versions.

```yaml
- uses: general-dexterity/actions/cloudflare-deploy-production@main
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
- uses: general-dexterity/actions/cloudflare-deploy-preview@main
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
- uses: general-dexterity/actions/check-dead-links@main
  with:
    directory: dist
```

### `delete-deployment-environment`
Delete a GitHub deployment environment.

```yaml
- uses: general-dexterity/actions/delete-deployment-environment@main
  with:
    github-token: ${{ github.token }}
    environment-name: pr-123
```

### `pnpm-install`
Install pnpm and dependencies with caching.

```yaml
- uses: general-dexterity/actions/pnpm-install@main
```
