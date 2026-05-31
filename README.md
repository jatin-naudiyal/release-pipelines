# Release Pipelines (Automated Build, Test, and Deploy)

This repository demonstrates a complete CI/CD pipeline for a Node.js application using GitHub Actions. The pipeline automates the process of building, testing, tagging, releasing, and deploying code across development (dev), quality assurance (qa), and production (prod) environments.

## 🚀 Environments Overview

The pipeline uses three distinct environments, each with its own workflow and promotion rules:

| Environment | Trigger | Branch | Artifacts | Deployment |
|-------------|---------|--------|-----------|------------|
| **Dev** | `push` to `develop` | `develop` | Latest image | Automatic |
| **QA** | Manual trigger (workflow_dispatch) | `qa` | Tagged images | Automatic |
| **Prod** | `release` event or manual trigger | `main` | Tagged images | Manual (no build) |

---

## 📂 Project Structure

```
release-pipelines/
├── .github/workflows/   # CI/CD pipeline configurations
│   ├── dev.yml          # Development environment workflow
│   ├── qa.yml           # QA environment workflow
│   └── prod.yml         # Production environment workflow
├── src/
│   ├── index.js         # Node.js application entry point
│   └── package.json     # Project dependencies and scripts
├── Dockerfile           # Docker image configuration
└── README.md            # This file
```

---

## 🛠️ Development Setup

### Prerequisites

- Node.js and npm installed
- Docker installed (for local testing)
- GitHub Container Registry (GHCR) access
- Required secrets in GitHub repository:
  - `GITHUB_TOKEN` (automatically available)

### Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd release-pipelines
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Run locally:
   ```bash
   npm start
   ```

---

## ⚙️ CI/CD Pipeline Details

### 1. Dev Pipeline (`.github/workflows/dev.yml`)

**Trigger**: Any push to the `develop` branch

**Process**:
1. Checkout code
2. Login to GitHub Container Registry (GHCR)
3. Build Docker image
4. Tag image with Git SHA and branch
5. Push image to GHCR
6. Deploy to dev environment (placeholder)

**Example Workflow Dispatch**: (Manual trigger)
```yaml
name: Dev Workflow
on:
  workflow_dispatch:
    inputs:
      head:
        description: 'Branch to merge into QA'
        required: true
        default: 'develop'
      bump:
        description: 'Version bump type'
        required: true
        default: 'patch'
        type: choice
        options:
          - patch
          - minor
          - major
```

### 2. QA Pipeline (`.github/workflows/qa.yml`)

**Trigger**: Manual trigger (workflow_dispatch)

**Process**:
1. Checkout code
2. **Merge** specified branch into `qa` branch
3. **Bump version** (patch/minor/major)
4. Push version tag
5. Create prerelease on GitHub
6. Extract version number
7. Login to GHCR
8. Build and push Docker image with version tag
9. Deploy to qa environment (placeholder)

**Example Manual Trigger**:
```bash
gh workflow run qa.yml -f head=develop -f bump=patch
```

### 3. Prod Pipeline (`.github/workflows/prod.yml`)

**Trigger**: GitHub Release event or manual trigger

**Process**:
1. Checkout code
2. **Merge** `qa` branch into `main` branch
3. Extract version number from release
4. Deploy to prod environment (placeholder) - **No build step**

**Example Manual Trigger**:
```bash
gh workflow run prod.yml -f releaseVersion=v1.0.0
```

---

## 📦 Container Registry (GHCR)

All Docker images are pushed to GitHub Container Registry with the following tagging strategy:

- **Dev** (automatic): Tagged with Git SHA and branch name
- **QA** (automatic): Tagged with version number (e.g., `v1.1.0`, `v1.1.1-prerelease.1`)
- **Prod** (manual): Tagged with version number (e.g., `v1.0.0`)

**Example Image URLs**:
- `ghcr.io/your-org/hello-world:sha-abcdef123` (Dev)
- `ghcr.io/your-org/hello-world:v1.0.0-prerelease.1` (QA)
- `ghcr.io/your-org/hello-world:v1.0.0` (Prod)

---

## 🏃 How to Run Workflows

### Trigger Dev Workflow
```bash
# Push to develop (automatic)
git add .
git commit -m "feat: add new feature"
git push origin develop
```

### Trigger QA Workflow
```bash
# Using GitHub CLI
gh workflow run qa.yml -f head=develop -f bump=patch

# Manual trigger via GitHub Actions UI
# Go to Actions → Select "QA Workflow" → Click "Run workflow"
```

### Trigger Prod Workflow
```bash
# Using GitHub CLI
gh workflow run prod.yml -f releaseVersion=v1.0.0

# Or create a release on GitHub:
# Go to Releases → Draft a new release → Set version, publish
```

---

## 🧪 Testing

### Unit Tests
(No tests currently configured in `package.json`)

### Integration Tests
(No integration tests currently configured)

### Manual Verification

**Check Dev Environment**:
```bash
# After dev workflow completes
curl http://localhost:3000
```

**Check QA Environment**:
```bash
# After qa workflow completes
curl http://localhost:3000
```

**Check Prod Environment**:
```bash
# After prod workflow completes
curl http://localhost:3000
```

---

## 🔐 Secrets Configuration

Ensure the following secrets are added to your GitHub repository settings:

- `GITHUB_TOKEN`: (Required) Auto-generated, sufficient for most operations
- `REGISTRY_USER`: Your GHCR username (defaults to `github.actor`)
- `REGISTRY_TOKEN`: Your GHCR access token (if not using `GITHUB_TOKEN`)
- `KUBE_CONFIG`: Base64-encoded Kubernetes config (for actual deployment)
- `KUBE_NAMESPACE`: Kubernetes namespace (for actual deployment)

---

## ⚠️ Important Notes

1. **Deployment Placeholders**:
   The current workflows include placeholder deployment steps:
   ```bash
   # Replace with your actual deployment command
   echo "Deploying to QA environment..."
   ```
   
   For actual Kubernetes deployment, you would use:
   ```bash
   # Example with kubectl
   echo "${{ env.KUBE_CONFIG }}" | base64 -d > kubeconfig.yaml
   KUBECONFIG=kubeconfig.yaml kubectl set image deployment/${{ env.DEPLOYMENT_NAME }} \
     ${{ env.CONTAINER_NAME }}=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.VERSION_NUM }}
   ```

2. **Branch Management**: The workflows automatically merge branches:
   - Dev: `develop` → `qa` (in `prod.yml`)
   - Always use the `qa` branch for manual triggers

3. **Version Bumping**: Only patch version bumps are currently configured in `qa.yml`. You can extend this to support minor/major releases.

---

## 🤝 Contributing

1. Create a feature branch from `develop`
2. Make your changes
3. Test locally
4. Create a pull request to `develop`
5. Once