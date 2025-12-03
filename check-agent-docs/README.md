# Check Agent Docs

Verifies that agent documentation files (AGENTS.md, CLAUDE.md) are up-to-date with the codebase. Uses Claude Code to analyze docs vs reality, scores freshness, and adds labels to PRs.

## Usage

```yaml
name: Check Agent Docs
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  check:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      issues: write
      id-token: write
    steps:
      - uses: actions/checkout@v5

      - uses: general-dexterity/actions/check-agent-docs@v1
        with:
          claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

## Setup

Run `/install-github-app` in Claude Code to set up authentication. This creates the `CLAUDE_CODE_OAUTH_TOKEN` secret automatically.

## How it works

1. PR opened/updated triggers the action
2. Claude analyzes documentation files against the actual codebase
3. Scores documentation freshness (0-100%)
4. Comments on PR with findings
5. Adds labels based on score thresholds
6. Fails check if score is below blocking threshold

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `anthropic-api-key` | No | - | Anthropic API key (direct API access) |
| `claude-code-oauth-token` | No | - | OAuth token from `/install-github-app` (recommended) |
| `doc-files` | No | `AGENTS.md,CLAUDE.md,.claude/CLAUDE.md` | Comma-separated list of doc files to check |
| `additional-instructions` | No | - | Custom scoring guidance for Claude |
| `model` | No | `claude-sonnet-4-20250514` | Claude model to use |
| `threshold-blocking` | No | `30` | Score below this fails the check |
| `threshold-warning` | No | `75` | Score below this adds warning label |
| `label-blocking` | No | `agent-docs-critical` | Label when docs are severely outdated |
| `label-warning` | No | `agent-docs-needs-update` | Label when docs need minor updates |
| `github-token` | No | `${{ github.token }}` | GitHub token for API access |

One of `anthropic-api-key` or `claude-code-oauth-token` is required.

## Outputs

| Output | Description |
|--------|-------------|
| `skipped` | `true` if check was skipped (no auth or no doc files) |
| `score` | Documentation freshness score (0-100) |
| `status` | `pass`, `warning`, or `blocking` |
| `label-added` | Label that was added to the PR (if any) |

## Scoring criteria

- **100%**: Documentation is fully accurate
- **75-99%**: Minor issues (typos, slightly outdated examples)
- **30-74%**: Moderate issues (missing sections, incorrect commands)
- **0-29%**: Severely outdated (wrong structure, broken commands)

## Examples

### Basic usage

```yaml
- uses: general-dexterity/actions/check-agent-docs@v1
  with:
    claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

### Custom doc files

```yaml
- uses: general-dexterity/actions/check-agent-docs@v1
  with:
    claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    doc-files: "CLAUDE.md,docs/ARCHITECTURE.md,docs/API.md"
```

### Custom thresholds

```yaml
- uses: general-dexterity/actions/check-agent-docs@v1
  with:
    claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    threshold-blocking: "20"
    threshold-warning: "80"
```

### Additional scoring instructions

```yaml
- uses: general-dexterity/actions/check-agent-docs@v1
  with:
    claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    additional-instructions: |
      Pay special attention to:
      - API endpoint documentation must be 100% accurate
      - Database schema changes are critical
      - Ignore code style conventions, focus on commands and structure
```

### Integration with pr-label-check

Block merging when docs are critically outdated:

```yaml
jobs:
  check-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: general-dexterity/actions/check-agent-docs@v1
        with:
          claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}

  block-merge:
    runs-on: ubuntu-latest
    steps:
      - uses: general-dexterity/actions/pr-label-check@v1
        with:
          blocking-labels: "blocked,agent-docs-critical"
```

## PR comment format

Claude comments on PRs with this format:

```markdown
## Agent Docs Freshness Check

**Score: 65%** ⚠️

### Summary
Documentation is moderately outdated with missing worker references.

### Issues Found
- AGENTS.md references 2 workers but 4 exist
- Migration directory inconsistency not documented

### Suggested Fixes
- Update worker count in architecture section
- Document migration directory differences
```
