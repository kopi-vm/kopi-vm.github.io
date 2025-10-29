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

- **Priority**: Highest (overrides all other settings)
- **Scope**: Current shell and child processes
- **Use case**: Testing, CI/CD, temporary overrides

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

- **Default**: `~/.kopi`
- **Scope**: All Kopi operations
- **Use case**: Shared installations, custom locations

## Configuration Override Variables

These environment variables override settings from the configuration file using a structured naming pattern.

### Override Pattern: KOPI\_\<SECTION>\_\_\<FIELD>

Kopi uses a systematic naming convention for environment variable overrides with these rules:

**Pattern structure:**

- Prefix: Always `KOPI_`
- Section name: Configuration section in UPPERCASE
- Separator: Double underscore `__` (two underscores)
- Field name: Configuration key in UPPERCASE

**Conversion rules:**

1. Convert section and field names to UPPERCASE
2. Replace dots (`.`) with double underscores (`__`)
3. Nested configuration uses multiple double underscores

**Examples of pattern application:**

| Configuration File Path        | Environment Variable                  | Example Value |
| ------------------------------ | ------------------------------------- | ------------- |
| `default_distribution`         | `KOPI_DEFAULT_DISTRIBUTION`           | `corretto`    |
| `locking.mode`                 | `KOPI_LOCKING__MODE`                  | `advisory`    |
| `locking.timeout`              | `KOPI_LOCKING__TIMEOUT`               | `900`         |
| `auto_install.enabled`         | `KOPI_AUTO_INSTALL__ENABLED`          | `false`       |
| `auto_install.prompt`          | `KOPI_AUTO_INSTALL__PROMPT`           | `true`        |
| `storage.min_disk_space_mb`    | `KOPI_STORAGE__MIN_DISK_SPACE_MB`     | `1024`        |
| `metadata.cache.max_age_hours` | `KOPI_METADATA__CACHE__MAX_AGE_HOURS` | `1440`        |

**Why double underscores?**

Single underscores separate words within a field name (e.g., `min_disk_space_mb`), while double underscores separate configuration hierarchy levels. This distinction allows Kopi to correctly parse nested configuration paths.

**Type handling:**

- **Strings**: Direct string values (e.g., `KOPI_DEFAULT_DISTRIBUTION=temurin`)
- **Integers**: Numeric values (e.g., `KOPI_LOCK_TIMEOUT=600`)
- **Booleans**: `true` or `false` (lowercase, e.g., `KOPI_AUTO_INSTALL__ENABLED=false`)
- **Arrays**: Comma-separated lists with no spaces (e.g., `KOPI_ADDITIONAL_DISTRIBUTIONS=custom-jdk,company-build`)

### General Configuration

| Variable                        | Description                          | Default   | Type   |
| ------------------------------- | ------------------------------------ | --------- | ------ |
| `KOPI_DEFAULT_DISTRIBUTION`     | Default JDK distribution             | `temurin` | String |
| `KOPI_ADDITIONAL_DISTRIBUTIONS` | Additional distributions to consider | `[]`      | Array  |

### Storage Configuration

| Variable                          | Description                       | Default | Type    |
| --------------------------------- | --------------------------------- | ------- | ------- |
| `KOPI_STORAGE__MIN_DISK_SPACE_MB` | Minimum required disk space in MB | `500`   | Integer |

### Auto-Install Configuration

| Variable                          | Description                         | Default | Type    |
| --------------------------------- | ----------------------------------- | ------- | ------- |
| `KOPI_AUTO_INSTALL__ENABLED`      | Enable automatic JDK installation   | `true`  | Boolean |
| `KOPI_AUTO_INSTALL__PROMPT`       | Prompt before auto-installation     | `true`  | Boolean |
| `KOPI_AUTO_INSTALL__TIMEOUT_SECS` | Timeout for auto-install operations | `300`   | Integer |

### Shims Configuration

| Variable                          | Description                              | Default | Type    |
| --------------------------------- | ---------------------------------------- | ------- | ------- |
| `KOPI_SHIMS__AUTO_CREATE_SHIMS`   | Automatically create shims for JDK tools | `true`  | Boolean |
| `KOPI_SHIMS__ADDITIONAL_TOOLS`    | Additional tools to create shims for     | `[]`    | Array   |
| `KOPI_SHIMS__EXCLUDE_TOOLS`       | Tools to exclude from shim creation      | `[]`    | Array   |
| `KOPI_SHIMS__AUTO_INSTALL`        | Auto-install JDK when shim is executed   | `false` | Boolean |
| `KOPI_SHIMS__AUTO_INSTALL_PROMPT` | Prompt before shim auto-install          | `true`  | Boolean |
| `KOPI_SHIMS__INSTALL_TIMEOUT`     | Timeout for shim installations (seconds) | `600`   | Integer |

### Locking Configuration

| Variable             | Description                                       | Default | Type          |
| -------------------- | ------------------------------------------------- | ------- | ------------- |
| `KOPI_LOCK_TIMEOUT`  | Lock acquisition timeout (seconds or "infinite")  | `600`   | Int or String |
| `KOPI_LOCKING__MODE` | Lock strategy ("auto", "advisory", or "fallback") | `auto`  | String        |

**KOPI_LOCK_TIMEOUT** controls how long Kopi waits to acquire locks before timing out. This variable is checked during lock acquisition and overrides the configuration file setting but is overridden by the `--lock-timeout` CLI flag.

**Timeout precedence** (highest to lowest):

1. CLI flag: `--lock-timeout`
2. Environment variable: `KOPI_LOCK_TIMEOUT`
3. Configuration file: `locking.timeout`
4. Built-in default: 600 seconds

**Accepted values:**

- Positive integers: Number of seconds to wait (e.g., `30`, `600`, `3600`)
- String `"infinite"`: Wait indefinitely without timeout

**Examples:**

```bash
# Wait up to 2 minutes for lock
export KOPI_LOCK_TIMEOUT=120
kopi install temurin@21

# Wait indefinitely
export KOPI_LOCK_TIMEOUT=infinite
kopi cache refresh

# Override for single command
KOPI_LOCK_TIMEOUT=30 kopi uninstall corretto@17
```

For more information about the locking system, see the [Locking concept page](../concepts/locking.md).

### Metadata Cache Configuration

| Variable                                | Description                         | Default | Type    |
| --------------------------------------- | ----------------------------------- | ------- | ------- |
| `KOPI_METADATA__CACHE__MAX_AGE_HOURS`   | Maximum age of cached metadata      | `720`   | Integer |
| `KOPI_METADATA__CACHE__AUTO_REFRESH`    | Automatically refresh expired cache | `true`  | Boolean |
| `KOPI_METADATA__CACHE__REFRESH_ON_MISS` | Refresh cache when item not found   | `true`  | Boolean |

### Examples

```bash
# Set default distribution to Corretto
export KOPI_DEFAULT_DISTRIBUTION=corretto

# Require at least 1GB free space
export KOPI_STORAGE__MIN_DISK_SPACE_MB=1024

# Disable auto-installation prompts
export KOPI_AUTO_INSTALL__PROMPT=false

# Exclude specific tools from shim creation
export KOPI_SHIMS__EXCLUDE_TOOLS=jvisualvm,jconsole

# Set longer cache retention (60 days)
export KOPI_METADATA__CACHE__MAX_AGE_HOURS=1440
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
