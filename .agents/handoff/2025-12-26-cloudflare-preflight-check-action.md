# Handoff: Create Cloudflare Preflight Check Action

## Context Summary

We're setting up GitHub Actions workflows for `quelle-ecole.com` (a French post-bac education platform) that deploy to Cloudflare Workers. The workflows use reusable actions from `general-dexterity/actions`. During testing, we discovered that deployments fail silently or with cryptic errors when:

1. The Cloudflare API token lacks Workers permissions
2. The Worker doesn't exist yet (first deploy)
3. The token can't access the specific worker

These issues only surface at deploy time with unhelpful error messages like:
```
Could not route to /client/v4/accounts/***/workers/services/quelle-ecole-site, perhaps your object identifier is invalid? [code: 7003]
```

## Work Completed

1. Created GitHub Actions workflows in `quelle-ecole.com`:
   - `.github/workflows/preview.yml` - PR preview deployments
   - `.github/workflows/production.yml` - Production deployments on push to main
   - `.github/workflows/cleanup.yml` - Delete preview environments on PR close
   - `.github/workflows/claude-tag.yml` - Label PRs from claude/* branches
   - `.github/workflows/pr-label-check.yml` - Block PRs with "blocked" label

2. Created local action `.github/actions/setup-registry/action.yml` for npm registry auth

3. Identified root cause of deployment failures:
   - User's API token only had DNS/Email permissions, not Workers permissions
   - The `versions upload` command requires an existing worker

## Current State

The `general-dexterity/actions` repository has these Cloudflare-related actions:
- `cloudflare-deploy-preview/action.yml` - Uses `wrangler versions upload --preview-alias`
- `cloudflare-deploy-production/action.yml` - Uses `wrangler versions upload` then `wrangler versions deploy`

Both assume:
- Valid API token with Workers permissions
- Worker already exists on Cloudflare

When these assumptions fail, users get confusing errors.

## Next Steps

Create a new action `cloudflare-preflight-check` that validates the Cloudflare setup before deployment. It should:

### 1. Check API Token Validity
- Verify the token is valid by calling Cloudflare API
- Confirm it has Workers Scripts permissions

### 2. Check Worker Exists
- Query if the worker service exists: `GET /accounts/{account_id}/workers/services/{service_name}`
- If not, provide clear error message suggesting first deploy with `wrangler deploy`

### 3. Check Token Has Access to Worker
- Verify the token can access the specific worker
- Handle case where token is valid but scoped to different account/zone

### Suggested Implementation

```yaml
# cloudflare-preflight-check/action.yml
name: "Cloudflare Preflight Check"
description: "Validate Cloudflare API token and worker existence before deployment"

inputs:
  cloudflare-api-token:
    description: "Cloudflare API token"
    required: true
  cloudflare-account-id:
    description: "Cloudflare account ID"
    required: true
  worker-name:
    description: "Name of the worker service"
    required: true

outputs:
  worker-exists:
    description: "Whether the worker exists (true/false)"
  token-valid:
    description: "Whether the token is valid and has correct permissions"

runs:
  using: "composite"
  steps:
    - name: Check API token and worker
      shell: bash
      run: |
        # Check token validity by verifying user
        TOKEN_CHECK=$(curl -s -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" \
          -H "Authorization: Bearer ${{ inputs.cloudflare-api-token }}" \
          -H "Content-Type: application/json")
        
        if echo "$TOKEN_CHECK" | grep -q '"success":false'; then
          echo "::error::Cloudflare API token is invalid"
          exit 1
        fi
        
        # Check if worker exists
        WORKER_CHECK=$(curl -s -X GET "https://api.cloudflare.com/client/v4/accounts/${{ inputs.cloudflare-account-id }}/workers/services/${{ inputs.worker-name }}" \
          -H "Authorization: Bearer ${{ inputs.cloudflare-api-token }}" \
          -H "Content-Type: application/json")
        
        if echo "$WORKER_CHECK" | grep -q '"code":10007'; then
          echo "::error::Worker '${{ inputs.worker-name }}' does not exist. Run 'wrangler deploy' locally first to create it."
          echo "worker-exists=false" >> $GITHUB_OUTPUT
          exit 1
        fi
        
        if echo "$WORKER_CHECK" | grep -q '"code":7003'; then
          echo "::error::API token does not have permission to access Workers. Ensure token has 'Workers Scripts: Edit' permission."
          exit 1
        fi
        
        if echo "$WORKER_CHECK" | grep -q '"success":false'; then
          ERROR_MSG=$(echo "$WORKER_CHECK" | jq -r '.errors[0].message // "Unknown error"')
          echo "::error::Failed to check worker: $ERROR_MSG"
          exit 1
        fi
        
        echo "worker-exists=true" >> $GITHUB_OUTPUT
        echo "token-valid=true" >> $GITHUB_OUTPUT
        echo "✅ Preflight check passed: Token valid and worker exists"
```

### Integration

Update `cloudflare-deploy-preview` and `cloudflare-deploy-production` to use this as a first step, or document it for users to add before deploy actions.

## Important Notes

1. **API Token Permissions Required**: Token needs "Edit Cloudflare Workers" template or manual `Account > Workers Scripts > Edit` permission

2. **First Deploy**: The `versions upload` command only works on existing workers. First deploy must use `wrangler deploy`

3. **Error Code Reference**:
   - `7003`: Route not found / no permission to access resource
   - `10007`: Worker/service not found

4. **Cloudflare API Docs**: 
   - Token verify: `GET /user/tokens/verify`
   - Worker services: `GET /accounts/{account_id}/workers/services/{service_name}`

## File References

**In `general-dexterity/actions`:**
- `cloudflare-deploy-preview/action.yml` - Current preview deploy action
- `cloudflare-deploy-production/action.yml` - Current production deploy action
- `cloudflare-preflight-check/action.yml` - TO BE CREATED

**In `quelle-ecole.com` (for context):**
- `.github/workflows/preview.yml` - Uses cloudflare-deploy-preview
- `.github/workflows/production.yml` - Uses cloudflare-deploy-production
- `apps/site/wrangler.jsonc` - Worker config with name `quelle-ecole-site`
