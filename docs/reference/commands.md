# Commands Reference

Complete reference for all Kopi commands and options.

## Global Options

Options available for all commands:

```bash
kopi [OPTIONS] <COMMAND>

Options:
  -h, --help       Print help information
  -V, --version    Print version information
  -v, --verbose    Increase verbosity (-v info, -vv debug, -vvv trace)
```

## Core Commands

### kopi install

Install a JDK version.

```bash
kopi install [OPTIONS] <VERSION>

Arguments:
  <VERSION>    Version to install (e.g., "21", "temurin@21", "corretto@17.0.9")

Options:
  -f, --force                  Force reinstall even if already installed
  --dry-run                    Show what would be installed without actually installing
  --no-progress                Disable progress indicators
  --timeout <SECONDS>          Download timeout in seconds
  --javafx-bundled             Include packages regardless of JavaFX bundled status
  -h, --help                   Print help information

Examples:
  kopi install 21                      # Install latest JDK 21
  kopi install temurin@21              # Install Temurin JDK 21
  kopi install corretto@17.0.9         # Install specific Corretto version
  kopi install --force 21              # Reinstall JDK 21
  kopi install --dry-run 21            # Preview installation
```

### kopi uninstall

Remove an installed JDK.

```bash
kopi uninstall [OPTIONS] [VERSION]

Arguments:
  [VERSION]    Version to uninstall (optional)

Options:
  -f, --force          Skip confirmation prompts
  --dry-run            Show what would be uninstalled without actually removing
  --all                Uninstall all versions of a distribution
  --cleanup            Clean up failed or partial uninstall operations
  -h, --help          Print help information

Examples:
  kopi uninstall temurin@21            # Uninstall Temurin 21
  kopi uninstall 17                    # Uninstall JDK 17
  kopi uninstall --force corretto@17   # Force uninstall without prompt
  kopi uninstall --all                 # Uninstall all JDKs
  kopi uninstall --cleanup             # Clean up failed uninstalls
```

### kopi global

Set the global default JDK version.

```bash
kopi global <VERSION>

Arguments:
  <VERSION>    Version to set as global default

Options:
  -h, --help             Print help information

Examples:
  kopi global 21                       # Use JDK 21 globally
  kopi global temurin@17               # Use Temurin 17 globally
  kopi global system                   # Use system JDK
```

### kopi local

Set JDK version for current project.

```bash
kopi local <VERSION>

Arguments:
  <VERSION>    Version to set for current project

Options:
  -h, --help                 Print help information

Examples:
  kopi local 21                        # Set JDK 21 for project
  kopi local temurin@21                # Set Temurin 21 for project
  kopi local corretto@17.0.9           # Set specific version for project
```

### kopi shell

Set JDK version for current shell session.

```bash
kopi shell [OPTIONS] <VERSION>

Arguments:
  <VERSION>    JDK version to use

Options:
  --shell <SHELL>      Override shell detection
  -h, --help          Print help information

Aliases:
  kopi use            Same as kopi shell

Examples:
  kopi shell 11                        # Use JDK 11 in current shell
  kopi shell temurin@17                # Use Temurin 17 in current shell
  kopi use 21                          # Use JDK 21 in current shell (alias)
```

### kopi current

Display currently active JDK version.

```bash
kopi current [OPTIONS]

Options:
  -q, --quiet         Show only version number
  --json              Output in JSON format
  -h, --help          Print help information

Examples:
  kopi current                         # Show active JDK
  kopi current --quiet                 # Show version only
  kopi current --json                  # Output as JSON
```

## Listing Commands

### kopi list

List installed JDK versions.

```bash
kopi list

Options:
  -h, --help                 Print help information

Aliases:
  kopi ls                    Same as kopi list

Examples:
  kopi list                            # List all installed JDKs
  kopi ls                              # List all installed JDKs (alias)
```

### kopi search

Search available JDK versions (alias for cache search).

```bash
kopi search [OPTIONS] [VERSION]

Arguments:
  [VERSION]    Version pattern to search (e.g., "21", "corretto", "corretto@17")

Options:
  -c, --compact                Show compact output (version numbers only)
  -d, --detailed              Show detailed information including download URLs
  --json                      Output results as JSON
  --lts-only                  Show only LTS versions
  --javafx-bundled            Include packages regardless of JavaFX bundled status
  -h, --help                 Print help information

Aliases:
  kopi s                      Same as kopi search

Examples:
  kopi search                          # List available JDKs
  kopi search 21                       # Search for JDK 21
  kopi search corretto                 # Search for Corretto distributions
  kopi search --lts-only               # List LTS versions
  kopi search corretto@21              # Search for Corretto 21
```

## Utility Commands

### kopi which

Show installation path for a JDK version.

```bash
kopi which [OPTIONS] [VERSION]

Arguments:
  [VERSION]    JDK version specification (optional)

Options:
  --tool <TOOL>        Show path for specific JDK tool (default: java)
  --home               Show JDK home directory instead of executable path
  --json               Output in JSON format
  -h, --help          Print help information

Aliases:
  kopi w              Same as kopi which

Examples:
  kopi which                           # Path to current JDK's java executable
  kopi which 21                        # Path to JDK 21's java executable
  kopi which --tool javac              # Path to javac
  kopi which --home                    # JDK home directory
```

### kopi doctor

Run diagnostics on kopi installation.

```bash
kopi doctor [OPTIONS]

Options:
  --json               Output results in JSON format
  --check <CATEGORY>   Run only specific category of checks
  -h, --help          Print help information

Examples:
  kopi doctor                          # Full diagnostic
  kopi doctor --check shell            # Check specific category
  kopi doctor --json                   # Output as JSON
```

### kopi setup

Initial setup and configuration.

