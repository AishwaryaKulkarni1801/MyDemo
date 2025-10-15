# GitHub Actions CI/CD Workflow for Angular Deployment

## Issue Resolution: Multiple Deployment Fixes

**Problem 1:** The `environment` configuration was incorrectly placed at the root level of the workflow YAML, causing the error:
```
(Line: 13, Col: 1): Unexpected value 'environment'
```

**Solution 1:** The `environment` configuration must be placed inside the job definition, not at the workflow root level.

**Problem 2:** The file copy command failed with:
```
cp: cannot create regular file 'dist/*/404.html': No such file or directory
Error: Process completed with exit code 1.
```

**Solution 2:** The wildcard pattern `dist/*/` wasn't being expanded properly by the shell. Fixed by using `find` command to locate and copy the index.html file to 404.html in the correct directory.

**Problem 3:** The artifact upload failed with:
```
tar: dist/*: Cannot open: No such file or directory
tar: Error is not recoverable: exiting now
Error: Process completed with exit code 2.
```

**Solution 3:** Changed the upload path from `dist/*` to `dist` to upload the entire dist directory instead of using wildcards.

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
          
      - name: Fix routing for SPA
        run: |
          find dist -name "index.html" -execdir cp {} 404.html \;

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## Key Corrections Made:
1. Moved `environment` block inside the `build-deploy` job
2. Maintained proper YAML indentation
3. Separated build and file copy into different steps
4. Used `find` command with `-execdir` to properly copy index.html to 404.html
5. Changed artifact upload path from `dist/*` to `dist` to avoid wildcard issues
6. Kept all other configurations intact

## Workflow Features:
- ✅ Triggers on push to main branch and manual dispatch
- ✅ Proper permissions for GitHub Pages deployment
- ✅ Environment tracking with dynamic URL output
- ✅ Node.js 18 with npm caching
- ✅ Automatic repository name detection
- ✅ SPA routing fix with 404.html fallback
- ✅ Concurrency control for deployments