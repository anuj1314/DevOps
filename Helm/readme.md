# Helm Chart Workflow Documentation

Complete guide for creating, managing, and distributing Helm charts, including chart structure, validation, packaging, and repository management.

## Table of Contents

- [Overview](#overview)
- [Creating a New Helm Chart](#creating-a-new-helm-chart)
- [Chart Validation](#chart-validation)
- [Dependency Management](#dependency-management)
- [Chart Packaging](#chart-packaging)
- [Complete Development Workflow](#complete-development-workflow)
- [Helm Repository Management](#helm-repository-management)
- [Installing Charts](#installing-charts)
- [Alternative Distribution Methods](#alternative-distribution-methods)
- [Best Practices](#best-practices)
- [Command Reference Summary](#command-reference-summary)

## Overview

This documentation covers the complete workflow for creating, managing, and distributing Helm charts. Whether you're new to Helm or looking to improve your chart development process, this guide provides step-by-step instructions and best practices.

## Creating a New Helm Chart

Use the `helm create` command to generate a new chart with default structure and files:

```bash
helm create <chart-name>
```

### Generated Chart Structure

The command creates the following directory structure:

```
test-activity-project/
├── Chart.yaml              # Chart metadata and version info
├── values.yaml             # Default configuration values
├── charts/                 # Chart dependencies (sub-charts)
├── templates/              # Kubernetes manifest templates
│   ├── deployment.yaml     # Deployment resource template
│   ├── service.yaml        # Service resource template
│   ├── hpa.yaml           # Horizontal Pod Autoscaler template
│   ├── ingress.yaml       # Ingress resource template
│   ├── _helpers.tpl       # Template helper functions
│   └── tests/
│       └── test-connection.yaml
└── .helmignore            # Files to exclude from packaging
```

> **Note:** The `charts/` folder stores chart dependencies (sub-charts). If empty, it means your chart has no dependencies yet.

## Chart Validation

### Syntax and Structure Validation

The `helm lint` command validates your chart for syntax errors, template issues, and configuration problems without deploying anything:

```bash
helm lint <chart>
```

**Use cases:**
- Checks YAML syntax and structure
- Catches template rendering errors
- Validates chart configuration
- Does not deploy to Kubernetes
- Best used before installation/upgrade or in CI/CD pipelines

### Template Rendering Preview

The `helm template` command renders your Helm templates into plain Kubernetes YAML for inspection:

```bash
helm template <release> -f values.yaml
```

**Key features:**
- Renders templates using provided values file
- Outputs plain Kubernetes YAML
- Does not communicate with Kubernetes cluster
- Useful for debugging template logic
- Enables code review and diff comparison
- Supports environment-specific value files

## Dependency Management

Helm provides two commands for managing chart dependencies, each serving different purposes in the development and deployment lifecycle.

### helm dependency update

```bash
helm dependency update
```

**Purpose:**
- Reads dependencies from `Chart.yaml`
- Downloads the latest matching versions based on version constraints
- Updates `Chart.lock` with resolved versions
- Stores dependency charts in `charts/` directory

**Best for:** Development environments where you want to fetch the latest compatible versions of dependencies

### helm dependency build

```bash
helm dependency build
```

**Purpose:**
- Uses only versions pinned in `Chart.lock` (ignoring `Chart.yaml`)
- Rebuilds `charts/` directory with exact locked versions
- Does not change or update version numbers
- Ensures reproducible builds

**Best for:** CI/CD pipelines and production deployments where reproducibility and stability are critical

> 💡 **Rule of Thumb:** Use `helm dependency update` to change dependencies, `helm dependency build` to reproduce them exactly.

## Chart Packaging

The `helm package` command creates a distributable archive of your Helm chart:

```bash
helm package .
```

**What it does:**
- Packages the chart in the current directory
- Creates a versioned `.tgz` archive file
- Uses chart name and version from `Chart.yaml`
- Does not deploy the chart
- Does not communicate with Kubernetes

**Output format:** `<chart-name>-<version>.tgz`

## Complete Development Workflow

Follow this sequence for a typical Helm chart development workflow:

```
1. helm create <chart-name>
   ↓ Create new chart structure
   
2. helm lint <chart>
   ↓ Validate syntax and structure
   
3. helm template <release> -f values.yaml
   ↓ Preview rendered manifests
   
4. helm package .
   ↓ Create distributable archive
   
5. helm install | helm upgrade
   ↓ Deploy to Kubernetes
```

## Helm Repository Management

### Creating a Repository Index

The `helm repo index` command generates an `index.yaml` file that catalogs all charts in your repository:

```bash
helm repo index .
```

The `index.yaml` file contains:
- List of all available charts
- Chart versions and metadata
- Download URLs for each chart package
- Chart dependencies and requirements

> **Think of `index.yaml` as:** The catalog or table of contents for your Helm repository

### Publishing to GitHub Pages

To serve your Helm charts via GitHub Pages, follow these steps:

**1. Package your charts**

```bash
helm package .
```

**2. Generate repository index**

```bash
helm repo index .
```

**3. Push to GitHub repository**

```bash
git add .
git commit -m "Add Helm charts"
git push
```

**4. Enable GitHub Pages in repository settings**

> **Note:** Once GitHub Pages is enabled, your repository will serve the `index.yaml` and `.tgz` chart files at the configured URL.

### Adding a Remote Repository

To add your published Helm repository for use:

```bash
helm repo add my-repo https://anuj1314.github.io/helm-test-repo-private
```

> **Important:** This command will only work after you have published your changes to GitHub and enabled GitHub Pages in the repository settings.

### Verifying Repository

Check all configured repositories:

```bash
helm repo list
```

### Updating Repository Cache

Fetch the latest chart information from all configured repositories:

```bash
helm repo update
```

**Expected output:**

```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "my-repo" chart repository
Update Complete. ⎈Happy Helming!⎈
```

### Searching Repository Charts

View all available charts in a repository:

```bash
helm search repo my-repo
```

**Example output:**

```
NAME                         CHART VERSION  APP VERSION  DESCRIPTION
my-repo/redis                19.5.1         7.2.5        Redis® is an open source...
my-repo/test-activity-project 0.1.0         1.16.0       A Helm chart for Kubernetes
```

## Installing Charts

Install a chart from your repository:

```bash
helm install test-activity my-repo/test-activity-project
```

**Syntax:** `helm install <release-name> <repository>/<chart-name>`

## Alternative Distribution Methods

### Limitations of Private GitHub Organizations

**Challenge:** GitHub Pages may not be available for private organization repositories, limiting your ability to serve Helm charts publicly.

### Alternative Distribution Options

When GitHub Pages is not available, consider these alternatives:

- **ChartMuseum** - Self-hosted Helm chart repository server
- **Artifactory / Nexus** - Artifact repository managers with Helm support
- **Amazon S3 / Azure Blob Storage** - Cloud storage with HTTP access
- **Harbor** - Cloud-native registry for container images and Helm charts
- **GitLab Package Registry** - Built-in Helm chart registry
- **Direct distribution** - Share `.tgz` files via internal file servers

## Best Practices

- **Version control** - Always commit `Chart.lock` to ensure reproducible builds
- **Validation** - Run `helm lint` before every commit and in CI/CD pipelines
- **Testing** - Use `helm template` to preview changes before deployment
- **Dependencies** - Use `helm dependency build` in production for stability
- **Documentation** - Maintain clear README files with installation instructions
- **Semantic versioning** - Follow SemVer for chart versions in `Chart.yaml`
- **Values documentation** - Comment all configurable values in `values.yaml`
- **Security** - Scan charts for vulnerabilities before distribution

## Command Reference Summary

| Command | Purpose |
|---------|---------|
| `helm create <chart>` | Create new chart with default structure |
| `helm lint <chart>` | Validate chart syntax and structure |
| `helm template <release>` | Render templates to YAML |
| `helm dependency update` | Download latest dependencies |
| `helm dependency build` | Build with locked versions |
| `helm package .` | Create .tgz archive |
| `helm repo index .` | Generate repository index |
| `helm repo add <n> <url>` | Add remote repository |
| `helm repo list` | List configured repositories |
| `helm repo update` | Update repository cache |
| `helm search repo <n>` | Search charts in repository |
| `helm install <n> <chart>` | Install chart to cluster |
| `helm upgrade <n> <chart>` | Upgrade existing release |

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

[Specify your license here]

## Support

For questions or issues, please open an issue in this repository.