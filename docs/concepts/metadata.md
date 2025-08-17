# Metadata System

Understanding how Kopi discovers and manages JDK information.

## Overview

Kopi's metadata system provides comprehensive information about available JDK distributions, versions, and download locations. It uses a hybrid approach combining pre-generated metadata files for speed with the Foojay API for real-time updates.

## Architecture

```
┌────────────────────────────────────────────┐
│            Metadata Sources                 │
├──────────────┬───────────────┬─────────────┤
│  Pre-cached  │   Foojay API  │   Local     │
│   Metadata   │   (Real-time) │   Files     │
└──────┬───────┴───────┬───────┴─────────────┘
       │               │
       ▼               ▼
┌────────────────────────────────────────────┐
│           Metadata Aggregator               │
│         (Merge, dedupe, validate)           │
└──────────────────┬─────────────────────────┘
                   ▼
┌────────────────────────────────────────────┐
│            Local Cache                      │
│        ~/.kopi/cache/metadata.json          │
└────────────────────────────────────────────┘
```

## Metadata Sources

### 1. Pre-generated Metadata

Fast, offline-capable metadata hosted at kopi-vm.github.io:

```json
{
  "distributions": [
    {
      "name": "temurin",
      "vendor": "Eclipse Foundation",
      "versions": [
        {
          "version": "21.0.2+13.0.LTS",
          "url": "https://github.com/adoptium/temurin21-binaries/...",
          "checksum": "sha256:abc123...",
          "size": 195000000
        }
      ]
    }
  ]
}
```

**Advantages:**

- Fast access (CDN-hosted)
- Offline capability
- Reduced API calls
- Predictable structure

### 2. Foojay API

Real-time JDK information from disco.foojay.io:

```rust
// API endpoint
const FOOJAY_API: &str = "https://api.foojay.io/disco/v3.0";

// Query structure
struct FoojayQuery {
    version: String,
    distribution: Option<String>,
    architecture: String,
    operating_system: String,
    archive_type: String,
}
```

**Advantages:**

- Always up-to-date
- Comprehensive coverage
- Official source
- New releases immediately available

### 3. Local Metadata Files

User-provided or custom metadata:

```toml
# ~/.kopi/metadata/custom.toml
[[distributions]]
name = "custom-jdk"
vendor = "Internal"

[[distributions.versions]]
version = "17.0.0-internal"
url = "https://internal.company.com/jdk/17.tar.gz"
checksum = "sha256:def456..."
```

## Metadata Structure

### Distribution Information

```rust
#[derive(Serialize, Deserialize)]
struct Distribution {
    name: String,              // "temurin", "corretto", etc.
    vendor: String,            // "Eclipse Foundation", "Amazon"
    description: String,       // Human-readable description
    homepage: String,          // Distribution website
    license: String,           // "GPL-2.0 WITH Classpath-exception-2.0"
    supported_platforms: Vec<Platform>,
    versions: Vec<Version>,
}
```

### Version Information

```rust
#[derive(Serialize, Deserialize)]
struct Version {
    version: String,           // "21.0.2+13.0.LTS"
    major: u32,               // 21
    minor: u32,               // 0
    patch: u32,               // 2
    build: Option<String>,    // "13.0.LTS"
    lts: bool,                // true for LTS versions
    url: String,              // Download URL
    checksum: String,         // SHA256 checksum
    size: u64,                // File size in bytes
    release_date: DateTime,   // Release date
    end_of_life: Option<DateTime>,
}
```

### Platform Information

```rust
#[derive(Serialize, Deserialize)]
struct Platform {
    os: OperatingSystem,      // Linux, macOS, Windows
    arch: Architecture,        // x64, aarch64, x86
    libc: Option<LibC>,       // glibc, musl
}
```

## Metadata Operations

### 1. Fetching Metadata

```rust
async fn fetch_metadata() -> Result<Metadata> {
    // Try pre-cached first (fastest)
    if let Ok(cached) = fetch_precached().await {
        return Ok(cached);
    }

    // Fall back to Foojay API
    if let Ok(api_data) = fetch_foojay().await {
        return Ok(api_data);
    }

    // Use local cache as last resort
    load_local_cache()
}
```

### 2. Caching Strategy

```rust
struct CacheStrategy {
    // Cache validity periods
    const PRECACHED_TTL: Duration = Duration::hours(24);
    const API_TTL: Duration = Duration::hours(6);
    const OFFLINE_TTL: Duration = Duration::days(30);

    fn should_refresh(&self, cache: &Cache) -> bool {
        match cache.source {
            Source::PreCached => cache.age() > Self::PRECACHED_TTL,
            Source::API => cache.age() > Self::API_TTL,
            Source::Offline => false,  // Don't refresh in offline mode
        }
    }
}
```

### 3. Metadata Merging

```rust
fn merge_metadata(sources: Vec<Metadata>) -> Metadata {
    let mut merged = Metadata::new();

    for source in sources {
        for dist in source.distributions {
            // Merge versions, preferring newer
            merged.add_or_update(dist);
        }
    }

    // Deduplicate and sort
    merged.deduplicate();
    merged.sort_by_version();

    merged
}
```

## Search and Query

### Version Search

