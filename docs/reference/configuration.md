# Configuration Reference

Complete reference for Kopi configuration options.

## Configuration File

Kopi's configuration is stored in `~/.kopi/config.toml` using TOML format.

### File Location

| Platform | Default Location |
|----------|-----------------|
| Linux/macOS | `~/.kopi/config.toml` |
| Windows | `%USERPROFILE%\.kopi\config.toml` |

### Creating Configuration

```bash
# Create default configuration
kopi config init

# Edit configuration
kopi config edit

# View current configuration
kopi config list
```

## Configuration Sections

### [defaults]

Default settings for JDK installation and management.

```toml
[defaults]
# Default JDK distribution when not specified
distribution = "temurin"

# Default major version for commands
version = "21"

# Prefer LTS versions when available
prefer_lts = true

# Auto-install missing versions
auto_install = false
```

### [preferences]

User preferences for JDK selection.

```toml
[preferences]
# Distribution preference order
distributions = ["temurin", "corretto", "zulu", "graalvm"]

# Architecture preference (auto-detected if not set)
# architecture = "x64"  # or "aarch64", "x86"

# Operating system (auto-detected if not set)
# os = "linux"  # or "macos", "windows"

# C library preference for Linux
libc = "glibc"  # or "musl"
```

### [cache]

Cache configuration for metadata and downloads.

```toml
[cache]
# Maximum cache size in MB
max_size = 100

# Metadata cache TTL in hours
metadata_ttl = 6

# Version resolution cache TTL in seconds
resolution_ttl = 1

# Enable cache compression
compress = true

# Auto-update interval in hours (0 = disabled)
auto_update = 24

# Cache directory (defaults to ~/.kopi/cache)
# directory = "/custom/cache/path"
```

### [cache.sources]

Priority for metadata sources (lower = higher priority).

```toml
[cache.sources]
precached = 1    # Pre-generated metadata from CDN
foojay = 2       # Foojay API
custom = 3       # User-defined sources
```

### [network]

Network settings for downloads and API calls.

```toml
[network]
# Connection timeout in seconds
connect_timeout = 30

# Read timeout in seconds
read_timeout = 300

# Number of retry attempts
retries = 3

# Retry delay in seconds
retry_delay = 2

# Maximum parallel downloads
parallel_downloads = 4

# Proxy settings (optional)
# http_proxy = "http://proxy.example.com:8080"
# https_proxy = "https://proxy.example.com:8080"
# no_proxy = "localhost,127.0.0.1"
```

### [security]

Security settings for downloads and verification.

```toml
[security]
# Verify checksums for downloads
verify_checksums = true

# Verify HTTPS certificates
verify_certificates = true

# Allowed download hosts
allowed_hosts = [
    "github.com",
    "api.foojay.io",
    "download.java.net",
    "cdn.azul.com",
    "corretto.aws"
]

# GPG verification (future feature)
# verify_signatures = false
```

### [logging]

Logging configuration.

```toml
[logging]
# Log level: trace, debug, info, warn, error
level = "info"

# Log file location (optional)
# file = "~/.kopi/kopi.log"

# Log format: text, json
format = "text"

# Include timestamps
timestamps = true

# Color output
color = "auto"  # auto, always, never
```

### [aliases]

Command and version aliases for convenience.

```toml
[aliases]
# Version aliases
lts = "temurin@21"
latest = "temurin@22"
java8 = "temurin@8"
java11 = "temurin@11"
java17 = "temurin@17"
java21 = "temurin@21"

# Distribution aliases
aws = "corretto"
eclipse = "temurin"
oracle = "graalvm"
ibm = "semeru"
ms = "microsoft"
```

### [shell]

Shell integration settings.

```toml
[shell]
# Auto-switch on directory change
auto_switch = true

# Show version in prompt
show_in_prompt = false

# Prompt format
prompt_format = "[kopi: {version}]"

# Hook scripts (optional)
# pre_switch_hook = "~/.kopi/hooks/pre-switch.sh"
# post_switch_hook = "~/.kopi/hooks/post-switch.sh"
```

### [paths]

Custom path configuration.

```toml
[paths]
# JDK installation directory
jdks = "~/.kopi/jdks"

# Shims directory
shims = "~/.kopi/shims"

# Downloads directory
downloads = "~/.kopi/downloads"

# Cache directory
cache = "~/.kopi/cache"

# Metadata directory
metadata = "~/.kopi/metadata"
```

### [metadata.sources]

Custom metadata sources.

```toml
[[metadata.sources]]
name = "internal"
url = "https://internal.company.com/kopi/metadata.json"
priority = 1
ttl = 12  # hours

[[metadata.sources]]
name = "local"
path = "/shared/kopi/metadata.json"
priority = 2
```

### [experimental]

Experimental features (may change or be removed).

```toml
[experimental]
# Enable experimental features
enabled = false

# Specific features
parallel_install = false
predictive_caching = false
smart_cleanup = false
version_constraints = false
```

## Complete Example Configuration

