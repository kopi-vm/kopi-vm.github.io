# Configuration Reference

Complete reference for Kopi configuration options.

## Configuration File

Kopi's configuration is stored in a TOML format file in your home directory. The configuration file allows you to customize default behaviors, cache settings, auto-installation preferences, and metadata sources.

### File Location

| Platform    | Default Location                  |
| ----------- | --------------------------------- |
| Linux/macOS | `~/.kopi/config.toml`             |
| Windows     | `%USERPROFILE%\.kopi\config.toml` |

The configuration file is optional. If it doesn't exist, Kopi uses sensible defaults for all settings.

### Home Directory

The Kopi home directory can be customized using the KOPI_HOME environment variable. If set to an absolute path, Kopi will use that directory instead of the default \~/.kopi location. Relative paths in the KOPI_HOME variable are ignored and Kopi falls back to the default location.

## Configuration Sections

### Top-Level Settings

The following settings can be configured at the root level of the configuration file:

| Setting                  | Type   | Default   | Description                                              |
| ------------------------ | ------ | --------- | -------------------------------------------------------- |
| default_distribution     | String | `temurin` | Default JDK distribution when not specified              |
| additional_distributions | Array  | `[]`      | Additional distribution names for custom or private JDKs |

```toml
# Example top-level configuration
default_distribution = "corretto"
additional_distributions = ["custom-jdk", "internal-jdk"]
```

### Storage Configuration

The storage section controls disk space management:

| Setting           | Type    | Default | Description                                         |
| ----------------- | ------- | ------- | --------------------------------------------------- |
| min_disk_space_mb | Integer | `500`   | Minimum free disk space (MB) required for downloads |

```toml
# Example storage configuration
[storage]
min_disk_space_mb = 1024
```

### Auto-Install Configuration

The auto_install section controls automatic JDK installation behavior:

| Setting      | Type    | Default | Description                                      |
| ------------ | ------- | ------- | ------------------------------------------------ |
| enabled      | Boolean | `true`  | Automatically install missing JDK versions       |
| prompt       | Boolean | `true`  | Ask for confirmation before auto-installing      |
| timeout_secs | Integer | `300`   | Maximum installation time in seconds (5 minutes) |

```toml
# Example auto-install configuration
[auto_install]
enabled = true
prompt = false
timeout_secs = 600  # 10 minutes
```

### Shims Configuration

The shims section manages the creation and behavior of command shims:

| Setting             | Type    | Default | Description                                            |
| ------------------- | ------- | ------- | ------------------------------------------------------ |
| auto_create_shims   | Boolean | `true`  | Automatically create shims for Java executables        |
| additional_tools    | Array   | `[]`    | Additional tool names to create shims for              |
| exclude_tools       | Array   | `[]`    | Tool names to exclude from shim creation               |
| auto_install        | Boolean | `false` | Shims trigger auto-installation of missing JDKs        |
| auto_install_prompt | Boolean | `true`  | Prompt before auto-installing via shims                |
| install_timeout     | Integer | `600`   | Maximum shim installation time in seconds (10 minutes) |

```toml
# Example shims configuration
[shims]
auto_create_shims = true
additional_tools = ["custom-tool", "jlink"]
exclude_tools = ["jshell"]
auto_install = false
auto_install_prompt = true
install_timeout = 300
```

### Locking Configuration

The locking section controls how Kopi coordinates concurrent operations to prevent conflicts during installations, uninstalls, and cache updates:

| Setting | Type          | Default | Description                                                      |
| ------- | ------------- | ------- | ---------------------------------------------------------------- |
| mode    | String        | `auto`  | Lock strategy: "auto", "advisory", or "fallback"                 |
| timeout | Int or String | `600`   | Lock acquisition timeout in seconds or "infinite" for no timeout |

**Lock Modes:**

- **auto** (recommended): Attempts advisory locks first, automatically downgrades to fallback locks if advisory locks are unavailable or unsupported by the filesystem. This mode provides the best balance of performance and compatibility.
- **advisory**: Uses only operating system-level locks (fcntl on Unix, file locks on Windows). Fails if the filesystem doesn't support advisory locks. Provides the best performance but may not work on network filesystems.
- **fallback**: Uses only file-based marker locks with atomic create_new semantics. Works on all filesystems but requires periodic cleanup of stale lock files through automatic hygiene processes.

**Timeout Values:**

- **Seconds** (integer): Number of seconds to wait for lock acquisition (e.g., `30`, `600`, `3600`)
- **"infinite"** (string): Wait indefinitely without timeout

**Timeout Precedence:**

The lock timeout can be configured in multiple places with the following precedence order (highest to lowest):

1. CLI flag: `--lock-timeout <DURATION>`
2. Environment variable: `KOPI_LOCK_TIMEOUT`
3. Configuration file: `locking.timeout`
4. Built-in default: 600 seconds (10 minutes)

```toml
# Example locking configuration
[locking]
# Use automatic lock strategy (recommended)
mode = "auto"

# Wait up to 10 minutes for lock acquisition
timeout = 600

# Alternative: wait indefinitely
# timeout = "infinite"

# Alternative: use only advisory locks (for local filesystems)
# mode = "advisory"

# Alternative: use only fallback locks (for network filesystems)
# mode = "fallback"
```

