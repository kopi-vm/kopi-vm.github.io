# Caching

Understanding Kopi's caching mechanisms for optimal performance.

## Overview

Kopi uses multiple caching layers to provide fast JDK version management while minimizing network requests and disk I/O. The caching system balances performance with freshness of data.

## Cache Layers

```
┌─────────────────────────────────────────┐
│          Memory Cache (L1)               │
│         (In-process, < 1ms)              │
└────────────────┬────────────────────────┘
                 ▼
┌─────────────────────────────────────────┐
│          File Cache (L2)                 │
│     (~/.kopi/cache/, < 10ms)             │
└────────────────┬────────────────────────┘
                 ▼
┌─────────────────────────────────────────┐
│         Remote Cache (L3)                │
│   (CDN/API, 50-500ms)                    │
└─────────────────────────────────────────┘
```

## Cache Types

### 1. Metadata Cache

Stores JDK availability information:

```
~/.kopi/cache/
├── metadata.json           # Merged metadata
├── metadata.json.gz        # Compressed backup
├── metadata.timestamp      # Last update time
└── sources/
    ├── precached.json     # From CDN
    ├── foojay.json        # From API
    └── custom.json        # User-defined
```

**Cache Policy:**
- TTL: 6 hours (API), 24 hours (CDN)
- Size: ~2-5 MB uncompressed
- Update: On-demand or scheduled

### 2. Version Resolution Cache

Caches resolved versions for paths:

```rust
struct VersionCache {
    // Path -> Version mapping
    cache: HashMap<PathBuf, CachedVersion>,
}

struct CachedVersion {
    version: String,
    distribution: String,
    resolved_at: Instant,
    ttl: Duration,
}
```

**Cache Policy:**
- TTL: 1 second (for rapid directory changes)
- Size: < 1 KB per entry
- Scope: Per-process

### 3. Download Cache

Temporary storage for JDK downloads:

```
~/.kopi/downloads/
├── temurin-21.0.2.tar.gz.partial
├── temurin-21.0.2.tar.gz
└── checksums/
    └── temurin-21.0.2.sha256
```

**Cache Policy:**
- Retention: Until installation complete
- Cleanup: Automatic after success
- Resume: Supports partial downloads

### 4. Shim Cache

Caches shim execution results:

```rust
struct ShimCache {
    // Command -> JDK path mapping
    paths: HashMap<String, PathBuf>,
    // Recent resolutions
    resolutions: LruCache<String, Resolution>,
}
```

**Cache Policy:**
- TTL: 1 second
- Size: 100 entries (LRU)
- Scope: Per-shim execution

## Cache Operations

### Reading from Cache

```rust
impl Cache {
    fn get<T>(&self, key: &str) -> Option<T> {
        // L1: Memory cache
        if let Some(value) = self.memory.get(key) {
            if !value.is_expired() {
                return Some(value.data);
            }
        }
        
        // L2: File cache
        if let Some(value) = self.file.get(key) {
            if !value.is_expired() {
                // Promote to L1
                self.memory.insert(key, value.clone());
                return Some(value.data);
            }
        }
        
        // L3: Remote fetch required
        None
    }
}
```

### Writing to Cache

```rust
impl Cache {
    fn set<T>(&mut self, key: &str, value: T, ttl: Duration) {
        let entry = CacheEntry {
            data: value,
            expires_at: Instant::now() + ttl,
        };
        
        // Write to L1
        self.memory.insert(key, entry.clone());
        
        // Write to L2
        self.file.insert(key, entry);
        
        // Trigger background sync if needed
        self.schedule_sync();
    }
}
```

### Cache Invalidation

```rust
impl Cache {
    fn invalidate(&mut self, pattern: &str) {
        // Clear matching entries
        self.memory.retain(|k, _| !k.contains(pattern));
        self.file.retain(|k, _| !k.contains(pattern));
        
        // Mark for refresh
        self.needs_refresh.insert(pattern.to_string());
    }
    
    fn invalidate_all(&mut self) {
        self.memory.clear();
        self.file.clear();
        self.needs_refresh.clear();
    }
}
```

## Cache Management Commands

### Viewing Cache Status

```bash
# Show cache statistics
kopi cache status

# Output:
Cache Status:
  Metadata: 2.3 MB (updated 2 hours ago)
  Downloads: 0 B (empty)
  Total size: 2.3 MB
  
  Entries:
    - Distributions: 42
    - Versions: 1,827
    - Platforms: 15
```

### Updating Cache

```bash
# Update metadata cache
kopi cache update

# Force update (ignore TTL)
kopi cache update --force

# Update specific source
kopi cache update --source foojay
```

### Clearing Cache

```bash
# Clear all caches
kopi cache clear

# Clear specific cache
kopi cache clear --metadata
kopi cache clear --downloads

# Clear expired entries only
kopi cache prune
```

## Performance Optimization

### 1. Memory Cache

In-process caching for hot paths:

```rust
use once_cell::sync::Lazy;

static VERSION_CACHE: Lazy<Mutex<HashMap<String, String>>> = 
    Lazy::new(|| Mutex::new(HashMap::new()));

fn resolve_version_cached(path: &Path) -> String {
    let key = path.to_string_lossy().to_string();
    
    // Check cache
    if let Some(version) = VERSION_CACHE.lock().unwrap().get(&key) {
        return version.clone();
    }
    
    // Resolve and cache
    let version = resolve_version(path);
    VERSION_CACHE.lock().unwrap().insert(key, version.clone());
    
    version
}
```

