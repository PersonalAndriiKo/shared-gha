# Shared GitHub Actions

Reusable composite actions for this org's CI.

> **2026-09-29:** the `auth`, `terraform` and `docker-push` actions were
> removed. Nothing referenced them — an org-wide code search found consumers
> only for `scan-image` — and they had drifted years behind the versions the
> repos actually use, so they were a maintenance liability that looked like a
> supported path. They remain in git history if one is ever needed again.
>
> For GCP auth, call `google-github-actions/auth` directly, as every workflow
> in the org already does.

## Actions

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
