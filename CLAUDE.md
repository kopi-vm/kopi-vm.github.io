# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the documentation website for Kopi, a fast JDK version management tool written in Rust. The site is built with MkDocs Material and hosted on GitHub Pages at <https://kopi-vm.github.io/>.

The main Kopi application source code is located at `/workspaces/kopi-workspace/kopi/`.

## Key Commands

### Documentation Development

- `mkdocs serve` - Start local development server (default: <http://127.0.0.1:8000>)
- `mkdocs build` - Build static site to `site/` directory
- `mkdocs serve --dev-addr 0.0.0.0:8000` - Serve on all interfaces

### Metadata Management

The site hosts pre-generated JDK metadata files that are automatically updated daily via GitHub Actions:

- Manual update: `kopi-metadata-gen update --input docs/metadata --output docs/metadata`
- Metadata location: `docs/metadata/` (JSON files organized by platform)

## Architecture

### Directory Structure

```bash
kopi-vm.github.io/
├── mkdocs.yml              # MkDocs configuration with Material theme
├── docs/                   # Documentation content
│   ├── index.md           # Homepage
│   ├── assets/            # Logos and images
│   ├── getting-started/   # Installation and quickstart guides
│   ├── guide/             # User guides for managing JDKs
│   ├── concepts/          # Architecture and technical concepts
│   ├── reference/         # Command and API reference
│   ├── metadata/          # Pre-generated JDK metadata (auto-updated)
│   ├── troubleshooting.md # Common issues and solutions
│   └── faq.md            # Frequently asked questions
└── .github/workflows/     # GitHub Actions for automated updates
```

### Key Components

**Documentation System:**

- Static site generator: MkDocs with Material theme
- Markdown content with Mermaid diagram support
- Automatic search indexing and offline support
- Responsive navigation with tabs and sections

**Metadata System:**

- Pre-generated JSON files for each platform/vendor combination
- Organized by: `<os>-<arch>-<libc>/<vendor>.json`
- Index file at `metadata/index.json` listing all platforms
- Updated daily via GitHub Actions cron job
- Serves as primary metadata source for Kopi CLI performance

**GitHub Actions Workflow:**

- File: `.github/workflows/update-metadata.yml`
- Runs daily at midnight UTC
- Uses `kopi-metadata-gen` binary from main Kopi project
- Automatically commits updated metadata files

## Content Guidelines

### Documentation Structure

- **Getting Started**: Installation methods, quickstart guide, first steps
- **Guide**: Practical guides for version management, projects, shell integration
- **Concepts**: Technical explanations of shims, metadata, caching, architecture
- **Reference**: Complete command documentation, configuration options, environment variables

### Writing Style

- Clear, concise technical documentation
- Use code examples for all commands
- Include platform-specific notes where relevant
- Link between related pages for better navigation

### Platform-Specific Content

Document differences between:

- macOS (Intel & Apple Silicon)
- Linux (various distributions)
- Windows (Package Manager & Scoop)

## Documentation Testing

### Linting and Formatting

After updating documentation, always lint and format the files:

1. **Format the documentation files:**

   ```bash
   npm run format
   ```

   This runs both:
   - `markdownlint-cli2 --fix` to fix Markdown formatting issues
   - `prettier --write` to format all files consistently

2. **Lint the documentation to check for issues:**

   ```bash
   npm run lint
   ```

   This runs both:
   - `markdownlint-cli2` to check Markdown syntax and style
   - `prettier --check` to verify formatting consistency

3. **Common linting errors and solutions:**
   - **MD001-MD048**: Various Markdown style violations (e.g., heading levels, list formatting)
   - **Line length violations**: Lines exceeding 120 characters (configured in package.json)
   - **Trailing whitespace**: Remove spaces at end of lines
   - **Inconsistent indentation**: Use 2 spaces for indentation (configured in prettier)
   - **Missing newline at EOF**: Ensure files end with a newline

4. **If linting errors occur:**
   - Run `npm run format` first to auto-fix most issues
   - For remaining errors, check the specific error codes and fix manually
   - Re-run `npm run lint` to verify all issues are resolved

### Build Validation

After linting and formatting, validate the build:

1. **Build the documentation to check for errors:**
   ```bash
   mkdocs build
   ```
2. **Common build errors and solutions:**
   - **Invalid Markdown syntax**: Check for unclosed code blocks, incorrect indentation, or malformed tables
   - **Missing links**: Verify all internal links point to existing files (e.g., `[link](../guide/missing.md)`)
   - **Invalid YAML in mkdocs.yml**: Ensure proper indentation and syntax in configuration
   - **Missing images**: Verify all referenced images exist in `docs/assets/`
   - **Mermaid diagram errors**: Validate diagram syntax if using Mermaid blocks
3. **If build errors occur:**
   - Read the error message carefully - MkDocs provides specific line numbers and file paths
   - Fix the identified issues in the Markdown files
   - Re-run `mkdocs build` to verify all errors are resolved
   - Only proceed to visual testing after a successful build

### Visual Verification with Playwright

After a successful build, verify the changes are displaying correctly:

1. Start the local MkDocs server:

   ```bash
   mkdocs serve
   ```

2. Use Playwright MCP to check the documentation:
   - Navigate to `http://localhost:8000` using `mcp__playwright__browser_navigate`
   - Take a snapshot with `mcp__playwright__browser_snapshot` to verify the page structure
   - Check specific pages you've modified (e.g., `http://localhost:8000/getting-started/installation/`)
   - Verify navigation links work correctly
   - Ensure code blocks and diagrams render properly

3. Common checks to perform:
   - Homepage loads with correct logo and navigation
   - Updated content appears as expected
   - Links between pages work correctly
   - Search functionality returns relevant results
   - Mobile responsive view displays properly (resize browser window)

## Working with Main Kopi Project

The main Rust application at `/workspaces/kopi-workspace/kopi/` contains:

- Source code in `src/`
- Build with: `cargo build --release`
- Test with: `cargo test --quiet`
- Format with: `cargo fmt`
- Lint with: `cargo clippy`

Refer to `/workspaces/kopi-workspace/kopi/CLAUDE.md` for detailed development guidelines.

## Important Notes

1. **Metadata Updates**: The metadata files in `docs/metadata/` are automatically generated. Manual edits will be overwritten by the daily GitHub Actions workflow.

2. **Documentation Deployment**: The site is automatically deployed to GitHub Pages when changes are pushed to the main branch.

3. **Cross-References**: When updating documentation, ensure consistency between:
   - This documentation site
   - Main project README at `/workspaces/kopi-workspace/kopi/README.md`
   - In-code documentation and help text

4. **Version Compatibility**: Documentation should cover all supported JDK distributions and versions listed in `docs/reference/distributions.md`.
