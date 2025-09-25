# Pipeline Templates Documentation

This directory contains comprehensive documentation for using and customizing the pipeline templates provided in this repository.

## Documentation Structure

- **[usage-guide.md](usage-guide.md)** - Complete guide on how to use the templates in your projects
- **[customization-guide.md](customization-guide.md)** - Instructions for customizing templates to fit your needs
- **[examples/](examples/)** - Working examples showing template usage in real projects
- **[troubleshooting.md](troubleshooting.md)** - Common issues and solutions

## Quick Start

1. **Choose Your Platform**: Select either GitHub Actions or Azure Pipelines templates
2. **Pick Your Technology**: Use .NET or Node.js templates as starting points
3. **Configure Parameters**: Customize the template parameters for your project
4. **Test & Deploy**: Validate your pipeline configuration and deploy

## Template Categories

### GitHub Actions Templates
- `dotnet-build-test.yml` - .NET build and test workflow
- `nodejs-build-test.yml` - Node.js build and test workflow

### Azure Pipelines Templates  
- `dotnet-build-test.yml` - .NET build and test pipeline
- `nodejs-build-test.yml` - Node.js build and test pipeline

## Contributing

If you find issues with the templates or have suggestions for improvements, please:

1. Check the [troubleshooting guide](troubleshooting.md) first
2. Review existing issues in the repository
3. Open a new issue with detailed information about your use case
4. Consider contributing improvements via pull requests

## Template Philosophy

These templates are designed with the following principles:

- **Modularity**: Each template focuses on a specific technology stack
- **Reusability**: Templates can be used across multiple projects with minimal changes
- **Flexibility**: Extensive parameterization allows customization without template modification
- **Best Practices**: Templates incorporate industry-standard CI/CD practices
- **Platform Agnostic**: Similar functionality across different CI/CD platforms