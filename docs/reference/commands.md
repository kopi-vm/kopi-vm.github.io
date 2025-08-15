# Commands Reference

Complete reference for all Kopi commands and options.

## Global Options

Options available for all commands:

```bash
kopi [OPTIONS] <COMMAND>

Options:
  -h, --help       Print help information
  -V, --version    Print version information
  -v, --verbose    Enable verbose output
  -q, --quiet      Suppress non-error output
  --color <WHEN>   Color output [never, auto, always]
  --config <PATH>  Use specific config file
```

## Core Commands

### kopi install

Install a JDK version.

```bash
kopi install [OPTIONS] <VERSION>

Arguments:
  <VERSION>    Version to install (e.g., "21", "temurin@21", "corretto@17.0.9")

Options:
  -d, --distribution <NAME>    JDK distribution [temurin, corretto, zulu, etc.]
  -f, --force                  Force reinstall if already installed
  --no-cache                   Skip metadata cache
  --verify                     Verify checksum after download
  --parallel                   Use parallel downloads
  -h, --help                   Print help information

Examples:
  kopi install 21                      # Install latest JDK 21
  kopi install temurin@21              # Install Temurin JDK 21
  kopi install corretto@17.0.9         # Install specific Corretto version
  kopi install --force 21              # Reinstall JDK 21
```

### kopi uninstall

Remove an installed JDK.

```bash
kopi uninstall [OPTIONS] <VERSION>

Arguments:
  <VERSION>    Version to uninstall

Options:
  -f, --force          Skip confirmation prompt
  --clean-metadata     Remove associated metadata
  -h, --help          Print help information

Examples:
  kopi uninstall temurin@21            # Uninstall Temurin 21
  kopi uninstall 17                    # Uninstall JDK 17
  kopi uninstall --force corretto@17   # Force uninstall without prompt
```

### kopi use

Set the global default JDK version.

```bash
kopi use [OPTIONS] <VERSION>

Arguments:
  <VERSION>    Version to use globally

Options:
  --install-if-missing    Install if not present
  -h, --help             Print help information

Examples:
  kopi use 21                          # Use JDK 21 globally
  kopi use temurin@17                  # Use Temurin 17 globally
  kopi use system                      # Use system JDK
```

### kopi pin

Pin JDK version for current project.

```bash
kopi pin [OPTIONS] [VERSION]

Arguments:
  [VERSION]    Version to pin (defaults to current)

Options:
  -d, --distribution <NAME>    Specify distribution
  --format <FORMAT>           Version file format [kopi, java]
  -h, --help                 Print help information

Examples:
  kopi pin                             # Pin current JDK
  kopi pin 21                          # Pin JDK 21
  kopi pin temurin@21                  # Pin Temurin 21
  kopi pin --format java 17            # Create .java-version file
```

### kopi shell

Set JDK version for current shell session.

```bash
kopi shell [OPTIONS] [VERSION]

Arguments:
  [VERSION]    Version for shell session

Options:
  --unset         Clear shell override
  --status        Show current shell override
  -h, --help      Print help information

Examples:
  kopi shell 11                        # Use JDK 11 in current shell
  kopi shell temurin@17                # Use Temurin 17 in current shell
  kopi shell --unset                   # Clear shell override
  kopi shell --status                  # Show current override
```

### kopi current

Display currently active JDK version.

```bash
kopi current [OPTIONS]

Options:
  --global        Show global default
  --project       Show project version
  --verbose       Show resolution details
  --short         Show version only
  -h, --help      Print help information

Examples:
  kopi current                         # Show active JDK
  kopi current --global                # Show global default
  kopi current --verbose               # Show with details
  kopi current --short                 # Show version only
```

## Listing Commands

### kopi list

List installed JDK versions.

```bash
kopi list [OPTIONS]

Options:
  -d, --distribution <NAME>    Filter by distribution
  --lts                       Show only LTS versions
  --verbose                   Show detailed information
  --json                      Output as JSON
  -h, --help                 Print help information

Examples:
  kopi list                            # List all installed JDKs
  kopi list --lts                      # List only LTS versions
  kopi list -d temurin                 # List Temurin installations
  kopi list --json                     # Output as JSON
```

### kopi search

Search available JDK versions.

```bash
kopi search [OPTIONS] [VERSION]

Arguments:
  [VERSION]    Version to search for (e.g., "21", "corretto", "corretto@21")

Options:
  --compact                   Show compact output (version numbers only)
  --detailed                  Show detailed information
  --json                      Output as JSON
  --lts-only                  Show only LTS versions
  -h, --help                 Print help information

Examples:
  kopi search                          # List available JDKs
  kopi search 21                       # Search for JDK 21
  kopi search corretto                 # Search for Corretto distributions
  kopi search --lts-only               # List LTS versions
  kopi search corretto@21              # Search for Corretto 21
```

## Utility Commands

### kopi which

Show path to JDK executable.

```bash
kopi which [OPTIONS] <TOOL>

Arguments:
  <TOOL>       Tool name (java, javac, jar, etc.)

Options:
  --version    Show version of the tool
  -h, --help   Print help information

Examples:
  kopi which java                      # Path to java executable
  kopi which javac                     # Path to javac
  kopi which --version java            # Java version info
```

### kopi doctor

Diagnose Kopi installation issues.

```bash
kopi doctor [OPTIONS]

Options:
  --check <COMPONENT>    Check specific component
  --fix                  Attempt to fix issues
  -h, --help            Print help information

Components:
  - shell      Shell integration
  - shims      Shim installation
  - path       PATH configuration
  - metadata   Metadata cache
  - config     Configuration file

Examples:
  kopi doctor                          # Full diagnostic
  kopi doctor --check shell            # Check shell integration
  kopi doctor --fix                    # Fix detected issues
```

