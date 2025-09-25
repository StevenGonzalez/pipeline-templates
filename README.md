# Pipeline Templates

Modular CI/CD templates for scalable automation. Designed for clarity, reuse, and platform-agnostic workflows—supporting GitHub Actions, Azure Pipelines, and more. Composable, future-proof, and easy to extend across monorepos, microservices, and hybrid setups.

## 🚀 Quick Start

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  dotnet-build:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
    with:
      dotnet-version: '8.0.x'
      build-configuration: 'Release'

  nodejs-build:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/nodejs-build-test.yml@main
    with:
      node-version: '18.x'
      package-manager: 'npm'
```

### Azure Pipelines

```yaml
# azure-pipelines.yml
jobs:
- template: templates/azure-pipelines/dotnet-build-test.yml
  parameters:
    dotnetVersion: '8.0.x'
    buildConfiguration: 'Release'

- template: templates/azure-pipelines/nodejs-build-test.yml
  parameters:
    nodeVersion: '18.x'
    packageManager: 'npm'
```

## 📁 Repository Structure

```
├── templates/
│   ├── github-actions/          # Reusable GitHub Actions workflows
│   │   ├── dotnet-build-test.yml
│   │   └── nodejs-build-test.yml
│   └── azure-pipelines/         # Azure DevOps pipeline templates
│       ├── dotnet-build-test.yml
│       └── nodejs-build-test.yml
├── docs/                        # Comprehensive documentation
│   ├── usage-guide.md
│   ├── customization-guide.md
│   ├── troubleshooting.md
│   └── examples/                # Working examples
└── README.md                    # This file
```

## 🛠 Available Templates

### .NET Templates
- **Cross-platform builds** with configurable .NET versions
- **Automated testing** with code coverage support
- **NuGet package restoration** and dependency caching
- **Flexible solution/project paths** for monorepos
- **Build artifact management** and test result publishing

**Key Parameters:**
- `dotnet-version` (required): .NET SDK version (e.g., '6.0.x', '8.0.x')
- `build-configuration`: Debug or Release (default: Release)
- `solution-path`: Path to .sln or project files
- `enable-code-coverage`: Boolean to enable/disable coverage (default: true)
- `run-tests`: Boolean to run tests (default: true)

### Node.js Templates
- **npm and yarn support** with automatic package manager detection
- **Multi-version Node.js testing** across different environments
- **Configurable build and test scripts** from package.json
- **Dependency caching** for faster builds
- **Lint integration** with configurable linting scripts
- **Monorepo support** with custom working directories

**Key Parameters:**
- `node-version` (required): Node.js version (e.g., '16.x', '18.x', '20.x')
- `package-manager`: 'npm' or 'yarn' (default: npm)
- `build-script`: Build script name (default: build)
- `test-script`: Test script name (default: test)
- `lint-script`: Lint script name (optional)
- `working-directory`: Custom working directory path

## 📖 Documentation

- **[Usage Guide](docs/usage-guide.md)** - Complete guide with examples and best practices
- **[Customization Guide](docs/customization-guide.md)** - How to extend and modify templates
- **[Examples](docs/examples/)** - Real-world usage examples for different scenarios
- **[Troubleshooting](docs/troubleshooting.md)** - Common issues and solutions

## 🎯 Use Cases

### Single Repository
Perfect for standard applications with straightforward CI/CD needs:
```yaml
uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
with:
  dotnet-version: '8.0.x'
```

### Monorepo/Microservices
Handle multiple projects with different configurations:
```yaml
jobs:
  frontend:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/nodejs-build-test.yml@main
    with:
      node-version: '18.x'
      working-directory: './apps/frontend'
      
  backend:
    uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
    with:
      dotnet-version: '8.0.x'
      solution-path: './apps/backend/'
```

### Multi-Platform Testing
Test across different environments and versions:
```yaml
strategy:
  matrix:
    dotnet-version: ['6.0.x', '7.0.x', '8.0.x']
uses: StevenGonzalez/pipeline-templates/.github/workflows/dotnet-build-test.yml@main
with:
  dotnet-version: ${{ matrix.dotnet-version }}
```

## 🔧 Customization

Templates are highly parameterized for flexibility without modification. For advanced customization:

1. **Fork the templates** for organization-specific needs
2. **Extend with composition** by creating wrapper templates
3. **Override specific steps** using conditional parameters
4. **Add custom steps** before/after template execution

See the [Customization Guide](docs/customization-guide.md) for detailed instructions.

## ✨ Features

- **🔀 Platform Agnostic**: Works with GitHub Actions and Azure Pipelines
- **📦 Parameterized**: Extensive configuration options without template modification  
- **🚀 Performance Optimized**: Includes caching and parallel execution where possible
- **🧪 Test Integration**: Built-in support for unit tests, code coverage, and reporting
- **📊 Artifact Management**: Automatic handling of build outputs and test results
- **🔍 Debugging Friendly**: Comprehensive logging and error handling
- **📚 Well Documented**: Extensive documentation with examples and troubleshooting

## 🤝 Contributing

We welcome contributions! Please:

1. Review existing [issues](../../issues) and [pull requests](../../pulls)
2. Follow the established template patterns and documentation style
3. Test your changes across both GitHub Actions and Azure Pipelines
4. Update documentation for any new features or parameters

## 📄 License

This project is licensed under the [MIT License](LICENSE) - see the file for details.

## 🆘 Support

- **Documentation**: Check the [docs](docs/) directory first
- **Issues**: Open an issue for bugs or feature requests
- **Discussions**: Use GitHub Discussions for questions and community support

---

**Made with ❤️ for the developer community**
