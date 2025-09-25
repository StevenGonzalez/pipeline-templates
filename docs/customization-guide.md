# Customization Guide

This guide explains how to customize the pipeline templates to meet your specific requirements.

## Table of Contents

1. [Template Structure](#template-structure)
2. [Adding Custom Parameters](#adding-custom-parameters)
3. [Extending Templates](#extending-templates)
4. [Creating Variations](#creating-variations)
5. [Platform-Specific Customizations](#platform-specific-customizations)
6. [Advanced Scenarios](#advanced-scenarios)

## Template Structure

Understanding the template structure helps you customize effectively:

```
templates/
├── github-actions/
│   ├── dotnet-build-test.yml      # Reusable GitHub Actions workflow
│   └── nodejs-build-test.yml      # Reusable GitHub Actions workflow
└── azure-pipelines/
    ├── dotnet-build-test.yml      # Azure DevOps template
    └── nodejs-build-test.yml      # Azure DevOps template
```

Each template follows this pattern:
1. **Header Comments**: Usage instructions and parameter documentation
2. **Input Parameters**: Configurable values with defaults
3. **Job Definition**: Main pipeline logic
4. **Steps**: Individual tasks with conditional execution

## Adding Custom Parameters

### GitHub Actions Example

To add a custom parameter to the .NET template:

```yaml
on:
  workflow_call:
    inputs:
      # Existing parameters...
      
      # Add your custom parameter
      custom-build-args:
        description: 'Additional build arguments'
        required: false
        type: string
        default: ''
        
      enable-static-analysis:
        description: 'Run static analysis tools'
        required: false
        type: boolean
        default: false

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      # Use the custom parameter in a step
      - name: Build with custom args
        run: dotnet build --configuration ${{ inputs.build-configuration }} ${{ inputs.custom-build-args }}
        
      # Conditional step based on boolean parameter
      - name: Run static analysis
        if: ${{ inputs.enable-static-analysis }}
        run: dotnet run --project tools/analyzer
```

### Azure Pipelines Example

To add custom parameters to an Azure Pipeline template:

```yaml
parameters:
# Existing parameters...

# Add your custom parameters
- name: customBuildArgs
  type: string
  displayName: 'Additional build arguments'
  default: ''
  
- name: enableStaticAnalysis
  type: boolean
  displayName: 'Run static analysis'
  default: false

jobs:
- job: BuildAndTest
  steps:
  # Use the custom parameter
  - task: DotNetCoreCLI@2
    displayName: 'Build with custom args'
    inputs:
      command: 'build'
      arguments: '--configuration $(buildConfiguration) ${{ parameters.customBuildArgs }}'
      
  # Conditional step
  - script: dotnet run --project tools/analyzer
    displayName: 'Run static analysis'
    condition: eq('${{ parameters.enableStaticAnalysis }}', true)
```

## Extending Templates

### Method 1: Template Composition

Create a new template that uses existing ones:

```yaml
# templates/github-actions/dotnet-full-pipeline.yml
name: Full .NET Pipeline

on:
  workflow_call:
    inputs:
      dotnet-version:
        required: true
        type: string
      deploy-environment:
        required: false
        type: string
        default: 'staging'

jobs:
  # Use existing build template
  build-test:
    uses: ./.github/workflows/dotnet-build-test.yml
    with:
      dotnet-version: ${{ inputs.dotnet-version }}
      enable-code-coverage: true
      
  # Add deployment job
  deploy:
    needs: build-test
    runs-on: ubuntu-latest
    if: success()
    steps:
      - name: Deploy to ${{ inputs.deploy-environment }}
        run: echo "Deploying to ${{ inputs.deploy-environment }}"
```

### Method 2: Template Inheritance

Fork the template and add your customizations:

```yaml
# templates/github-actions/dotnet-with-security-scan.yml
# Based on dotnet-build-test.yml with security scanning added

name: .NET Build, Test, and Security Scan

on:
  workflow_call:
    inputs:
      # Include all original parameters
      dotnet-version:
        required: true
        type: string
      # Add security-specific parameters
      security-scan-severity:
        required: false
        type: string
        default: 'medium'

jobs:
  build-test-scan:
    runs-on: ubuntu-latest
    steps:
      # Include all original steps...
      
      # Add security scanning
      - name: Run security scan
        uses: securecodewarrior/github-action-add-sarif@v1
        with:
          sarif-file: security-results.sarif
```

## Creating Variations

### Database Integration Template

Create a variation that includes database testing:

```yaml
# templates/github-actions/dotnet-with-database.yml
name: .NET with Database Testing

on:
  workflow_call:
    inputs:
      dotnet-version:
        required: true
        type: string
      database-type:
        required: false
        type: string
        default: 'postgresql'
      run-integration-tests:
        required: false
        type: boolean
        default: true

jobs:
  test-with-database:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
          
    steps:
      # Standard build steps...
      
      # Database-specific testing
      - name: Run integration tests
        if: ${{ inputs.run-integration-tests }}
        run: dotnet test --filter Category=Integration
        env:
          ConnectionStrings__DefaultConnection: "Host=localhost;Database=testdb;Username=postgres;Password=postgres"
```

### Multi-Platform Template

Create a template that runs on multiple operating systems:

```yaml
# templates/github-actions/nodejs-multiplatform.yml
name: Node.js Multi-Platform

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      test-platforms:
        required: false
        type: string
        default: '["ubuntu-latest", "windows-latest", "macos-latest"]'

jobs:
  test-multiplatform:
    strategy:
      matrix:
        os: ${{ fromJson(inputs.test-platforms) }}
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          
      # Platform-specific commands
      - name: Install dependencies (Windows)
        if: runner.os == 'Windows'
        run: npm ci
        shell: cmd
        
      - name: Install dependencies (Unix)
        if: runner.os != 'Windows'
        run: npm ci
```

## Platform-Specific Customizations

### GitHub Actions Specific Features

Leverage GitHub Actions-specific features:

```yaml
# Add marketplace actions
- name: SonarCloud Scan
  uses: SonarSource/sonarcloud-github-action@master
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

# Use GitHub's cache action
- name: Cache NuGet packages
  uses: actions/cache@v3
  with:
    path: ~/.nuget/packages
    key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
```

### Azure Pipelines Specific Features

Use Azure DevOps-specific capabilities:

```yaml
# Use Azure-specific tasks
- task: SonarCloudPrepare@1
  inputs:
    SonarCloud: 'SonarCloud'
    organization: 'your-org'
    scannerMode: 'MSBuild'

# Publish to Azure Artifacts
- task: NuGetCommand@2
  inputs:
    command: 'push'
    packagesToPush: '$(Build.ArtifactStagingDirectory)/**/*.nupkg'
    nuGetFeedType: 'internal'
    publishVstsFeed: 'your-feed'
```

## Advanced Scenarios

### Dynamic Parameter Generation

Create templates that adapt based on repository content:

```yaml
# Auto-detect project type and configure accordingly
- name: Detect project structure
  id: detect
  run: |
    if [ -f "package.json" ]; then
      echo "project-type=nodejs" >> $GITHUB_OUTPUT
    elif [ -f "*.csproj" ] || [ -f "*.sln" ]; then
      echo "project-type=dotnet" >> $GITHUB_OUTPUT
    fi
    
- name: Build Node.js project
  if: steps.detect.outputs.project-type == 'nodejs'
  uses: ./.github/workflows/nodejs-build-test.yml
  with:
    node-version: '18.x'
    
- name: Build .NET project
  if: steps.detect.outputs.project-type == 'dotnet'
  uses: ./.github/workflows/dotnet-build-test.yml
  with:
    dotnet-version: '8.0.x'
```

### Template Versioning Strategy

Maintain backward compatibility while evolving templates:

```yaml
# v2 template with new features but backward compatibility
name: .NET Build and Test v2

on:
  workflow_call:
    inputs:
      # Keep all v1 parameters for compatibility
      dotnet-version:
        required: true
        type: string
        
      # Add new v2 parameters with safe defaults
      use-central-package-management:
        required: false
        type: boolean
        default: false
        
      # Deprecated parameter (still supported)
      legacy-test-framework:
        required: false
        type: boolean
        default: false
        # Note: This parameter is deprecated, use 'test-framework' instead

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      # Handle deprecated parameters
      - name: Handle legacy parameters
        run: |
          if [ "${{ inputs.legacy-test-framework }}" = "true" ]; then
            echo "⚠️  legacy-test-framework is deprecated"
          fi
```

### Custom Error Handling

Add robust error handling to templates:

```yaml
- name: Build with error handling
  run: |
    set -e  # Exit on error
    
    # Attempt build
    if ! dotnet build --configuration Release; then
      echo "::error::Build failed - checking for common issues"
      
      # Check for missing dependencies
      if ! dotnet list package --outdated; then
        echo "::warning::Some packages may be outdated"
      fi
      
      # Check for target framework issues
      if grep -r "net[0-9]" *.csproj; then
        echo "::warning::Check target framework compatibility"
      fi
      
      exit 1
    fi
```

This customization guide provides a foundation for adapting the templates to your specific needs while maintaining their reusability and clarity.