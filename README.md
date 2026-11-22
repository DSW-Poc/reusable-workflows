# reusable-workflows

Central, versioned reusable GitHub Actions workflows for DSW-Poc.

| Workflow | Purpose |
|---|---|
| `build-test.yml` | Set up Node, run `npm test`, upload an artifact, output its name |
| `terraform.yml` | Plan with a read-only role, wait for environment approval, apply the same plan with a write role |
| `aws-identity.yml` | Assume an AWS role via OIDC in a caller environment; output the account ID |

## Usage

```yaml
jobs:
  build:
    uses: DSW-Poc/reusable-workflows/.github/workflows/build-test.yml@v1
    with:
      node-version: "22"
```

Pin to a tag (`@v1`) or a commit SHA, never `@main`, in app repos.

### AWS (OIDC)

The caller job must grant `id-token: write`:

```yaml
  whoami-dev:
    needs: build
    permissions:
      id-token: write
      contents: read
    uses: DSW-Poc/reusable-workflows/.github/workflows/aws-identity.yml@v1
    with:
      environment: dev
      role-arn: ${{ vars.AWS_ROLE_ARN_DEV }}
      aws-region: ${{ vars.AWS_REGION }}
```

### Terraform (plan -> approval -> apply)

```yaml
  prod:
    concurrency: { group: terraform-prod, cancel-in-progress: false }
    permissions: { id-token: write, contents: read }
    uses: DSW-Poc/reusable-workflows/.github/workflows/terraform.yml@v1
    with:
      environment: prod
      aws-region: ${{ vars.AWS_REGION }}
      plan-role-arn: ${{ vars.AWS_ROLE_ARN_PROD_PLAN }}
      apply-role-arn: ${{ vars.AWS_ROLE_ARN_PROD_APPLY }}
      state-bucket: ${{ vars.TF_STATE_BUCKET }}
      state-key: demo-app/prod/terraform.tfstate
```

## `terraform-checkov.yml`: static security scan of Terraform

Runs [Checkov](https://www.checkov.io/) on a directory of `.tf` files. It needs no cloud credentials. Findings appear in three places: the job log, a table in the job summary, and (optionally) GitHub code scanning via SARIF.

```yaml
jobs:
  scan:
    permissions: { contents: read, security-events: write }   # private repos: + actions: read
    uses: DSW-Poc/reusable-workflows/.github/workflows/terraform-checkov.yml@v1
    with:
      working-directory: infra
      upload-sarif: ${{ !github.event.pull_request.head.repo.fork }}   # fork PRs get a read-only token
```

| Input | Default | Meaning |
|---|---|---|
| `working-directory` | `infra` | Directory to scan |
| `soft-fail` | `false` | Report findings without failing (audit / rollout mode) |
| `skip-checks` | `""` | Comma-separated check IDs to skip. Prefer an inline `# checkov:skip=ID:reason` on the resource. |
| `upload-sarif` | `true` | Upload to code scanning |

Output: `failed-count`. Make the `scan / checkov` check **required** in a ruleset, and don't put a `paths:` filter on the caller, or PRs that skip the workflow stay blocked waiting for the check.
