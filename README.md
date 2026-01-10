# Gwent Workflows

Shared, reusable GitHub Actions workflows for Gwent microservices.

## Available Workflows

### `build-test-node.yml`

Generic Node.js build and test workflow.

```yaml
jobs:
  build:
    uses: cobyzim/gwent-workflows/.github/workflows/build-test-node.yml@v1
    with:
      node-version: '22'  # optional, defaults to '22'
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `node-version` | No | `22` | Node.js version |
| `working-directory` | No | `.` | Directory to run commands in |
| `skip-build` | No | `false` | Skip the build step |
| `skip-test` | No | `false` | Skip the test step |

---

### `deploy-lambda.yml`

Deploy a Lambda function snapshot using Terragrunt.

```yaml
jobs:
  deploy:
    uses: cobyzim/gwent-workflows/.github/workflows/deploy-lambda.yml@v1
    with:
      lambda-name: profile-lambda
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      TF_MODULES_SSH_KEY: ${{ secrets.TF_MODULES }}
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `lambda-name` | Yes | - | Lambda function name |
| `aws-region` | No | `us-east-1` | AWS region to deploy to |
| `terragrunt-runner-image` | No | `ghcr.io/cobyzim/terragrunt-runner:latest` | Docker image |

**Required Secrets:**
- `AWS_ACCESS_KEY_ID` - AWS deployment credentials
- `AWS_SECRET_ACCESS_KEY` - AWS deployment credentials
- `TF_MODULES_SSH_KEY` - SSH key for private Terraform modules

**Expected Repository Structure:**
```
your-lambda-repo/
├── package.json        # Contains version
├── src/                # Lambda function code
├── nodejs/             # Lambda layer dependencies
└── infrastructure/     # Terragrunt configs
    ├── root.hcl
    ├── lambda_function/
    ├── lambda_layer/
    ├── alias/
    └── api_gateway/
```

---

### `release-lambda.yml`

Create a GitHub release and bump to next snapshot version.

```yaml
jobs:
  release:
    if: github.event.pull_request.merged == true
    uses: cobyzim/gwent-workflows/.github/workflows/release-lambda.yml@v1
    with:
      lambda-name: profile-lambda
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `lambda-name` | Yes | - | Lambda function name (used in release tag) |

**What it does:**
1. Extracts version from `package.json`
2. Creates GitHub release with tag `{lambda-name}-v{version}`
3. Bumps version to next snapshot (e.g., `1.0.0` → `1.0.1-SNAPSHOT`)
4. Commits and pushes the version bump

---

## Example: Complete Lambda Repository Workflows

**`.github/workflows/ci.yml`**
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    uses: cobyzim/gwent-workflows/.github/workflows/build-test-node.yml@v1
```

**`.github/workflows/deploy.yml`**
```yaml
name: Deploy Snapshot
on:
  workflow_dispatch:

jobs:
  deploy:
    uses: cobyzim/gwent-workflows/.github/workflows/deploy-lambda.yml@v1
    with:
      lambda-name: profile-lambda
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      TF_MODULES_SSH_KEY: ${{ secrets.TF_MODULES }}
```

**`.github/workflows/release.yml`**
```yaml
name: Release
on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  release:
    if: github.event.pull_request.merged == true
    uses: cobyzim/gwent-workflows/.github/workflows/release-lambda.yml@v1
    with:
      lambda-name: profile-lambda
```

---

## Versioning

Use Git tags for versioning (e.g., `v1`, `v1.0.0`). Reference workflows by tag:

```yaml
uses: cobyzim/gwent-workflows/.github/workflows/deploy-lambda.yml@v1
```
