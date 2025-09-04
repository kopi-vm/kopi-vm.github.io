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
