# shared-gha Repository Skills

This document defines the patterns and workflows for working with the shared-gha repository.

## Repository Purpose

Shared composite actions for this org's CI:
- **scan-image**: container vulnerability scan (Grype) that gates the build

`auth`, `terraform` and `docker-push` were removed on 2026-09-29. An org-wide
code search found no workflow referencing any of them, while they had drifted
years behind the action versions the repos actually use -- a maintenance
liability that read as a supported path. They are in git history if needed.
Call `google-github-actions/auth` directly for GCP auth, as every workflow in
the org already does.

## Before Any Change

**ALWAYS follow this pattern:**

1. **Research** the current state
   ```bash
   ls /Users/andriikostenetskyi/dev/homelab/shared-gha/
   ```

2. **Audit** to find the correct location
   - Image scan action: `scan-image/`

3. **Summary** before changing
   - State the root cause
   - Identify the file(s) to modify
   - Describe the fix

4. **Confirm** with the operator before proceeding

## Directory Structure

```
shared-gha/
├── scan-image/                # Grype scan + build gate
│   └── action.yml
└── README.md
```

## Available Actions

### scan-image - Container Vulnerability Scan
```yaml
- uses: PersonalAndriiKo/shared-gha/scan-image@main
  with:
    image: europe-west1-docker.pkg.dev/PROJECT_ID/repo/image:tag
    category: backend        # distinct per image, or uploads overwrite each other
```

Consumed by l1-gh-runners, threat-detector and deya-monitoring. It is referenced
as `@main`, so a change here reaches all of them on their next run -- there is
no pinning and no staging. Check consumers before changing behaviour:

```bash
gh api -X GET /search/code -f q='shared-gha org:PersonalAndriiKo'
```

## Required Permissions

Consuming workflows must include:
```yaml
permissions:
  contents: read
  id-token: write
```

## Security Benefits

- No long-lived credentials stored
- OIDC tokens expire in 1 hour
- Per-repository access control via WIF
- Full audit trail in Cloud Audit Logs

## Dependencies

- **tf-gcp**: WIF configuration in Terraform
- **GCP**: Workload Identity Federation setup

## Related Repositories

| Repo | Relationship |
|------|--------------|
| tf-gcp | WIF Terraform configuration |
| All repos | Consumers of these actions |
