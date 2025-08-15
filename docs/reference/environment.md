# Environment Variables

Complete reference for environment variables that affect Kopi's behavior.

## Core Variables

### KOPI_VERSION

Force a specific JDK version, overriding all other version detection mechanisms.

```bash
# Use JDK 21 regardless of project settings
export KOPI_VERSION=21
java --version  # Uses JDK 21

# Use specific distribution and version
export KOPI_VERSION=temurin@21.0.2
java --version  # Uses Temurin 21.0.2

# Unset to restore normal behavior
unset KOPI_VERSION
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

### KOPI_CONFIG

Path to alternative configuration file.

```bash
# Use custom configuration
export KOPI_CONFIG=/etc/kopi/config.toml

# Use project-specific configuration
export KOPI_CONFIG=./project-kopi.toml
```

**Default**: `$KOPI_HOME/config.toml`  
**Scope**: Configuration loading  
**Use case**: Multiple configurations, testing

## Cache Variables

### KOPI_CACHE_DIR

Override cache directory location.

```bash
# Use temporary cache
export KOPI_CACHE_DIR=/tmp/kopi-cache

# Use shared cache
export KOPI_CACHE_DIR=/var/cache/kopi
```

**Default**: `$KOPI_HOME/cache`  
**Scope**: Cache operations  
**Use case**: Disk space management, shared caches

### KOPI_NO_CACHE

Disable caching completely.

```bash
# Disable all caching
export KOPI_NO_CACHE=1

# Always fetch fresh metadata
kopi search  # Bypasses cache
```

**Values**: `1`, `true`, `yes` (to disable)  
**Scope**: Cache operations  
**Use case**: Debugging, forcing updates

### KOPI_OFFLINE

Force offline mode, only use cached data.

```bash
# Enable offline mode
export KOPI_OFFLINE=1

# All operations use local data only
kopi search  # Uses cached metadata
kopi install 21   # Fails if not cached
```

**Values**: `1`, `true`, `yes` (to enable)  
**Scope**: Network operations  
**Use case**: Airplane mode, network restrictions

## Debug Variables

### KOPI_DEBUG

Enable general debug output.

```bash
# Enable debug mode
export KOPI_DEBUG=1
kopi install 21

# Output includes:
# - Version resolution steps
# - Network requests
# - File operations
# - Timing information
```

**Values**: `1`, `true`, `yes` (to enable)  
**Scope**: All operations  
**Use case**: Troubleshooting, development

### KOPI_DEBUG_METADATA

Debug metadata operations.

```bash
export KOPI_DEBUG_METADATA=1
kopi search

# Shows:
# - Metadata sources
# - Cache hits/misses
# - Parsing details
# - Merge operations
```

**Values**: `1`, `true`, `yes` (to enable)  
**Scope**: Metadata operations  
**Use case**: Metadata troubleshooting

### KOPI_DEBUG_CACHE

Debug cache operations.

```bash
export KOPI_DEBUG_CACHE=1
kopi cache update

# Shows:
# - Cache reads/writes
# - TTL calculations
# - Eviction decisions
# - Compression stats
```

**Values**: `1`, `true`, `yes` (to enable)  
**Scope**: Cache operations  
**Use case**: Cache troubleshooting

### KOPI_DEBUG_SHIM

Debug shim execution.

```bash
export KOPI_DEBUG_SHIM=1
java --version

# Shows:
# - Shim invocation
# - Version resolution
# - Binary selection
# - Execution details
```

**Values**: `1`, `true`, `yes` (to enable)  
**Scope**: Shim operations  
**Use case**: Shim troubleshooting

### KOPI_TRACE

Enable trace-level logging for specific components.

```bash
# Trace all operations
export KOPI_TRACE=all

# Trace specific components
export KOPI_TRACE=metadata,cache,network

# Multiple components
export KOPI_TRACE=shim:network:download
```

**Values**: Component names (comma or colon separated)  
**Scope**: Specified components  
**Use case**: Deep debugging

## Network Variables

### KOPI_HTTP_PROXY / KOPI_HTTPS_PROXY

Configure HTTP(S) proxy settings.

```bash
# HTTP proxy
export KOPI_HTTP_PROXY=http://proxy.company.com:8080

# HTTPS proxy
export KOPI_HTTPS_PROXY=https://proxy.company.com:8443

# With authentication
export KOPI_HTTP_PROXY=http://user:pass@proxy.company.com:8080
```

**Format**: `http://[user:pass@]host:port`  
**Scope**: Network requests  
**Use case**: Corporate networks

