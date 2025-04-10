# GitHub Custom Actions with Dependency Caching

This repository demonstrates how to implement custom GitHub Actions with dependency caching to optimize CI/CD workflows for JavaScript/React applications.

## Project Overview

This project showcases:
- Creating custom GitHub Actions
- Implementing efficient dependency caching
- Setting up a complete CI/CD pipeline with linting, testing, building, and deployment stages

## Project Structure

```
GH-ACTIONS-CUSTOM-ACTIONS/
├── .github/
│   ├── actions/
│   │   └── cached-deps/
│   │       └── action.yml         # Custom action for dependency caching
│   └── workflows/
│       └── deploy.yml             # Main workflow configuration
├── public/                        # Static assets
├── src/
│   ├── assets/
│   │   └── images/                # Image assets
│   ├── components/                # React components
│   │   ├── HelpArea.css
│   │   ├── HelpArea.jsx
│   │   ├── HelpBox.css
│   │   ├── HelpBox.jsx
│   │   ├── MainContent.jsx
│   │   └── MainContent.test.jsx   # Component tests
│   ├── test/                      # Test utilities
│   ├── App.jsx                    # Main application component
│   ├── index.css                  # Global styles
│   └── main.jsx                   # Application entry point
├── .eslintrc.json                 # ESLint configuration
├── .gitignore                     # Git ignore rules
├── index.html                     # HTML entry point
├── package-lock.json              # Dependency lock file
├── package.json                   # Project configuration
├── test.json                      # Test results output
└── vite.config.js                 # Vite bundler configuration

```

## Custom Action: Cached Dependencies

The project features a custom GitHub Action located at `.github/actions/cached-deps/action.yml` that optimizes CI/CD workflows by caching dependencies:

```yaml
name: 'Get Cache & Dependencies'
description: 'Get dependencies and cache them.'
runs:
  using: 'composite'
  steps:
    - name: Cache dependencies
      id: cache
      uses: actions/cache@v4
      with:
        path: node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    
    - name: Install dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
      shell: bash
```

This custom action:
1. Attempts to restore a cached version of `node_modules` based on the hash of `package-lock.json`
2. Installs dependencies only if no cache is found
3. Saves time in subsequent workflow runs by reusing cached dependencies

## Main Workflow: Deploy

The main workflow (`.github/workflows/deploy.yml`) demonstrates a complete CI/CD pipeline:

```yaml
name: Deployment
on:
  push:
    branches:
      - main
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v4
      - name: Load & cache dependencies
        uses: ./.github/actions/cached-deps
      - name: Lint code
        run: npm run lint
  
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v4
      - name: Load & cache dependencies
        uses: ./.github/actions/cached-deps
      - name: Test code
        id: run-tests
        run: npm run test
      - name: Upload test report
        if: failure() && steps.run-tests.outcome == 'failure'
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: test.json
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v4
      - name: Load & cache dependencies
        uses: ./.github/actions/cached-deps
      - name: Build website
        run: npm run build
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: dist
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v4
      - name: Get build artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist-files
          path: ./dist
      - name: Output contents
        run: ls
      - name: Deploy site
        run: echo "Deploying..."
```

## Workflow Features

This workflow includes several key features:

1. **Sequential Job Execution**: 
   - Lint → Test → Build → Deploy
   - Each job depends on the successful completion of previous jobs

2. **Dependency Caching**:
   - Uses custom action to cache and restore `node_modules` 
   - Significantly speeds up workflow execution

3. **Artifact Management**:
   - Uploads test reports if tests fail
   - Passes build artifacts between jobs

4. **Environment Consistency**:
   - Uses `ubuntu-latest` for all runners
   - Ensures compatibility across pipeline stages

## Usage

To use this repository as a template:

1. Clone the repository:
   ```
   git clone https://github.com/PasinduWaidyarathna/gh-actions-custom-actions.git
   cd gh-actions-custom-actions
   ```

2. Install dependencies:
   ```
   npm install
   ```

3. Make changes to the code and push to GitHub:
   ```
   git add .
   git commit -m "Your commit message"
   git push
   ```

4. Watch the GitHub Actions workflow run in the "Actions" tab of your repository.

## Benefits of Custom Actions with Caching

- **Faster Workflows**: Cached dependencies significantly reduce execution time
- **Reusability**: Custom actions can be reused across multiple workflows
- **Maintainability**: Centralized dependency management makes updates easier
- **Cost Efficiency**: Reduced compute time means lower GitHub Actions costs

## Learning Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Creating Custom GitHub Actions](https://docs.github.com/en/actions/creating-actions)
- [Caching Dependencies](https://docs.github.com/en/actions/guides/caching-dependencies-to-speed-up-workflows)
