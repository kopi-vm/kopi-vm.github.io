# Environment Variables

Environment variables that affect Kopi's behavior.

## Core Variables

### KOPI_JAVA_VERSION

Force a specific JDK version, overriding all other version detection mechanisms.

```bash
# Use JDK 21 regardless of project settings
export KOPI_JAVA_VERSION=21
java --version  # Uses JDK 21

# Use specific distribution and version
export KOPI_JAVA_VERSION=temurin@21.0.2
java --version  # Uses Temurin 21.0.2

# Unset to restore normal behavior
unset KOPI_JAVA_VERSION
```

**Priority**: Highest (overrides all other settings)  
**Scope**: Current shell and child processes  
**Use case**: Testing, CI/CD, temporary overrides

### KOPI_HOME

Override the default Kopi installation directory.

```bash
# Default: ~/.kopi
export KOPI_HOME=/opt/kopi

# All Kopi data will be stored here:
# /opt/kopi/jdks/
# /opt/kopi/shims/
# /opt/kopi/cache/
# /opt/kopi/config.toml
```

**Default**: `~/.kopi`  
**Scope**: All Kopi operations  
**Use case**: Shared installations, custom locations

## Configuration Override Variables

These environment variables override settings from the configuration file. They follow the pattern `KOPI_<SECTION>__<KEY>` where section names are uppercase and nested keys are separated by double underscores.

### KOPI_DEFAULT_DISTRIBUTION

Override the default JDK distribution.

```bash
# Use Corretto as default instead of Temurin
export KOPI_DEFAULT_DISTRIBUTION=corretto
kopi install 21  # Installs corretto@21
```

**Default**: `temurin`  
**Values**: Any valid distribution name (temurin, corretto, zulu, etc.)

### KOPI_STORAGE__MIN_DISK_SPACE_MB

Minimum required disk space in MB for JDK installations.

```bash
# Require at least 1GB free space
export KOPI_STORAGE__MIN_DISK_SPACE_MB=1024
```

**Default**: `500`  
**Unit**: Megabytes

### Auto-Install Settings

Control automatic JDK installation behavior.

```bash
# Enable auto-installation
export KOPI_AUTO_INSTALL__ENABLED=true

# Enable prompting before auto-install
export KOPI_AUTO_INSTALL__PROMPT=true

# Set timeout for auto-install operations (seconds)
export KOPI_AUTO_INSTALL__TIMEOUT_SECS=300
```

### Shim Settings

```bash
# Control automatic shim creation
export KOPI_SHIMS__AUTO_CREATE_SHIMS=true
```

### Cache Settings

```bash
# Maximum age of cache in hours
export KOPI_CACHE__MAX_AGE_HOURS=24

# Enable automatic cache refresh
export KOPI_CACHE__AUTO_REFRESH=true

# Refresh cache when requested item is not found
export KOPI_CACHE__REFRESH_ON_MISS=true
```

## Debug Logging

### RUST_LOG

Kopi uses the Rust `env_logger` crate for debug logging. You can control the log level using the `RUST_LOG` environment variable.

```bash
# Enable debug logging
RUST_LOG=debug kopi install 21

# Enable trace-level logging (very verbose)
RUST_LOG=trace kopi current

# Enable debug for specific modules
RUST_LOG=kopi::version=debug kopi current
RUST_LOG=kopi::cache=trace kopi search 21

# Multiple modules with different levels
RUST_LOG=kopi::api=debug,kopi::download=trace kopi install 21
```

**Log Levels** (from least to most verbose):
- `error` - Only errors
- `warn` - Warnings and errors
- `info` - Informational messages
- `debug` - Debug information
- `trace` - Very detailed trace information

**Examples**:
```bash
# Debug version resolution
RUST_LOG=kopi::version::resolver=debug kopi current

# Debug cache operations
RUST_LOG=kopi::cache=debug kopi cache refresh

# Debug network operations
RUST_LOG=kopi::api=debug,kopi::download=debug kopi install 21

# Debug everything
RUST_LOG=debug kopi doctor
```