### 2. Compression

Reduce disk usage and I/O:

```rust
fn save_compressed(data: &[u8], path: &Path) -> Result<()> {
    use flate2::write::GzEncoder;
    use flate2::Compression;
    
    let file = File::create(path)?;
    let mut encoder = GzEncoder::new(file, Compression::default());
    encoder.write_all(data)?;
    encoder.finish()?;
    
    Ok(())
}
```

### 3. Parallel Fetching

Update multiple cache sources concurrently:

```rust
async fn update_all_caches() -> Result<()> {
    let futures = vec![
        update_precached(),
        update_foojay(),
        update_custom(),
    ];
    
    let results = futures::future::join_all(futures).await;
    
    // Merge results
    let metadata = merge_results(results)?;
    save_cache(metadata).await
}
```

## Cache Configuration

### Configuration File

```toml
# ~/.kopi/config.toml

[cache]
# Maximum cache size in MB
max_size = 100

# Metadata cache TTL in hours
metadata_ttl = 6

# Version resolution cache TTL in seconds
resolution_ttl = 1

# Enable compression
compress = true

# Auto-update interval in hours (0 = disabled)
auto_update = 24

[cache.sources]
# Source priorities (lower = higher priority)
precached = 1
foojay = 2
custom = 3
```

### Environment Variables

```bash
# Override cache directory
export KOPI_CACHE_DIR=/tmp/kopi-cache

# Disable caching
export KOPI_NO_CACHE=1

# Force offline mode
export KOPI_OFFLINE=1

# Debug cache operations
export KOPI_DEBUG_CACHE=1
```

## Offline Mode

### Automatic Offline Detection

```rust
fn is_offline() -> bool {
    // Try to reach known endpoints
    let endpoints = [
        "https://api.foojay.io",
        "https://kopi-vm.github.io",
        "https://github.com",
    ];
    
    !endpoints.iter().any(|url| can_reach(url))
}
```

### Offline Behavior

```rust
impl Cache {
    fn get_offline(&self, key: &str) -> Option<Value> {
        // Only use local caches
        self.memory.get(key)
            .or_else(|| self.file.get(key))
            // Ignore expiration in offline mode
            .map(|entry| entry.data)
    }
}
```

## Cache Strategies

### 1. Write-Through

Updates cache and source simultaneously:

```rust
fn install_jdk(version: &str) -> Result<()> {
    // Download and install
    let jdk = download_jdk(version)?;
    install(jdk)?;
    
    // Update cache immediately
    cache.add_installed(version);
    cache.save()?;
    
    Ok(())
}
```

### 2. Write-Back

Batches cache updates:

```rust
struct WriteBackCache {
    pending: Vec<CacheOp>,
    flush_interval: Duration,
}

impl WriteBackCache {
    fn write(&mut self, op: CacheOp) {
        self.pending.push(op);
        
        if self.should_flush() {
            self.flush();
        }
    }
    
    fn flush(&mut self) {
        // Batch write all pending operations
        self.execute_batch(&self.pending);
        self.pending.clear();
    }
}
```

### 3. Read-Ahead

Prefetch likely needed data:

```rust
fn prefetch_related(version: &str) {
    thread::spawn(move || {
        // Prefetch related versions
        let major = extract_major(version);
        cache.prefetch(format!("{}.*", major));
        
        // Prefetch common tools
        cache.prefetch_tools(&["java", "javac", "jar"]);
    });
}
```

## Cache Metrics

### Monitoring Cache Performance

```rust
struct CacheMetrics {
    hits: AtomicU64,
    misses: AtomicU64,
    evictions: AtomicU64,
    size_bytes: AtomicU64,
}

impl CacheMetrics {
    fn hit_rate(&self) -> f64 {
        let hits = self.hits.load(Ordering::Relaxed) as f64;
        let total = hits + self.misses.load(Ordering::Relaxed) as f64;
        
        if total > 0.0 {
            hits / total
        } else {
            0.0
        }
    }
}
```

### Cache Statistics

```bash
# View cache metrics
kopi cache stats

# Output:
Cache Statistics:
  Hit rate: 92.3%
  Total hits: 1,234
  Total misses: 94
  Evictions: 12
  
  Size by type:
    Metadata: 2.1 MB
    Resolutions: 45 KB
    Downloads: 0 B
  
  Age:
    Oldest entry: 23 hours
    Newest entry: 2 minutes
```

## Troubleshooting

### Cache Corruption

```bash
# Verify cache integrity
kopi cache verify

# Repair corrupted cache
kopi cache repair

# Full reset
kopi cache clear --all
kopi cache update
```

### Performance Issues

```bash
# Analyze cache performance
kopi cache analyze

# Optimize cache
kopi cache optimize

# Adjust cache size
kopi config set cache.max_size 200
```

### Debug Cache Operations

```bash
# Enable cache debugging
export KOPI_DEBUG_CACHE=1
kopi search

# Trace cache hits/misses
export KOPI_TRACE_CACHE=1
kopi current
```

## Best Practices

1. **Regular Updates**: Schedule cache updates during off-hours
2. **Size Limits**: Set appropriate cache size limits
3. **Compression**: Enable for large metadata sets
4. **Offline Prep**: Update cache before going offline
5. **Monitoring**: Track cache hit rates and adjust TTLs

## Next Steps

- [How Kopi Works](how-it-works.md) - System overview
- [Performance](../reference/commands.md#cache) - Cache commands
- [Configuration](../reference/configuration.md) - Cache settings