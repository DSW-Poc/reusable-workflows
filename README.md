# reusable-workflows

Central, versioned reusable GitHub Actions workflows for DSW-Poc.

| Workflow | Purpose |
|---|---|
| `build-test.yml` | Set up Node, run `npm test`, upload an artifact, output its name |

## Usage

```yaml
jobs:
  build:
    uses: DSW-Poc/reusable-workflows/.github/workflows/build-test.yml@v1
    with:
      node-version: "22"
```

Pin to a tag (`@v1`) or a commit SHA, never `@main`, in app repos.
