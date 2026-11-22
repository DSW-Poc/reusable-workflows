# reusable-workflows

Central, versioned reusable GitHub Actions workflows for DSW-Poc.

| Workflow | Purpose |
|---|---|
| `build-test.yml` | Set up Node, run `npm test`, upload an artifact, output its name |
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
