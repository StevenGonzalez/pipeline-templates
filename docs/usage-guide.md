# Usage Guide

This guide provides step-by-step instructions for using the pipeline templates in your projects.

## Table of Contents

1. [GitHub Actions Usage](#github-actions-usage)
2. [Azure Pipelines Usage](#azure-pipelines-usage)
3. [Template Parameters](#template-parameters)
4. [Common Scenarios](#common-scenarios)
5. [Best Practices](#best-practices)

## GitHub Actions Usage

### .NET Template Usage

To use the .NET build and test template in your GitHub Actions workflow:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-test:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
    with:
      dotnet-version: '8.0.x'
      build-configuration: 'Release'
      solution-path: './src/'
      enable-code-coverage: true
```

### Node.js Template Usage

To use the Node.js build and test template:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-test:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/nodejs-build-test.yml@main
    with:
      node-version: '18.x'
      package-manager: 'npm'
      build-script: 'build'
      test-script: 'test'
      run-lint: true
      lint-script: 'lint'
```

## Azure Pipelines Usage

### .NET Template Usage

To use the .NET template in Azure Pipelines:

```yaml
# azure-pipelines.yml
trigger:
- main
- develop

pool:
  vmImage: 'ubuntu-latest'

jobs:
- template: templates/azure-pipelines/dotnet-build-test.yml
  parameters:
    dotnetVersion: '8.0.x'
    buildConfiguration: 'Release'
    solutionPath: '**/*.sln'
    enableCodeCoverage: true
    runTests: true
```

### Node.js Template Usage

To use the Node.js template in Azure Pipelines:

```yaml
# azure-pipelines.yml
trigger:
- main
- develop

pool:
  vmImage: 'ubuntu-latest'

jobs:
- template: templates/azure-pipelines/nodejs-build-test.yml
  parameters:
    nodeVersion: '18.x'
    packageManager: 'npm'
    buildScript: 'build'
    testScript: 'test'
    runLint: true
    lintScript: 'lint'
```

## Template Parameters

### .NET Template Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `dotnet-version` | string | **required** | .NET version (e.g., '6.0.x', '8.0.x') |
| `build-configuration` | string | 'Release' | Build configuration (Debug/Release) |
| `solution-path` | string | './' | Path to solution or project files |
| `test-results-path` | string | 'TestResults/' | Directory for test results |
| `enable-code-coverage` | boolean | true | Enable code coverage collection |
| `run-tests` | boolean | true | Whether to run tests |

### Node.js Template Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `node-version` | string | **required** | Node.js version (e.g., '16.x', '18.x', '20.x') |
| `package-manager` | string | 'npm' | Package manager ('npm' or 'yarn') |
| `build-script` | string | 'build' | Build script name in package.json |
| `test-script` | string | 'test' | Test script name in package.json |
| `lint-script` | string | '' | Lint script name (optional) |
| `working-directory` | string | './' | Working directory for commands |
| `run-build` | boolean | true | Whether to run build step |
| `run-tests` | boolean | true | Whether to run tests |
| `run-lint` | boolean | false | Whether to run linting |

## Common Scenarios

### Multiple .NET Versions

Test your .NET application against multiple versions:

```yaml
# GitHub Actions
jobs:
  test-multiple-versions:
    strategy:
      matrix:
        dotnet-version: ['6.0.x', '7.0.x', '8.0.x']
    uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
    with:
      dotnet-version: ${{ matrix.dotnet-version }}
```

### Node.js with Different Package Managers

```yaml
# GitHub Actions
jobs:
  test-package-managers:
    strategy:
      matrix:
        package-manager: ['npm', 'yarn']
        node-version: ['16.x', '18.x', '20.x']
    uses: StevenGonzalez/pipeline-templates/.github/workflows/nodejs-build-test.yml@main
    with:
      node-version: ${{ matrix.node-version }}
      package-manager: ${{ matrix.package-manager }}
```

### Monorepo with Multiple Projects

For monorepos, specify working directories:

```yaml
# GitHub Actions
jobs:
  frontend:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/nodejs-build-test.yml@main
    with:
      node-version: '18.x'
      working-directory: './frontend'
      
  backend:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
    with:
      dotnet-version: '8.0.x'
      solution-path: './backend/'
```

## Best Practices

### 1. Pin Template Versions

Use specific tags or commit hashes instead of `@main` for production:

```yaml
uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@v1.0.0
```

### 2. Environment-Specific Configurations

Use different configurations for different environments:

```yaml
jobs:
  development:
    if: github.ref == 'refs/heads/develop'
    uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
    with:
      build-configuration: 'Debug'
      
  production:
    if: github.ref == 'refs/heads/main'
    uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
    with:
      build-configuration: 'Release'
      enable-code-coverage: true
```

### 3. Conditional Execution

Use conditions to run templates only when relevant files change:

```yaml
jobs:
  dotnet-changes:
    uses: dorny/paths-filter@v2
    with:
      filters: |
        dotnet:
          - 'src/**/*.cs'
          - 'src/**/*.csproj'
          - 'src/**/*.sln'
          
  build-dotnet:
    needs: dotnet-changes
    if: needs.dotnet-changes.outputs.dotnet == 'true'
    uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
    with:
      dotnet-version: '8.0.x'
```

### 4. Artifact Management

Templates automatically upload artifacts, but you can customize paths:

```yaml
uses: StevenGonzalez/pipeline-templates/.github/workflows/nodejs-build-test.yml@main
with:
  node-version: '18.x'
  build-script: 'build:prod'
  # Artifacts will be uploaded from standard paths: dist/, build/, out/
```

### 5. Secret Management

Templates don't handle secrets directly. Manage them in your calling workflow:

```yaml
jobs:
  build:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/nodejs-build-test.yml@main
    with:
      node-version: '18.x'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```