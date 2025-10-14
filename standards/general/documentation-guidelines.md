# Documentation Guidelines

## Overview

All project documentation **MUST** be written in Markdown and organized using **MkDocs**. This ensures consistency, maintainability, and professional presentation of documentation across all projects.

## Mandatory: MkDocs Documentation Framework

### Why MkDocs?

- **Markdown-native**: Write documentation in simple, version-controlled Markdown files
- **Static site generation**: Build fast, searchable, and beautiful documentation sites
- **Built-in search**: Full-text search across all documentation out of the box
- **Material theme**: Modern, responsive design with excellent developer experience
- **CI/CD friendly**: Automated deployment to GitHub Pages or cloud platforms
- **Navigation**: Automatic table of contents and navigation structure

## Required Folder Structure

Every project **MUST** follow this documentation structure:

```
/
├── docs/                          # MkDocs documentation root
│   ├── index.md                   # Landing page (required)
│   ├── getting-started/
│   │   ├── installation.md
│   │   ├── quick-start.md
│   │   └── configuration.md
│   ├── architecture/
│   │   ├── overview.md
│   │   ├── system-design.md
│   │   └── data-flow.md
│   ├── api/
│   │   ├── rest-api.md
│   │   └── websocket-api.md
│   ├── guides/
│   │   ├── development.md
│   │   ├── deployment.md
│   │   └── troubleshooting.md
│   └── reference/
│       ├── configuration.md
│       └── environment-variables.md
├── specs/                         # Product specifications
│   ├── adr/                       # Architecture Decision Records (MADRs)
│   │   ├── 0001-use-mkdocs.md
│   │   └── 0002-adopt-material-theme.md
│   ├── journal/                   # Engineering journal entries
│   └── product-specs/             # Product requirement documents
├── mkdocs.yml                     # MkDocs configuration (required)
└── README.md                      # Project overview (GitHub/repo view)
```

## Required Tools & Installation

### 1. Python & MkDocs

Install MkDocs and required plugins:

```bash
# Install MkDocs
pip install mkdocs

# Install Material theme (required)
pip install mkdocs-material

# Install recommended plugins
pip install mkdocs-git-revision-date-localized-plugin
pip install mkdocs-minify-plugin
pip install mkdocs-redirects
```

### 2. Required `mkdocs.yml` Configuration

Create `mkdocs.yml` at the repository root with this **minimum** configuration:

```yaml
site_name: Your Project Name
site_description: Brief description of your project
site_author: Your Team Name
repo_url: https://github.com/your-org/your-repo
repo_name: your-org/your-repo

theme:
  name: material
  palette:
    # Light mode
    - scheme: default
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    # Dark mode
    - scheme: slate
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
  features:
    - navigation.instant
    - navigation.tracking
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - navigation.top
    - search.suggest
    - search.highlight
    - content.code.copy
    - content.code.annotate

plugins:
  - search
  - git-revision-date-localized:
      enable_creation_date: true
  - minify:
      minify_html: true

markdown_extensions:
  - admonition
  - pymdownx.details
  - pymdownx.superfences
  - pymdownx.tabbed:
      alternate_style: true
  - pymdownx.highlight:
      anchor_linenums: true
  - pymdownx.inlinehilite
  - pymdownx.snippets
  - tables
  - attr_list
  - md_in_html
  - toc:
      permalink: true

nav:
  - Home: index.md
  - Getting Started:
      - Installation: getting-started/installation.md
      - Quick Start: getting-started/quick-start.md
      - Configuration: getting-started/configuration.md
  - Architecture:
      - Overview: architecture/overview.md
      - System Design: architecture/system-design.md
      - Data Flow: architecture/data-flow.md
  - API Reference:
      - REST API: api/rest-api.md
      - WebSocket API: api/websocket-api.md
  - Guides:
      - Development: guides/development.md
      - Deployment: guides/deployment.md
      - Troubleshooting: guides/troubleshooting.md
  - Reference:
      - Configuration: reference/configuration.md
      - Environment Variables: reference/environment-variables.md
```

## Documentation Standards

### 1. Markdown Files

- **All documentation MUST be in Markdown** (`.md` files)
- Use clear, descriptive filenames in kebab-case: `getting-started.md`, `api-reference.md`
- Include frontmatter metadata when needed (title, description, tags)
- Use proper heading hierarchy (H1 → H2 → H3, no skipping levels)

### 2. Required Documentation Sections

Every project **MUST** include these core documents:

1. **`docs/index.md`**: Landing page with project overview, features, and quick links
2. **`docs/getting-started/installation.md`**: Setup and installation instructions
3. **`docs/getting-started/quick-start.md`**: 5-minute tutorial to get running
4. **`docs/architecture/overview.md`**: High-level architecture explanation
5. **`docs/guides/development.md`**: Developer workflow, commands, and best practices
6. **`docs/guides/deployment.md`**: Deployment procedures and CI/CD setup