### kopi setup

Initialize or repair Kopi installation.

```bash
kopi setup [OPTIONS]

Options:
  --regenerate-shims      Regenerate all shims
  --shim <NAME>          Regenerate specific shim
  --reset                Reset to defaults
  -h, --help            Print help information

Examples:
  kopi setup                           # Initial setup
  kopi setup --regenerate-shims        # Regenerate all shims
  kopi setup --shim java               # Regenerate java shim
  kopi setup --reset                   # Reset installation
```

### kopi init

Generate shell initialization script.

```bash
kopi init [OPTIONS] <SHELL>

Arguments:
  <SHELL>      Shell type [bash, zsh, fish, powershell]

Options:
  --complete    Include completions
  -h, --help    Print help information

Examples:
  kopi init bash                       # Bash initialization
  kopi init zsh --complete             # Zsh with completions
  kopi init fish                       # Fish initialization
```

## Cache Commands

### kopi cache

Manage metadata cache.

```bash
kopi cache <SUBCOMMAND>

Subcommands:
  status       Show cache status
  update       Update metadata cache
  clear        Clear cache
  prune        Remove expired entries
  verify       Verify cache integrity

Examples:
  kopi cache status                    # Show cache info
  kopi cache update                    # Update metadata
  kopi cache clear                     # Clear all caches
  kopi cache prune                     # Clean expired entries
```

### kopi cache update

Update metadata cache.

```bash
kopi cache update [OPTIONS]

Options:
  --force             Ignore TTL and force update
  --source <NAME>     Update specific source
  -h, --help         Print help information

Examples:
  kopi cache update                    # Normal update
  kopi cache update --force            # Force update
  kopi cache update --source foojay    # Update Foojay data
```

### kopi cache clear

Clear cache data.

```bash
kopi cache clear [OPTIONS]

Options:
  --metadata      Clear metadata only
  --downloads     Clear downloads only
  --all          Clear everything
  -h, --help     Print help information

Examples:
  kopi cache clear                     # Clear all caches
  kopi cache clear --metadata          # Clear metadata only
  kopi cache clear --downloads         # Clear downloads only
```

## Configuration Commands

### kopi config

Manage Kopi configuration.

```bash
kopi config <SUBCOMMAND>

Subcommands:
  get <KEY>           Get configuration value
  set <KEY> <VALUE>   Set configuration value
  list                List all settings
  edit                Edit config file
  reset               Reset to defaults

Examples:
  kopi config get defaults.distribution
  kopi config set defaults.distribution temurin
  kopi config list
  kopi config edit
  kopi config reset
```

### kopi config get

Get configuration value.

```bash
kopi config get <KEY>

Arguments:
  <KEY>        Configuration key

Examples:
  kopi config get defaults.distribution    # Get default distribution
  kopi config get cache.max_size          # Get cache size limit
```

### kopi config set

Set configuration value.

```bash
kopi config set <KEY> <VALUE>

Arguments:
  <KEY>        Configuration key
  <VALUE>      New value

Examples:
  kopi config set defaults.distribution corretto
  kopi config set cache.max_size 200
  kopi config set cache.auto_update 24
```

## Advanced Commands

### kopi prune

Remove unused JDK installations.

```bash
kopi prune [OPTIONS]

Options:
  --dry-run       Show what would be removed
  --keep-lts      Keep all LTS versions
  --older-than    Remove versions older than N days
  -f, --force     Skip confirmation
  -h, --help      Print help information

Examples:
  kopi prune                           # Remove unused JDKs
  kopi prune --dry-run                 # Preview removal
  kopi prune --keep-lts                # Keep LTS versions
  kopi prune --older-than 90           # Remove > 90 days old
```

### kopi validate

Validate version files and configuration.

```bash
kopi validate [OPTIONS] [FILE]

Arguments:
  [FILE]       File to validate (defaults to .kopi-version)

Options:
  --strict      Strict validation
  -h, --help    Print help information

Examples:
  kopi validate                        # Validate .kopi-version
  kopi validate .java-version          # Validate specific file
  kopi validate --strict               # Strict validation
```

### kopi metadata

Manage JDK metadata.

```bash
kopi metadata <SUBCOMMAND>

Subcommands:
  generate      Generate metadata from local JDKs
  import        Import custom metadata
  export        Export metadata
  validate      Validate metadata file

Examples:
  kopi metadata generate --from /opt/jdks
  kopi metadata import custom.json
  kopi metadata export --output metadata.json
  kopi metadata validate metadata.json
```

## Environment Variables

Variables that affect Kopi behavior:

```bash
# Force specific JDK version
export KOPI_VERSION=21

# Skip shim processing
export KOPI_SKIP_SHIM=1

# Enable debug output
export KOPI_DEBUG=1

# Specific debug categories
export KOPI_DEBUG_METADATA=1
export KOPI_DEBUG_CACHE=1
export KOPI_DEBUG_SHIM=1

# Force offline mode
export KOPI_OFFLINE=1

# Custom cache directory
export KOPI_CACHE_DIR=/tmp/kopi-cache

# Custom config file
export KOPI_CONFIG=/path/to/config.toml

# Disable colors
export NO_COLOR=1
```

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
# Install and use latest LTS
kopi install 21  # Install latest LTS manually
kopi use 21

# Set up new project
cd my-project
kopi install 21
kopi pin 21

# Test with multiple JDKs
for version in 11 17 21; do
    kopi shell $version
    ./gradlew test
done

# Clean up old installations
kopi prune --older-than 30
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