```bash
kopi setup [OPTIONS]

Options:
  -f, --force          Force recreation of shims even if they exist
  -h, --help          Print help information

Examples:
  kopi setup                           # Initial setup
  kopi setup --force                   # Force recreate shims
```

### kopi env

Output environment variables for shell evaluation.

```bash
kopi env [OPTIONS] [VERSION]

Arguments:
  [VERSION]    Specific version to use (defaults to current)

Options:
  --shell <SHELL>      Override shell detection
  --export             Output export statements (default: true)
  -h, --help          Print help information

Examples:
  kopi env                             # Set environment for current JDK
  kopi env 21                          # Set environment for JDK 21
  eval "$(kopi env)"                  # Bash/Zsh
  kopi env | source                    # Fish
  kopi env | Invoke-Expression         # PowerShell
```

## Cache Commands

### kopi cache

Manage JDK metadata cache.

```bash
kopi cache <SUBCOMMAND>

Subcommands:
  refresh             Refresh metadata from foojay.io API
  info                Show cache information
  clear               Clear all cached data
  search              Search for available JDK versions
  list-distributions  List all available distributions in cache

Examples:
  kopi cache info                      # Show cache info
  kopi cache refresh                   # Refresh metadata
  kopi cache clear                     # Clear all caches
  kopi cache search 21                 # Search for JDK 21
  kopi cache list-distributions        # List available distributions
```

### kopi cache refresh

Refresh metadata from foojay.io API.

```bash
kopi cache refresh [OPTIONS]

Options:
  --javafx-bundled    Include packages regardless of JavaFX bundled status
  -h, --help         Print help information

Examples:
  kopi cache refresh                   # Normal refresh
  kopi cache refresh --javafx-bundled  # Include JavaFX bundled packages
```

### kopi cache search

Search for available JDK versions.

```bash
kopi cache search [OPTIONS] <VERSION>

Arguments:
  <VERSION>    Query to search for (e.g., "21", "17.0.9", "corretto@21")

Options:
  --compact                   Display minimal information
  --detailed                  Display detailed information
  --json                      Output results as JSON
  --lts-only                  Filter to show only LTS versions
  --javafx-bundled            Include packages regardless of JavaFX bundled status
  --java-version              Force search by java_version field
  --distribution-version      Force search by distribution_version field
  -h, --help                 Print help information

Examples:
  kopi cache search 21                 # Search for JDK 21
  kopi cache search corretto           # Search for Corretto
  kopi cache search --lts-only         # Show only LTS versions
```

## Shim Commands

### kopi shim

Manage tool shims.

```bash
kopi shim <SUBCOMMAND>

Subcommands:
  add <TOOL>        Add a shim for a specific tool
  remove <TOOL>     Remove a shim for a specific tool
  list              List installed shims
  verify            Verify and repair shims

Examples:
  kopi shim add java                  # Add java shim
  kopi shim remove javac              # Remove javac shim
  kopi shim list                      # List all shims
  kopi shim list --available          # Show available tools
  kopi shim verify                    # Verify shims
  kopi shim verify --fix              # Fix issues
```

## Command Aliases

Many Kopi commands have shorter aliases for convenience:

| Standard Command | Alias | Description |
|-----------------|-------|-------------|
| `kopi install` | `kopi i` | Install a JDK version |
| `kopi list` | `kopi ls` | List installed JDK versions |
| `kopi shell` | `kopi use` | Set JDK for current shell session |
| `kopi global` | `kopi g`, `kopi default` | Set global default JDK |
| `kopi local` | `kopi l`, `kopi pin` | Set project-specific JDK |
| `kopi which` | `kopi w` | Show JDK installation path |
| `kopi search` | `kopi s` | Search available JDK versions |
| `kopi refresh` | `kopi r` | Refresh metadata cache (hidden) |
| `kopi uninstall` | `kopi u`, `kopi remove` | Uninstall a JDK |

## Environment Variables

Variables that affect Kopi behavior:

```bash
# Force specific JDK version
export KOPI_JAVA_VERSION=21

# Override Kopi home directory
export KOPI_HOME=/opt/kopi

# Enable debug output (using Rust's env_logger)
RUST_LOG=debug kopi install 21
RUST_LOG=trace kopi current

# Standard proxy variables
export HTTP_PROXY=http://proxy:8080
export HTTPS_PROXY=http://proxy:8080
export NO_PROXY=localhost,*.internal.com
```

See [Environment Variables](environment.md) for complete reference.

## Exit Codes

Kopi uses specific exit codes for different error conditions:

| Code | Description |
|------|-------------|
| 0    | Success |
| 1    | General error |
| 2    | Invalid command or arguments |
| 3    | Version file not found |
| 4    | JDK not installed |
| 5    | Installation failed |
| 10   | Configuration error |
| 13   | Permission denied |
| 20   | Network error |
| 28   | Insufficient disk space |
| 127  | Command not found |

## Examples

### Common Workflows

```bash
# Install and use latest LTS globally
kopi install 21
kopi global 21

# Set up new project
cd my-project
kopi install 21
kopi local 21

# Test with multiple JDKs
for version in 11 17 21; do
    kopi shell $version
    ./gradlew test
done

# Manually clean up old installations
kopi list  # Check installed versions
kopi uninstall temurin@11  # Remove specific version
```

### CI/CD Integration

```bash
# GitHub Actions
kopi install $(cat .kopi-version)

# GitLab CI
kopi install --verify $(cat .kopi-version)

# Jenkins
kopi doctor --fix && kopi install $(cat .kopi-version)
```

## Next Steps

- [Configuration](configuration.md) - Configuration options
- [Environment Variables](environment.md) - Environment settings
- [Troubleshooting](../troubleshooting.md) - Common issues