### KOPI_NO_PROXY

Hosts to bypass proxy.

```bash
# Bypass proxy for specific hosts
export KOPI_NO_PROXY=localhost,127.0.0.1,internal.company.com

# Bypass for subnet
export KOPI_NO_PROXY=192.168.0.0/16

# Multiple entries
export KOPI_NO_PROXY=localhost,*.company.com,10.0.0.0/8
```

**Format**: Comma-separated hosts/patterns  
**Scope**: Network requests  
**Use case**: Internal resources

### KOPI_NETWORK_TIMEOUT

Override network timeout settings.

```bash
# Set timeout to 60 seconds
export KOPI_NETWORK_TIMEOUT=60

# Longer timeout for slow connections
export KOPI_NETWORK_TIMEOUT=300
```

**Unit**: Seconds  
**Default**: 30  
**Scope**: Network operations  
**Use case**: Slow connections

### KOPI_PARALLEL_DOWNLOADS

Number of parallel download connections.

```bash
# Single connection (slower, more compatible)
export KOPI_PARALLEL_DOWNLOADS=1

# More parallel connections (faster)
export KOPI_PARALLEL_DOWNLOADS=8
```

**Default**: 4  
**Range**: 1-16  
**Scope**: Download operations  
**Use case**: Bandwidth optimization

## Security Variables

### KOPI_VERIFY_CHECKSUMS

Control checksum verification.

```bash
# Disable checksum verification (not recommended)
export KOPI_VERIFY_CHECKSUMS=false

# Force verification even for cached files
export KOPI_VERIFY_CHECKSUMS=always
```

**Values**: `true`, `false`, `always`  
**Default**: `true`  
**Scope**: Download verification  
**Use case**: Development, testing

### KOPI_VERIFY_CERTIFICATES

Control SSL certificate verification.

```bash
# Disable certificate verification (INSECURE!)
export KOPI_VERIFY_CERTIFICATES=false

# Useful for:
# - Self-signed certificates
# - Internal CAs
# - Development only
```

**Values**: `true`, `false`  
**Default**: `true`  
**Scope**: HTTPS connections  
**Use case**: Development environments

### KOPI_CA_BUNDLE

Custom CA certificate bundle.

```bash
# Use custom CA bundle
export KOPI_CA_BUNDLE=/etc/ssl/certs/company-ca.pem

# Use system CA bundle
export KOPI_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
```

**Format**: Path to PEM file  
**Scope**: HTTPS connections  
**Use case**: Corporate CAs

## Shell Integration Variables

### KOPI_SKIP_SHIM

Skip shim processing entirely.

```bash
# Bypass shims
export KOPI_SKIP_SHIM=1
java --version  # Uses system Java

# Temporary bypass
KOPI_SKIP_SHIM=1 java --version
```

**Values**: `1`, `true`, `yes` (to skip)  
**Scope**: Shim execution  
**Use case**: Debugging, fallback

### KOPI_SHELL

Override detected shell type.

```bash
# Force bash behavior
export KOPI_SHELL=bash

# Force PowerShell behavior
export KOPI_SHELL=powershell
```

**Values**: `bash`, `zsh`, `fish`, `powershell`  
**Scope**: Shell integration  
**Use case**: Non-standard shells

### KOPI_AUTO_SWITCH

Control automatic version switching.

```bash
# Disable auto-switching
export KOPI_AUTO_SWITCH=false

# Re-enable
export KOPI_AUTO_SWITCH=true
```

**Values**: `true`, `false`  
**Default**: `true`  
**Scope**: Directory changes  
**Use case**: Performance, control

## Display Variables

### NO_COLOR

Disable colored output (standard).

```bash
# Disable all colors
export NO_COLOR=1

# Kopi respects this standard variable
kopi list  # No color output
```

**Values**: Any non-empty value  
**Scope**: Terminal output  
**Use case**: Scripts, CI/CD

### KOPI_COLOR

Override color settings.

```bash
# Force colors even in pipes
export KOPI_COLOR=always

# Disable colors
export KOPI_COLOR=never

# Auto-detect (default)
export KOPI_COLOR=auto
```

**Values**: `auto`, `always`, `never`  
**Default**: `auto`  
**Scope**: Terminal output  
**Use case**: Scripts, preferences

### KOPI_QUIET

Suppress non-error output.