```toml
# ~/.kopi/config.toml
# Kopi Configuration File

[defaults]
distribution = "temurin"
version = "21"
prefer_lts = true
auto_install = false

[preferences]
distributions = ["temurin", "corretto", "zulu"]
libc = "glibc"

[cache]
max_size = 100
metadata_ttl = 6
resolution_ttl = 1
compress = true
auto_update = 24

[cache.sources]
precached = 1
foojay = 2
custom = 3

[network]
connect_timeout = 30
read_timeout = 300
retries = 3
retry_delay = 2
parallel_downloads = 4

[security]
verify_checksums = true
verify_certificates = true
allowed_hosts = [
    "github.com",
    "api.foojay.io",
    "download.java.net",
    "cdn.azul.com",
    "corretto.aws"
]

[logging]
level = "info"
format = "text"
timestamps = true
color = "auto"

[aliases]
lts = "temurin@21"
latest = "temurin@22"
java8 = "temurin@8"
java11 = "temurin@11"
java17 = "temurin@17"
java21 = "temurin@21"
aws = "corretto"
eclipse = "temurin"

[shell]
auto_switch = true
show_in_prompt = false
prompt_format = "[kopi: {version}]"

[paths]
jdks = "~/.kopi/jdks"
shims = "~/.kopi/shims"
downloads = "~/.kopi/downloads"
cache = "~/.kopi/cache"
metadata = "~/.kopi/metadata"

[[metadata.sources]]
name = "company"
url = "https://artifacts.company.com/kopi/metadata.json"
priority = 1
ttl = 12

[experimental]
enabled = false
```

## Environment Variable Overrides

Most configuration options can be overridden with environment variables:

```bash
# Override configuration file
export KOPI_CONFIG=/path/to/config.toml

# Override specific settings
export KOPI_DEFAULT_DISTRIBUTION=corretto
export KOPI_CACHE_MAX_SIZE=200
export KOPI_NETWORK_TIMEOUT=60
export KOPI_LOG_LEVEL=debug

# Disable features
export KOPI_NO_CACHE=1
export KOPI_NO_VERIFY=1
export KOPI_OFFLINE=1
```

## Configuration Commands

### Viewing Configuration

```bash
# List all settings
kopi config list

# Get specific value
kopi config get defaults.distribution

# Show configuration file location
kopi config path
```

### Modifying Configuration

```bash
# Set value
kopi config set defaults.distribution corretto

# Add alias
kopi config set aliases.work "corretto@17"

# Remove setting
kopi config unset experimental.enabled
```

### Managing Configuration

```bash
# Edit configuration file
kopi config edit

# Validate configuration
kopi config validate

# Reset to defaults
kopi config reset

# Backup configuration
kopi config backup

# Import configuration
kopi config import /path/to/config.toml
```

## Platform-Specific Configuration

### Linux

```toml
[platform.linux]
# Prefer specific C library
libc = "glibc"  # or "musl"

# Custom installation prefix
prefix = "/opt/kopi"
```

### macOS

```toml
[platform.macos]
# Handle quarantine attribute
remove_quarantine = true

# Prefer native architecture
prefer_native = true
```

### Windows

```toml
[platform.windows]
# Use junction points instead of symlinks
use_junctions = true

# Add to system PATH
update_system_path = false
```

## Migration from Other Tools

### From SDKMAN!

```toml
# Import SDKMAN! settings
[migration.sdkman]
import_candidates = true
import_versions = true
config_path = "~/.sdkman/etc/config"
```

### From jEnv

```toml
# Import jEnv settings
[migration.jenv]
import_versions = true
config_path = "~/.jenv"
```

## Profiles

Support for multiple configuration profiles:

```toml
# Default profile is always active

[profiles.work]
defaults.distribution = "corretto"
network.http_proxy = "http://corp-proxy:8080"

[profiles.personal]
defaults.distribution = "temurin"
preferences.distributions = ["temurin", "graalvm"]

# Activate with: KOPI_PROFILE=work
```

## Validation

Configuration validation rules:

1. **Required fields**: None (all have defaults)
2. **Type checking**: Enforced by TOML parser
3. **Value constraints**:
   - `cache.max_size`: 1-10000 MB
   - `network.timeout`: 1-3600 seconds
   - `cache.metadata_ttl`: 0-168 hours
4. **Path validation**: Must be valid filesystem paths
5. **URL validation**: Must be valid HTTP(S) URLs

## Best Practices

1. **Keep it simple**: Only configure what you need
2. **Use aliases**: Create shortcuts for common versions
3. **Set preferences**: Order distributions by preference
4. **Configure cache**: Adjust based on disk space
5. **Security**: Always verify checksums in production
6. **Logging**: Use appropriate log levels
7. **Backup**: Keep a backup of your configuration

## Troubleshooting

### Configuration Not Loading

```bash
# Check configuration path
kopi config path

# Validate configuration
kopi config validate

# Test with minimal config
kopi --config /tmp/minimal.toml list
```

### Settings Not Applied

```bash
# Check for environment overrides
env | grep KOPI

# Verify configuration
kopi config get <setting>

# Reload configuration
kopi config reload
```

## Next Steps

- [Environment Variables](environment.md) - Environment settings
- [Commands](commands.md) - Command reference
- [Troubleshooting](../troubleshooting.md) - Common issues