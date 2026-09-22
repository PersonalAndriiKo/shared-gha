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

### `scan-image` - Container Vulnerability Scan

Scan a container image with Grype and fail the build on findings that have a
fix available.

```yaml
- uses: PersonalAndriiKo/shared-gha/scan-image@main
  with:
    image: europe-west1-docker.pkg.dev/PROJECT_ID/repo/image:tag
    category: backend        # distinct per image, or uploads overwrite each other
```

Scan the image **before** pushing it — build with `load: true`, scan, then
push — so a vulnerable image is never published.

| input | default | meaning |
| --- | --- | --- |
| `image` | required | image to scan; must be local or pullable |
| `grype_version` | `v0.119.0` | release tag, installed and checksum-verified |
| `severity_threshold` | `7.0` | CVSS at or above which a finding blocks (7.0 is High) |
| `require_fix` | `true` | only block when a fix exists |
| `upload_sarif` | `true` | send results to code scanning |
| `category` | `grype` | code scanning category |

`require_fix: true` is the default deliberately. Base images routinely carry
Highs with no published fix — `alpine:3.24` currently ships zlib
CVE-2026-85091 and four wget CVEs, none fixable — so blocking on every High
would stop every build on something nobody can resolve. That is how gates end
up switched off. Unfixable findings are still printed and uploaded, so they
stay visible and get picked up when a fix lands. Set `require_fix: false` for
a stricter gate where the base image is under your control.

Uploading requires `permissions: security-events: write` in the calling job.

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
