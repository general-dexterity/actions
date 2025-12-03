# Actions Repository

Reusable GitHub Actions for General Dexterity projects.

## Structure

Each action lives in its own directory with a single `action.yml` file (composite actions, no custom scripts).

Actions: `check-agent-docs`, `check-dead-links`, `cloudflare-deploy-preview`, `cloudflare-deploy-production`, `delete-deployment-environment`, `pnpm-install`, `pr-label-check`

## Commands

No build/test commands. Actions are YAML-only composite actions validated by GitHub when used.

## Conventions

- Directory names: kebab-case (`pr-label-check`)
- Input/output names: kebab-case (`blocking-labels`, `deployment-id`)
- Use `runs: using: "composite"` for all actions
- Prefer external actions via `uses:` over custom bash scripts
- Shell scripts use `shell: bash` with `$GITHUB_OUTPUT` for outputs

## Versioning

- `v1` tag always points to latest stable
- Patch bump (v1.0.x): docs, refactors, updates to existing actions
- Minor bump (v1.x.0): new actions
- Commit style: Conventional Commits with action scope, e.g., `feat(pr-label-check): add action`
