# shared-gha Repository Skills

This document defines the patterns and workflows for working with the shared-gha repository.

## Repository Purpose

Shared GitHub Actions for GCP WIF authentication:
- **auth**: GCP WIF authentication (keyless)
- **terraform**: Terraform with WIF
- **docker-push**: Docker build and push to GAR

## Before Any Change

**ALWAYS follow this pattern:**

1. **Research** the current state
   ```bash
   ls /Users/andriikostenetskyi/dev/homelab/shared-gha/
   ```

2. **Audit** to find the correct location
   - Auth action: `auth/`
   - Terraform action: `terraform/`
   - Docker push action: `docker-push/`

3. **Summary** before changing
   - State the root cause
   - Identify the file(s) to modify
   - Describe the fix

4. **Confirm** with the operator before proceeding

## Directory Structure

```
shared-gha/
├── auth/                      # GCP WIF authentication action
│   └── action.yml
├── terraform/                 # Terraform with WIF action
│   └── action.yml
├── docker-push/               # Docker build & push action
│   └── action.yml
└── README.md
```

## Available Actions

### auth - GCP WIF Authentication
```yaml
- uses: PersonalAndriiKo/shared-gha/auth@main
  with:
    workload_identity_provider: 'projects/PROJECT_ID/locations/global/workloadIdentityPools/github-actions/providers/github-oidc'
    service_account: 'my-sa@PROJECT_ID.iam.gserviceaccount.com'
```

### terraform - Terraform with WIF
```yaml
- uses: PersonalAndriiKo/shared-gha/terraform@main
  with:
    workload_identity_provider: ${{ vars.WIF_PROVIDER }}
    service_account: ${{ vars.TF_SERVICE_ACCOUNT }}
    command: plan
```

### docker-push - Docker Build and Push to GAR
```yaml
- uses: PersonalAndriiKo/shared-gha/docker-push@main
  with:
    workload_identity_provider: ${{ vars.WIF_PROVIDER }}
    service_account: ${{ vars.DOCKER_SERVICE_ACCOUNT }}
    registry: europe-west1-docker.pkg.dev
    image_name: europe-west1-docker.pkg.dev/PROJECT_ID/repo/image
    tags: latest,${{ github.sha }}
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
