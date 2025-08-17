# Caching

Understanding Kopi's caching mechanisms for optimal performance.

## Overview

Kopi uses a metadata caching system to provide fast JDK version management while minimizing network requests. The cache stores JDK distribution metadata fetched from the Foojay API and other sources.

## Cache Architecture

```
┌─────────────────────────────────────────┐
│         Metadata Cache                   │
│      (~/.kopi/cache.json)                │
│                                          │
│  - Distribution metadata                 │
│  - Package information                   │
│  - Version mappings                      │
│  - Synonym mappings                      │
└────────────────┬────────────────────────┘
                 ▼
┌─────────────────────────────────────────┐
│       Remote Sources                     │
│                                          │
│  - Foojay API                           │
│  - Precached data                       │
│  - Custom sources                       │
└─────────────────────────────────────────┘
```

## Cache Structure

### Metadata Cache

The main cache is stored as a JSON file containing:

```json
{
  "version": 1,
  "last_updated": "2024-03-15T10:30:00Z",
  "distributions": {
    "temurin": {
      "distribution": "Temurin",
      "display_name": "Eclipse Temurin",
      "packages": [...]
    },
    "corretto": {
      "distribution": "Corretto",
      "display_name": "Amazon Corretto",
      "packages": [...]
    }
  },
  "synonym_map": {
    "adoptopenjdk": "temurin",
    "amazon": "corretto"
  }
}
```

### Cache Components

1. **Distribution Cache**: Stores metadata for each JDK distribution
2. **Package Metadata**: Contains detailed information about each JDK package
3. **Synonym Map**: Maps distribution aliases to canonical names
4. **Version Index**: Enables fast version lookups

## Cache Operations

### Loading Cache

```rust
// Cache is loaded on-demand when needed
let cache = cache::get_metadata(None, &config)?;

// Check if specific version exists in cache
if cache.has_version("21.0.2") {
    // Use cached data
}
```

### Refreshing Cache

```bash
# Refresh metadata from API
kopi cache refresh

# Refresh with JavaFX bundled packages
kopi cache refresh --javafx-bundled
```

### Searching Cache

The cache supports various search operations:

```bash
# Search for specific version
kopi cache search 21

# Search for distribution-specific version
kopi cache search corretto@21

# Search by distribution version
kopi cache search 21.0.2.13.1 --distribution-version

# Search for latest version
kopi cache search latest
```

## Cache Management Commands

### Viewing Cache Status

```bash
# Show cache information
kopi cache info

# Output:
Cache Information:
  Location: /home/user/.kopi/cache.json
  Last Updated: 2024-03-15 10:30:00
  Size: 2.3 MB

  Distributions: 15
  Total Packages: 1,827
  Unique Versions: 342
```

### Clearing Cache

```bash
# Clear all cached data
kopi cache clear

# Cache will be rebuilt on next operation
```

### Listing Distributions

```bash
# List all available distributions
kopi cache list-distributions

# Output:
Available Distributions:
  • corretto (Amazon Corretto)
  • dragonwell (Alibaba Dragonwell)
  • graalvm (GraalVM)
  • liberica (BellSoft Liberica)
  • microsoft (Microsoft Build of OpenJDK)
  • oracle (Oracle JDK)
  • sapmachine (SapMachine)
  • semeru (IBM Semeru)
  • temurin (Eclipse Temurin)
  • zulu (Azul Zulu)
```

## Cache Staleness and Updates

### Automatic Refresh

The cache is automatically refreshed when:

- It doesn't exist
- A requested version is not found in the cache
- The cache file is corrupted

### Manual Refresh

```bash
# Force refresh of metadata
kopi cache refresh
```

## Search Implementation

### Version Type Detection

Kopi automatically detects the type of version search:

```rust
// Auto-detects whether to search by java_version or distribution_version
// - 4+ components → distribution_version (e.g., "21.0.2.13.1")
// - Non-numeric build identifiers → distribution_version
// - Standard format → java_version (e.g., "21", "21.0.2")
```

### Platform Filtering

The cache automatically filters packages based on:

- Operating System (linux, macos, windows)
- Architecture (x64, aarch64, arm32, etc.)
- libc type (glibc, musl)
- Archive type (tar.gz preferred on macOS, zip as fallback)

## Performance Characteristics

### Cache Performance

- **Load time**: < 10ms for typical cache
- **Search time**: < 1ms for version lookups
- **Memory usage**: ~5-10 MB in memory
- **Disk usage**: ~2-5 MB on disk

### Network Operations

- **Metadata fetch**: 50-500ms (depending on network)
- **Parallel fetching**: Multiple sources fetched concurrently
- **Compression**: JSON cache is stored in plain text for debugging

## Configuration

### Cache Location

```bash
# Default cache location
~/.kopi/cache.json

# Override via environment variable
export KOPI_HOME=/custom/path
# Cache will be at: /custom/path/cache.json
```

### Cache Settings

The cache behavior can be configured in `~/.kopi/config.toml`:

```toml
[cache]
# Metadata source preferences
metadata_sources = ["foojay", "precached", "custom"]

# Custom metadata source URL (optional)
custom_metadata_url = "https://example.com/jdk-metadata.json"
```

## Implementation Details

### Cache Storage

The cache uses atomic file operations to ensure consistency:

```rust
// Atomic save operation
// 1. Write to temporary file
// 2. Atomic rename to final location
// 3. Clean up any failed attempts
```

### Version Matching

The cache supports flexible version patterns:

```rust
// Exact version match
cache.lookup(&distribution, "21.0.2", arch, os, None, None)

// Pattern matching
version.matches_pattern("21.*")  // Matches any 21.x version
version.matches_pattern("21")    // Matches major version
```

### Distribution Synonyms

Common distribution aliases are automatically resolved:

```rust
// These all resolve to the same distribution:
"temurin", "adoptopenjdk", "adopt" → Eclipse Temurin
"corretto", "amazon" → Amazon Corretto
"microsoft", "ms" → Microsoft Build of OpenJDK
```

## Troubleshooting

### Cache Corruption

```bash
# If cache is corrupted, clear and refresh
kopi cache clear
kopi cache refresh
```

### Missing Versions

```bash
# If a version is missing, force refresh
kopi cache refresh

# Check if version exists
kopi cache search 21.0.2
```

### Debug Cache Operations

```bash
# Enable debug logging
export RUST_LOG=kopi=debug
kopi cache search 21
```

## Best Practices

1. **Regular Updates**: Refresh cache periodically for latest versions
2. **Offline Usage**: Cache contains all metadata needed for offline operation
3. **Version Verification**: Use `cache search` to verify version availability
4. **Distribution Names**: Use canonical distribution names for consistency

## Next Steps

- [How Kopi Works](how-it-works.md) - System overview
- [Version Management](../reference/version-management.md) - Version patterns
- [Commands Reference](../reference/commands.md#cache) - Cache commands