```bash
# Quiet mode
export KOPI_QUIET=1

# Only errors are shown
kopi install 21  # Silent unless error
```

**Values**: `1`, `true`, `yes` (to enable)  
**Scope**: Output verbosity  
**Use case**: Scripts, automation

### KOPI_VERBOSE

Enable verbose output.

```bash
# Verbose mode
export KOPI_VERBOSE=1

# Extra information shown
kopi install 21  # Detailed progress
```

**Values**: `1`, `true`, `yes` (to enable)  
**Scope**: Output verbosity  
**Use case**: Debugging, monitoring

## Platform-Specific Variables

### Windows

```powershell
# Windows-specific paths
$env:KOPI_HOME = "C:\kopi"
$env:KOPI_TEMP = "C:\Temp\kopi"

# Use Command Prompt mode
$env:KOPI_SHELL = "cmd"
```

### macOS

```bash
# Handle quarantine attributes
export KOPI_REMOVE_QUARANTINE=1

# Prefer native architecture
export KOPI_PREFER_NATIVE=1
```

### Linux

```bash
# Specify C library preference
export KOPI_LIBC=musl  # or glibc

# Custom installation prefix
export KOPI_PREFIX=/opt
```

## Standard Java Variables

Kopi sets and manages these standard Java environment variables:

### JAVA_HOME

Automatically set to active JDK.

```bash
# Kopi sets this automatically
echo $JAVA_HOME
# /home/user/.kopi/jdks/temurin-21

# Override if needed
export JAVA_HOME=/custom/jdk/path
```

### PATH

Modified to include JDK bin directory.

```bash
# Kopi adds shims to PATH
echo $PATH
# /home/user/.kopi/shims:/usr/bin:...

# Shims handle JDK bin directory
```

### JDK_HOME

Some tools expect this (set same as JAVA_HOME).

```bash
# Kopi sets both
echo $JDK_HOME
# /home/user/.kopi/jdks/temurin-21
```

## Variable Precedence

Environment variables are checked in this order:

1. **Command-line flags** (highest priority)
2. **Environment variables**
3. **Configuration file**
4. **Defaults** (lowest priority)

Example:

```bash
# Configuration file: distribution = "temurin"
# Environment: KOPI_DEFAULT_DISTRIBUTION=corretto
# Command: kopi install --distribution zulu 21

# Result: Uses zulu (command-line wins)
```

## Best Practices

### Development

```bash
# Development environment
export KOPI_DEBUG=1
export KOPI_VERBOSE=1
export KOPI_COLOR=always
export KOPI_VERIFY_CERTIFICATES=false  # If using self-signed
```

### Production

```bash
# Production environment
export KOPI_QUIET=1
export KOPI_VERIFY_CHECKSUMS=always
export KOPI_VERIFY_CERTIFICATES=true
export KOPI_AUTO_INSTALL=false
```

### CI/CD

```bash
# CI/CD environment
export KOPI_NO_COLOR=1
export KOPI_QUIET=1
export KOPI_OFFLINE=1  # Use cached data
export KOPI_VERSION=$(cat .kopi-version)
```

### Corporate

```bash
# Corporate environment
export KOPI_HTTP_PROXY=http://proxy.corp.com:8080
export KOPI_HTTPS_PROXY=http://proxy.corp.com:8080
export KOPI_NO_PROXY=localhost,*.corp.com
export KOPI_CA_BUNDLE=/etc/ssl/corp-ca.pem
```

## Debugging Issues

### Check Active Variables

```bash
# Show all Kopi variables
env | grep KOPI

# Show Java variables
env | grep -E "(JAVA|JDK)"

# Test with specific variable
KOPI_DEBUG=1 kopi current
```

### Variable Not Working

```bash
# Ensure variable is exported
export KOPI_VERSION=21  # Correct
KOPI_VERSION=21         # Wrong (not exported)

# Check variable spelling
export KOPI_VERSON=21   # Wrong (typo)
export KOPI_VERSION=21  # Correct

# Check value format
export KOPI_DEBUG=true  # Correct
export KOPI_DEBUG=TRUE  # Correct
export KOPI_DEBUG=yes   # Correct
export KOPI_DEBUG=1     # Correct
export KOPI_DEBUG=0     # Wrong (use unset instead)
```

## Next Steps

- [Configuration](configuration.md) - Configuration file reference
- [Commands](commands.md) - Command reference
- [Troubleshooting](../troubleshooting.md) - Common issues