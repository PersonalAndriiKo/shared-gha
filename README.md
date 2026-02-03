# Shared GitHub Actions for GCP WIF Authentication

Reusable GitHub Actions for Workload Identity Federation (WIF) authentication to Google Cloud Platform.

## Actions

### `auth` - GCP WIF Authentication

Authenticate to GCP using OIDC tokens (keyless authentication).

```yaml
- uses: PersonalAndriiKo/shared-gha/auth@main
  with:
    workload_identity_provider: 'projects/PROJECT_ID/locations/global/workloadIdentityPools/github-actions/providers/github-oidc'
    service_account: 'my-sa@PROJECT_ID.iam.gserviceaccount.com'
```

### `terraform` - Terraform with WIF

Run Terraform commands with automatic WIF authentication.

```yaml
- uses: PersonalAndriiKo/shared-gha/terraform@main
  with:
    workload_identity_provider: ${{ vars.WIF_PROVIDER }}
    service_account: ${{ vars.TF_SERVICE_ACCOUNT }}
    command: plan
```

### `docker-push` - Docker Build and Push to GAR

Build and push Docker images to Google Artifact Registry.

```yaml
- uses: PersonalAndriiKo/shared-gha/docker-push@main
  with:
    workload_identity_provider: ${{ vars.WIF_PROVIDER }}
    service_account: ${{ vars.DOCKER_SERVICE_ACCOUNT }}
    registry: europe-west1-docker.pkg.dev
    image_name: europe-west1-docker.pkg.dev/PROJECT_ID/repo/image
    tags: latest,${{ github.sha }}
```

## Prerequisites

1. **Workload Identity Federation** configured in GCP
2. **Service Account** with appropriate IAM bindings
3. **Repository permissions** must include `id-token: write`

```yaml
permissions:
  contents: read
  id-token: write
```

## Security

- No long-lived credentials stored
- OIDC tokens expire in 1 hour (max)
- Per-repository access control via WIF attribute conditions
- Full audit trail in Cloud Audit Logs

## Setup

See [tf-gcp](https://github.com/PersonalAndriiKo/tf-gcp) for Terraform configuration to set up WIF.

## License

MIT License - see [LICENSE](LICENSE)
