# GitHub Actions CI/CD Workflow for Angular Deployment

## Issue Resolution: Environment Configuration

**Problem:** The `environment` configuration was incorrectly placed at the root level of the workflow YAML, causing the error:
```
(Line: 13, Col: 1): Unexpected value 'environment'
```

**Solution:** The `environment` configuration must be placed inside the job definition, not at the workflow root level.

## Corrected Workflow Structure

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: 'pages'
  cancel-in-progress: true

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    environment:
      name: uat-environment
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build Angular app
        run: |
          npm run build -- --base-href "/${{ github.event.repository.name }}/"
          cp dist/*/index.html dist/*/404.html

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist/*

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## Key Corrections Made:
1. Moved `environment` block inside the `build-deploy` job
2. Maintained proper YAML indentation
3. Kept all other configurations intact

## Workflow Features:
- ✅ Triggers on push to main branch and manual dispatch
- ✅ Proper permissions for GitHub Pages deployment
- ✅ Environment tracking with dynamic URL output
- ✅ Node.js 18 with npm caching
- ✅ Automatic repository name detection
- ✅ SPA routing fix with 404.html fallback
- ✅ Concurrency control for deployments