**Lock Scopes:**

Kopi uses different locks for different operations:

- **Installation locks**: One lock per JDK being installed (e.g., `temurin-21.0.5+11`)
- **Cache writer lock**: Serializes metadata cache updates
- **Global config lock**: Protects configuration file updates

This fine-grained locking allows multiple operations to proceed concurrently when they don't conflict. For example, installing two different JDK versions can happen simultaneously.

**Lock Hygiene:**

Kopi automatically cleans up stale lock files on startup through a hygiene process. This cleanup:

- Scans fallback lock, marker, and staging files and removes those older than the derived stale-time threshold
- Leaves active or recently created locks in place so running operations continue uninterrupted
- Emits a debug-level summary of removals and only warns when cleanup encounters an error
- Never blocks normal operations

**Network Filesystem Considerations:**

When using Kopi on network filesystems like NFS or SMB:

- Advisory locks may not be supported, causing automatic downgrade to fallback mode
- Lock operations may be slower due to network latency
- Stale locks may accumulate more frequently
- Consider using local storage for `KOPI_HOME` when possible for better performance

For detailed information about the locking system, see the [Locking concept page](../concepts/locking.md).

### Metadata Configuration

The metadata section controls how Kopi fetches and caches JDK metadata:

#### Metadata Cache Settings

The metadata.cache subsection manages metadata-specific caching:

| Setting         | Type    | Default | Description                                   |
| --------------- | ------- | ------- | --------------------------------------------- |
| max_age_hours   | Integer | `720`   | Metadata validity duration in hours (30 days) |
| auto_refresh    | Boolean | `true`  | Automatically refresh expired metadata        |
| refresh_on_miss | Boolean | `true`  | Refresh cache when requested data not found   |

```toml
# Example metadata cache configuration
[metadata.cache]
max_age_hours = 72  # 3 days
auto_refresh = true
refresh_on_miss = false
```

#### Metadata Sources

The metadata.sources array defines where Kopi fetches JDK information. Each source can be one of three types:

**HTTP Sources**

| Field         | Type    | Default | Description                      |
| ------------- | ------- | ------- | -------------------------------- |
| type          | String  | -       | Must be `"http"`                 |
| name          | String  | -       | Unique identifier for the source |
| enabled       | Boolean | `true`  | Whether this source is active    |
| base_url      | String  | -       | URL to fetch metadata from       |
| cache_locally | Boolean | `true`  | Cache fetched data locally       |
| timeout_secs  | Integer | `30`    | Request timeout in seconds       |

**Local Sources**

| Field           | Type    | Default      | Description                      |
| --------------- | ------- | ------------ | -------------------------------- |
| type            | String  | -            | Must be `"local"`                |
| name            | String  | -            | Unique identifier for the source |
| enabled         | Boolean | `true`       | Whether this source is active    |
| directory       | String  | -            | Path to metadata directory       |
| archive_pattern | String  | `"*.tar.gz"` | File pattern for archives        |
| cache_extracted | Boolean | `true`       | Cache extracted metadata         |

**Foojay API Sources**

| Field        | Type    | Default                         | Description                      |
| ------------ | ------- | ------------------------------- | -------------------------------- |
| type         | String  | -                               | Must be `"foojay"`               |
| name         | String  | -                               | Unique identifier for the source |
| enabled      | Boolean | `false`                         | Whether this source is active    |
| base_url     | String  | `"https://api.foojay.io/disco"` | Foojay API endpoint              |
| timeout_secs | Integer | `30`                            | Request timeout in seconds       |

```toml
# Example metadata sources configuration
[[metadata.sources]]
type = "http"
name = "primary"
enabled = true
base_url = "https://kopi-vm.github.io/metadata"
cache_locally = true
timeout_secs = 30

[[metadata.sources]]
type = "local"
name = "offline"
enabled = true
directory = "/opt/kopi-metadata"
archive_pattern = "*.tar.gz"
cache_extracted = true

[[metadata.sources]]
type = "foojay"
name = "foojay-backup"
enabled = false
base_url = "https://api.foojay.io/disco"
timeout_secs = 60
```

## Complete Configuration Example

```toml
# ~/.kopi/config.toml
# Complete Kopi configuration example

# Top-level settings
default_distribution = "temurin"
additional_distributions = ["custom-jdk"]

# Storage settings
[storage]
min_disk_space_mb = 1024

# Auto-install settings
[auto_install]
enabled = true
prompt = true
timeout_secs = 300

# Shims settings
[shims]
auto_create_shims = true
additional_tools = ["jlink"]
exclude_tools = ["jshell"]
auto_install = false
auto_install_prompt = true
install_timeout = 600

# Locking settings
[locking]
mode = "auto"
timeout = 600

# Metadata cache settings
[metadata.cache]
max_age_hours = 720
auto_refresh = true
refresh_on_miss = true

# Metadata sources
[[metadata.sources]]
type = "http"
name = "primary-http"
enabled = true
base_url = "https://kopi-vm.github.io/metadata"
cache_locally = true
timeout_secs = 30

[[metadata.sources]]
type = "foojay"
name = "foojay-api"
enabled = true
base_url = "https://api.foojay.io/disco"
timeout_secs = 30
```