```rust
impl Metadata {
    fn search_versions(&self, query: &str) -> Vec<&Version> {
        self.versions
            .iter()
            .filter(|v| {
                v.version.contains(query) ||
                v.major.to_string() == query
            })
            .collect()
    }

    fn find_best_match(&self, spec: &VersionSpec) -> Option<&Version> {
        self.versions
            .iter()
            .filter(|v| spec.matches(v))
            .max_by_key(|v| (&v.lts, &v.major, &v.minor, &v.patch))
    }
}
```

### Platform Filtering

```rust
fn filter_by_platform(&self, platform: &Platform) -> Vec<&Version> {
    self.versions
        .iter()
        .filter(|v| v.supports_platform(platform))
        .collect()
}
```

## Metadata Files Organization

### Directory Structure

```
~/.kopi/cache/
├── metadata.json          # Merged metadata cache
├── sources/
│   ├── precached.json    # From kopi-vm.github.io
│   ├── foojay.json       # From Foojay API
│   └── custom.json       # User-defined
├── checksums/
│   ├── temurin-21.sha256
│   └── corretto-17.sha256
└── timestamps.json        # Cache timestamps
```

### Platform-Specific Metadata

```
kopi-vm.github.io/metadata/
├── index.json                      # Platform index
├── linux-x64-glibc/
│   ├── temurin.json
│   ├── corretto.json
│   └── graalvm.json
├── macos-aarch64-libc/
│   ├── temurin.json
│   └── corretto.json
└── windows-x64-c_std_lib/
    ├── temurin.json
    └── microsoft.json
```

## Performance Optimization

### 1. Lazy Loading

```rust
struct LazyMetadata {
    distributions: OnceCell<Vec<Distribution>>,
    index: OnceCell<HashMap<String, usize>>,
}

impl LazyMetadata {
    fn get_distribution(&self, name: &str) -> Option<&Distribution> {
        let index = self.index.get_or_init(|| self.build_index());
        let distributions = self.distributions.get_or_init(|| self.load());

        index.get(name).and_then(|&i| distributions.get(i))
    }
}
```

### 2. Incremental Updates

```rust
async fn update_metadata() -> Result<()> {
    let current = load_local_cache()?;
    let updates = fetch_updates_since(current.last_updated).await?;

    let merged = merge_incremental(current, updates);
    save_cache(merged)?;

    Ok(())
}
```

### 3. Compression

```rust
fn save_compressed(metadata: &Metadata) -> Result<()> {
    let json = serde_json::to_string(metadata)?;
    let compressed = compress_gzip(json.as_bytes())?;

    fs::write(cache_path(), compressed)?;
    Ok(())
}
```

## Offline Support

### Offline Mode Detection

```rust
fn is_offline() -> bool {
    // Check network connectivity
    !can_reach("https://api.foojay.io") &&
    !can_reach("https://kopi-vm.github.io")
}
```

### Offline Fallback

```rust
fn get_metadata() -> Result<Metadata> {
    if is_offline() {
        eprintln!("Working in offline mode");
        return load_local_cache()
            .context("No cached metadata available offline");
    }

    fetch_online_metadata()
}
```

## Metadata Validation

### Checksum Verification

```rust
fn verify_metadata(data: &[u8], checksum: &str) -> Result<()> {
    let computed = sha256(data);
    if computed != checksum {
        return Err("Metadata checksum mismatch");
    }
    Ok(())
}
```

### Schema Validation

```rust
fn validate_schema(json: &str) -> Result<Metadata> {
    let metadata: Metadata = serde_json::from_str(json)?;

    // Validate required fields
    for dist in &metadata.distributions {
        ensure!(!dist.name.is_empty(), "Distribution name required");
        ensure!(!dist.versions.is_empty(), "Versions required");

        for version in &dist.versions {
            ensure!(version.url.starts_with("https://"), "HTTPS required");
            ensure!(version.size > 0, "Invalid file size");
        }
    }

    Ok(metadata)
}
```

## Custom Metadata

### Adding Custom Sources

```toml
# ~/.kopi/config.toml
[[metadata.sources]]
name = "internal"
url = "https://internal.company.com/kopi/metadata.json"
priority = 1

[[metadata.sources]]
name = "local"
path = "/shared/kopi/metadata.json"
priority = 2
```

### Creating Custom Metadata

```bash
# Generate metadata from local JDKs
kopi metadata generate --from /opt/jdks --output custom.json

# Validate metadata file
kopi metadata validate custom.json

# Import metadata
kopi metadata import custom.json
```

## Troubleshooting

### Metadata Issues

```bash
# Check metadata status
kopi cache status

# Force metadata update
kopi cache update --force

# Clear corrupted cache
kopi cache clear

# Validate metadata integrity
kopi doctor --check metadata
```

### Debug Mode

```bash
# Enable metadata debug logging
KOPI_DEBUG_METADATA=1 kopi search

# Trace metadata sources
KOPI_TRACE=metadata kopi search 21
```

## API Rate Limiting

### Rate Limit Handling

```rust
struct RateLimiter {
    requests_per_hour: u32,
    current_requests: AtomicU32,
    reset_time: Instant,
}

impl RateLimiter {
    async fn request(&self) -> Result<Response> {
        if self.is_rate_limited() {
            self.wait_for_reset().await;
        }

        self.increment_counter();
        make_request().await
    }
}
```

## Next Steps

- [Caching](caching.md) - Cache management
- [How Kopi Works](how-it-works.md) - System overview
- [Configuration](../reference/configuration.md) - Metadata configuration
