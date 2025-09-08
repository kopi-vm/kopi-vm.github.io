# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Documentation Language Policy

All documentation output in this project must be written in English, including:

- Code comments
- Commit messages
- README files
- API documentation
- Error messages
- User-facing documentation
- Test descriptions
- TODO comments
- Any other written documentation

## Project Overview

This is the documentation website for Kopi, a fast JDK version management tool written in Rust. The site is built with MkDocs Material and hosted on GitHub Pages at <https://kopi-vm.github.io/>.

The main Kopi application source code is located at `/workspaces/kopi-workspace/kopi/`.

## Key Commands

### Documentation Development

- `mkdocs serve` - Start local development server (default: <http://127.0.0.1:8000>)
- `mkdocs build` - Build static site to `site/` directory
- `mkdocs serve --dev-addr 0.0.0.0:8000` - Serve on all interfaces

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
│   ├── troubleshooting.md # Common issues and solutions
│   └── faq.md            # Frequently asked questions
└── .github/workflows/     # GitHub Actions for deployment
```

### Key Components

**Documentation System:**

- Static site generator: MkDocs with Material theme
- Markdown content with Mermaid diagram support
- Automatic search indexing and offline support
- Responsive navigation with tabs and sections

**GitHub Actions Workflow:**

- File: `.github/workflows/deploy.yml`
- Automatically builds and deploys site to GitHub Pages on push to main branch
- Uses MkDocs Material and Mike for versioned documentation

## Content Guidelines

### Documentation Structure

- **Getting Started**: Installation methods, quickstart guide, first steps
- **Guide**: Practical guides for version management, projects, shell integration
- **Concepts**: Technical explanations of shims, metadata, caching, architecture
- **Reference**: Complete command documentation, configuration options, environment variables

### Writing Style

- Clear, concise technical documentation
- Documentation should be written using natural language and Mermaid diagrams only
- Avoid code blocks or code examples except for Mermaid diagrams
- Include platform-specific notes where relevant
- Link between related pages for better navigation
- Verify implementation basis before documenting

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
   bun run format
   ```

   This runs remark with:
   - `config/remark-format.json` configuration
   - Formats Markdown files according to style settings
   - Applies Prettier formatting through integrated plugins (`remark-preset-prettier` and `unified-prettier`)
   - Auto-fixes formatting issues and outputs the results

2. **Lint the documentation to check for issues:**

   ```bash
   bun run lint
   ```

   This runs remark with:
   - `config/remark-lint.json` configuration
   - Checks Markdown syntax and style
   - Validates internal links with `remark-validate-links`
   - Checks for dead URLs with `remark-lint-no-dead-urls` (10-second timeout)
   - Applies Prettier formatting rules through `remark-preset-prettier`

3. **Common linting errors and solutions:**
   - **Heading violations**: Maximum heading length is 80 characters
   - **List formatting**: Use `-` for bullets, proper indentation required
   - **Link validation**: Broken internal links will be caught by remark-validate-links
   - **Dead URLs**: External links are checked with 10-second timeout
   - **Line length violations**: Lines exceeding 120 characters (configured in prettier)
   - **Inconsistent formatting**: Use emphasis with `_` and rules with `-`
   - **Missing newline at EOF**: Ensure files end with a newline

4. **If linting errors occur:**
   - Run `bun run format` first to auto-fix most issues
   - For remaining errors, check the specific error messages from remark
   - Fix link errors by verifying paths and URLs
   - Re-run `bun run lint` to verify all issues are resolved

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

1. **Documentation Deployment**: The site is automatically deployed to GitHub Pages when changes are pushed to the main branch.

2. **Cross-References**: When updating documentation, ensure consistency between:
   - This documentation site
   - Main project README at `/workspaces/kopi-workspace/kopi/README.md`
   - In-code documentation and help text

3. **Version Compatibility**: Documentation should cover all supported JDK distributions and versions.
