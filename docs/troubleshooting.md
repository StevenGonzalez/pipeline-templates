# Troubleshooting Guide

This guide helps you resolve common issues when using the pipeline templates.

## Table of Contents

1. [Common Issues](#common-issues)
2. [GitHub Actions Specific Issues](#github-actions-specific-issues)
3. [Azure Pipelines Specific Issues](#azure-pipelines-specific-issues)
4. [.NET Template Issues](#net-template-issues)
5. [Node.js Template Issues](#nodejs-template-issues)
6. [Getting Help](#getting-help)

## Common Issues

### Template Not Found

**Problem**: Error message like "Template not found" or "Unable to resolve template"

**Solutions**:
1. **Check the template path**: Ensure you're using the correct path to the template
   ```yaml
   # ✅ Correct
   uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
   
   # ❌ Incorrect
   uses: StevenGonzalez/pipeline-templates/templates/github-actions/dotnet-build-test.yml@main
   ```

2. **Verify the branch/tag**: Make sure the branch or tag exists
   ```yaml
   # Use specific version for stability
   uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@v1.0.0
   ```

3. **Check repository access**: Ensure the template repository is public or you have access

### Parameter Validation Errors

**Problem**: "Invalid input" or parameter validation errors

**Solutions**:
1. **Check required parameters**: Ensure all required parameters are provided
   ```yaml
   with:
     dotnet-version: '8.0.x'  # This is required
   ```

2. **Verify parameter types**: Check that boolean values are not quoted
   ```yaml
   with:
     run-tests: true          # ✅ Correct
     run-tests: 'true'        # ❌ Incorrect
   ```

3. **Check parameter names**: Ensure parameter names match exactly (case-sensitive)

### Permission Issues

**Problem**: "Permission denied" or "Access denied" errors

**Solutions**:
1. **Repository permissions**: Ensure the calling repository has permission to use the template
2. **Token permissions**: Check that the GitHub token has necessary permissions
3. **Organization settings**: Verify organization policies allow reusable workflows

## GitHub Actions Specific Issues

### Workflow Not Triggering

**Problem**: Reusable workflow doesn't start when called

**Solutions**:
1. **Check syntax**: Validate YAML syntax using online validators
2. **Event triggers**: Ensure the calling workflow has proper triggers
   ```yaml
   on:
     push:
       branches: [ main ]
     pull_request:
       branches: [ main ]
   ```

3. **File location**: Reusable workflows must be in `.github/workflows/` directory

### Job Dependencies

**Problem**: Jobs running in wrong order or not waiting for dependencies

**Solutions**:
1. **Use `needs` keyword**: Specify job dependencies explicitly
   ```yaml
   jobs:
     build:
       uses: ./.github/workflows/build-template.yml
       
     deploy:
       needs: build  # Wait for build to complete
       runs-on: ubuntu-latest
   ```

### Secrets Not Available

**Problem**: Template can't access secrets from calling workflow

**Solutions**:
1. **Pass secrets explicitly**: Reusable workflows don't inherit secrets automatically
   ```yaml
   jobs:
     build:
       uses: ./.github/workflows/template.yml
       secrets:
         MY_SECRET: ${{ secrets.MY_SECRET }}
   ```

2. **Use `secrets: inherit`**: Pass all secrets (use with caution)
   ```yaml
   jobs:
     build:
       uses: ./.github/workflows/template.yml
       secrets: inherit
   ```

## Azure Pipelines Specific Issues

### Template Path Resolution

**Problem**: "Template not found" in Azure Pipelines

**Solutions**:
1. **Use relative paths**: Reference templates relative to the calling pipeline
   ```yaml
   # If template is in same repository
   - template: templates/azure-pipelines/dotnet-build-test.yml
   
   # If template is in different repository
   - template: templates/dotnet-build-test.yml@template-repo
   ```

2. **Repository resources**: Define external template repositories
   ```yaml
   resources:
     repositories:
     - repository: templates
       type: git
       name: StevenGonzalez/pipeline-templates
   
   jobs:
   - template: templates/azure-pipelines/dotnet-build-test.yml@templates
   ```

### Parameter Type Conversion

**Problem**: Parameter type errors in Azure Pipelines

**Solutions**:
1. **String to boolean conversion**: Use expressions for type conversion
   ```yaml
   # Convert string to boolean
   runTests: ${{ eq(parameters.runTests, 'true') }}
   ```

2. **Default values**: Ensure parameter defaults match expected types
   ```yaml
   parameters:
   - name: runTests
     type: boolean
     default: true  # Not 'true'
   ```

### Variable Scope Issues

**Problem**: Variables not available across jobs or stages

**Solutions**:
1. **Use template parameters**: Pass values through parameters instead of variables
2. **Output variables**: Use job outputs to pass data between jobs
   ```yaml
   jobs:
   - job: SetVariable
     steps:
     - script: echo "##vso[task.setvariable variable=myVar;isOutput=true]myValue"
       name: setVar
   
   - job: UseVariable
     dependsOn: SetVariable
     variables:
       myVar: $[ dependencies.SetVariable.outputs['setVar.myVar'] ]
   ```

## .NET Template Issues

### SDK Version Not Found

**Problem**: ".NET SDK version not found" or installation failures

**Solutions**:
1. **Use LTS versions**: Stick to Long Term Support versions when possible
   ```yaml
   with:
     dotnet-version: '8.0.x'  # LTS version
   ```

2. **Check version availability**: Verify the version exists on the runner
3. **Use preview versions carefully**: Include preview flag if needed
   ```yaml
   - name: Setup .NET
     uses: actions/setup-dotnet@v4
     with:
       dotnet-version: '9.0.x'
       include-prerelease: true
   ```

### Build Failures

**Problem**: Build fails with compilation errors

**Solutions**:
1. **Check target framework**: Ensure compatibility between .NET version and target framework
2. **Restore packages first**: Template handles this, but verify NuGet sources are accessible
3. **Build configuration**: Try different build configurations
   ```yaml
   with:
     build-configuration: 'Debug'  # Instead of Release
   ```

### Test Discovery Issues

**Problem**: "No tests found" or test discovery failures

**Solutions**:
1. **Check test project structure**: Ensure test projects follow naming conventions
   ```
   ✅ MyApp.Tests.csproj
   ✅ MyApp.UnitTests.csproj
   ❌ Tests.csproj (might not be discovered)
   ```

2. **Verify test framework**: Ensure compatible test framework (xUnit, NUnit, MSTest)
3. **Solution path**: Check that solution path includes test projects
   ```yaml
   with:
     solution-path: './src/'  # Make sure this includes test projects
   ```

## Node.js Template Issues

### Package Manager Issues

**Problem**: Package installation failures or wrong package manager used

**Solutions**:
1. **Lock file mismatch**: Ensure lock file matches package manager
   ```
   ✅ npm + package-lock.json
   ✅ yarn + yarn.lock
   ❌ npm + yarn.lock (mismatch)
   ```

2. **Cache issues**: Clear cache if dependencies are stale
3. **Registry access**: Check if private registries are properly configured

### Script Not Found

**Problem**: "Script not found" errors

**Solutions**:
1. **Check package.json**: Verify script names exist in package.json
   ```json
   {
     "scripts": {
       "build": "webpack --mode production",
       "test": "jest"
     }
   }
   ```

2. **Case sensitivity**: Script names are case-sensitive
3. **Use conditional execution**: Template handles missing scripts gracefully

### Node Version Compatibility

**Problem**: Node.js version incompatibility with dependencies

**Solutions**:
1. **Check engine requirements**: Review package.json engines field
   ```json
   {
     "engines": {
       "node": ">=16.0.0",
       "npm": ">=8.0.0"
     }
   }
   ```

2. **Use compatible versions**: Match Node.js version to your dependencies
3. **Update dependencies**: Consider updating packages for newer Node.js versions

### Build Artifact Issues

**Problem**: Build artifacts not found or uploaded incorrectly

**Solutions**:
1. **Check build output**: Verify your build script creates expected directories
   ```
   Common build directories:
   - dist/
   - build/
   - out/
   ```

2. **Custom build paths**: If using custom paths, they won't be auto-uploaded
3. **Build script configuration**: Ensure build script outputs to standard directories

## Getting Help

### Debug Mode

Enable debug logging for more detailed information:

**GitHub Actions**:
```yaml
env:
  ACTIONS_STEP_DEBUG: true
```

**Azure Pipelines**:
```yaml
variables:
  system.debug: true
```

### Template Issues

1. **Check template source**: Review the template source code for insights
2. **Minimal reproduction**: Create a minimal example that demonstrates the issue
3. **Version pinning**: Use specific template versions to avoid breaking changes

### Community Support

1. **Repository Issues**: Open an issue in the template repository
2. **Documentation**: Check if newer documentation addresses your issue
3. **Examples**: Look at the examples directory for working configurations

### Creating Bug Reports

When reporting issues, include:

1. **Template version**: Specify exact commit hash or tag
2. **Platform**: GitHub Actions or Azure Pipelines
3. **Configuration**: Your complete workflow/pipeline configuration
4. **Error logs**: Full error messages and logs
5. **Expected behavior**: What you expected to happen
6. **Actual behavior**: What actually happened

**Example Bug Report Template**:
```markdown
## Bug Description
Brief description of the issue

## Template Information
- Template: dotnet-build-test.yml
- Version/Commit: abc123def
- Platform: GitHub Actions

## Configuration
```yaml
# Your workflow configuration here
```

## Error Logs
```
# Full error logs here
```

## Expected vs Actual Behavior
- Expected: Should build successfully
- Actual: Build fails with compilation error
```