### 3. Architecture Decision Records (ADRs/MADRs)

- Store in `/specs/adr/` directory
- Use sequential numbering: `0001-decision-title.md`, `0002-next-decision.md`
- Follow MADR format:
  - Status (Proposed, Accepted, Deprecated, Superseded)
  - Context and Problem Statement
  - Decision Drivers
  - Considered Options
  - Decision Outcome
  - Consequences (Positive and Negative)

**MADR Template:**

```markdown
# [Number]. [Title]

Date: YYYY-MM-DD

## Status

Accepted | Proposed | Deprecated | Superseded by [0005-new-decision.md]

## Context and Problem Statement

[Describe the context and problem statement, e.g., in free form using two to three sentences.]

## Decision Drivers

* [driver 1, e.g., a force, facing concern, ...]
* [driver 2, e.g., a force, facing concern, ...]

## Considered Options

* [option 1]
* [option 2]
* [option 3]

## Decision Outcome

Chosen option: "[option 1]", because [justification].

### Consequences

**Positive:**
* [e.g., improvement of quality attribute satisfaction, follow-up decisions required, ...]

**Negative:**
* [e.g., compromising quality attribute, follow-up decisions required, ...]
```

## Local Development

### Running MkDocs Locally

```bash
# Serve documentation locally with live reload
mkdocs serve

# Open browser at http://127.0.0.1:8000
```

### Building Documentation

```bash
# Build static site to /site directory
mkdocs build

# Build with strict mode (fail on warnings)
mkdocs build --strict
```

## CI/CD Integration

### GitHub Actions Workflow

Add this workflow to `.github/workflows/docs.yml`:

```yaml
name: Deploy Documentation

on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'
      - 'mkdocs.yml'
  workflow_dispatch:

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      
      - name: Install dependencies
        run: |
          pip install mkdocs-material
          pip install mkdocs-git-revision-date-localized-plugin
          pip install mkdocs-minify-plugin
      
      - name: Build and deploy
        run: mkdocs gh-deploy --force
```

### Azure Static Web Apps (Alternative)

For Azure deployments, build during CI and deploy to Azure Static Web Apps:

```yaml
- name: Build MkDocs
  run: mkdocs build

- name: Deploy to Azure Static Web Apps
  uses: Azure/static-web-apps-deploy@v1
  with:
    app_location: "site"
    output_location: ""
```

## Documentation Best Practices

### 1. Keep Documentation Updated

- **Update in place**: Modify existing documentation files directly—never create separate summaries
- Documentation changes must be part of the same PR as code changes
- Use git hooks to remind developers to update docs when changing APIs or features

### 2. Writing Style

- Use clear, concise language
- Write in present tense ("The API returns..." not "The API will return...")
- Use second person ("You can configure..." not "One can configure...")
- Include code examples for all technical concepts
- Add diagrams using Mermaid for architecture and flow documentation

### 3. Code Examples

Use fenced code blocks with language syntax highlighting:

````markdown
```python
# Example Python code
def hello_world():
    return "Hello, World!"
```

```typescript
// Example TypeScript code
const greeting: string = "Hello, World!";
```
````

### 4. Admonitions

Use Material theme admonitions for important information:

```markdown
!!! note
    This is a note with additional information.

!!! warning
    This is a warning about potential issues.

!!! danger
    This is critical information about breaking changes.

!!! tip
    This is a helpful tip for users.
```

## Quality Checklist

Before merging any documentation changes, verify:

- ✅ `mkdocs serve` runs without errors or warnings
- ✅ `mkdocs build --strict` passes successfully
- ✅ All internal links resolve correctly
- ✅ Navigation structure is logical and complete
- ✅ Code examples are tested and accurate
- ✅ Spelling and grammar are correct
- ✅ Images and diagrams are optimized and display properly
- ✅ ADRs are numbered sequentially and follow MADR format

## Enforcement

- **PR requirement**: All PRs affecting functionality must include documentation updates
- **CI check**: MkDocs build must pass in CI pipeline before merge
- **Code review**: Reviewers must verify documentation completeness and accuracy
- **Automated checks**: Use pre-commit hooks to validate Markdown linting and MkDocs build

---

## Quick Start Summary

**For new projects:**

1. Install MkDocs: `pip install mkdocs mkdocs-material`
2. Create folder structure: `/docs`, `/specs/adr`, `/specs/journal`
3. Create `mkdocs.yml` with Material theme configuration
4. Write `docs/index.md` as the landing page
5. Run `mkdocs serve` to preview locally
6. Set up GitHub Actions workflow for automatic deployment
7. Commit and push—documentation builds and deploys automatically

**Remember**: Documentation is code. Treat it with the same rigor as your implementation.