## Network Proxy Variables

Kopi uses standard HTTP proxy environment variables for downloading JDKs and fetching metadata:

### HTTP_PROXY / http_proxy

Proxy server for HTTP requests.

```bash
# Set proxy for all HTTP requests
export HTTP_PROXY=http://proxy.company.com:8080

# With authentication
export HTTP_PROXY=http://username:password@proxy.company.com:8080
```

### HTTPS_PROXY / https_proxy

Proxy server for HTTPS requests.

```bash
# Set proxy for all HTTPS requests
export HTTPS_PROXY=http://proxy.company.com:8080

# With authentication
export HTTPS_PROXY=http://username:password@proxy.company.com:8080
```

### NO_PROXY / no_proxy

Comma-separated list of hosts to bypass proxy.

```bash
# Bypass proxy for specific hosts
export NO_PROXY=localhost,127.0.0.1,internal.company.com

# Supports wildcards
export NO_PROXY=localhost,*.internal.com
```

**Notes:**
- Proxy settings are automatically detected from environment variables
- Both uppercase and lowercase variable names are supported
- Authentication credentials can be included in the proxy URL
- The `NO_PROXY` variable supports wildcards

## Standard Java Variables

Kopi sets and manages these standard Java environment variables:

### JAVA_HOME

Automatically set to active JDK.

```bash
# Kopi sets this automatically
echo $JAVA_HOME
# /home/user/.kopi/jdks/temurin-21.0.5+11

# The kopi env command outputs this for shell evaluation
eval "$(kopi env)"
```

### PATH

Modified to include Kopi shims directory.

```bash
# Add shims to PATH (done during setup)
export PATH="$HOME/.kopi/shims:$PATH"

# Shims handle JDK binary execution
```

## Variable Precedence

Environment variables are checked in this order:

1. **Command-line flags** (highest priority)
2. **Environment variables**
3. **Configuration file**
4. **Defaults** (lowest priority)

Example:

```bash
# Configuration file: default_distribution = "temurin"
# Environment: KOPI_DEFAULT_DISTRIBUTION=corretto
# Command: kopi install 21

# Result: Uses corretto (environment variable overrides config)
```

## Version Resolution Order

When resolving which JDK version to use, Kopi checks in this order:

1. `KOPI_JAVA_VERSION` environment variable
2. `.kopi-version` file in current or parent directories
3. `.java-version` file in current or parent directories
4. Global default from configuration

## Best Practices

### CI/CD

```bash
# Set specific version for CI builds
export KOPI_JAVA_VERSION=$(cat .kopi-version)

# Use for GitHub Actions, GitLab CI, etc.
```

### Corporate Networks

```bash
# Configure proxy settings
export HTTP_PROXY=http://proxy.corp.com:8080
export HTTPS_PROXY=http://proxy.corp.com:8080
export NO_PROXY=localhost,*.corp.com
```

### Testing Multiple Versions

```bash
# Temporarily override version
KOPI_JAVA_VERSION=17 mvn test
KOPI_JAVA_VERSION=21 mvn test
```

## Debugging

### Check Active Variables

```bash
# Show all Kopi-related variables
env | grep KOPI

# Show Java-related variables
env | grep -E "(JAVA|JDK)"

# Check specific configuration
kopi current  # Shows active JDK and source
```

### Variable Not Working?

```bash
# Ensure variable is exported
export KOPI_JAVA_VERSION=21  # Correct
KOPI_JAVA_VERSION=21         # Wrong (not exported)

# Check variable spelling
export KOPI_JAVA_VERSION=21  # Correct
export KOPI_VERSION=21        # Wrong (old name)
```

## Next Steps

- [Configuration](configuration.md) - Configuration file reference
- [Commands](commands.md) - Command reference
- [Troubleshooting](../troubleshooting.md) - Common issues