# Managing JDK Versions

Learn how to effectively manage multiple JDK versions with Kopi.

## Version Specification

### Version Formats

Kopi supports flexible version specifications:

```bash
# Major version only (installs latest patch)
kopi install 21

# Distribution with major version
kopi install temurin@21

# Exact version
kopi install temurin@21.0.2+13.0.LTS

# Version range (finds best match)
kopi install "corretto@>=17.0.9"
```

### Distribution Selection

When no distribution is specified, Kopi selects based on:

1. Platform availability
2. LTS status
3. Vendor reliability
4. Community adoption

Default preference order:
1. Eclipse Temurin
2. Amazon Corretto
3. Azul Zulu
4. Microsoft OpenJDK

## Version Scopes

### Global Version

Set the system-wide default JDK:

```bash
# Set global version
kopi use 21

# View global version
kopi current --global

# Reset to system JDK
kopi use system
```

### Project Version

Pin JDK version for a project:

```bash
# Pin current JDK
kopi pin

# Pin specific version
kopi pin 17

# Pin with distribution
kopi pin temurin@17

# Remove project pin
rm .kopi-version
```

### Shell Version

Temporary override for current shell:

```bash
# Set shell version
kopi shell 11

# Verify override
java --version

# Clear override
kopi shell --unset
```

## Version Discovery

### Listing Versions

```bash
# List all installed versions
kopi list

# List with details
kopi list --verbose

# Filter by distribution
kopi list --distribution temurin
```

### Searching Available Versions

```bash
# Search all available
kopi search

# Search by major version
kopi search 21

# Search by distribution
kopi search graalvm

# Search LTS versions only
kopi search --lts-only
```

## Version Resolution

### Resolution Order

Kopi resolves versions in this priority:

1. Environment variable (`KOPI_VERSION`)
2. Shell override (`kopi shell`)
3. Project version file (`.kopi-version` or `.java-version`)
4. Parent directory version files (recursive)
5. Global default (`kopi use`)
6. System JDK

### Version File Formats

#### Kopi Format (.kopi-version)

```bash
# Distribution@version
temurin@21
corretto@17.0.9
graalvm@21.0.1

# Version only (uses default distribution)
21
17.0.9
```

#### Compatibility Format (.java-version)

```bash
# Simple version
17
21

# With distribution (some tools support this)
temurin-17
corretto-21
```

## Advanced Usage

### Version Aliases

Create shortcuts for commonly used versions:

```bash
# Create alias (in ~/.kopi/config.toml)
[aliases]
lts = "temurin@21"
latest = "temurin@22"
graal = "graalvm@21"

# Use alias
kopi use lts
kopi pin latest
```

### Multiple JDK Installations

Install multiple versions of the same major version:

```bash
# Install different distributions
kopi install temurin@21
kopi install corretto@21
kopi install graalvm@21

# Switch between them
kopi use temurin@21
kopi use graalvm@21
```

### Version Constraints

Use semantic versioning constraints:

```bash
# Minimum version
kopi install ">=21.0.1"

# Version range
kopi install ">=17.0.0 <18.0.0"

# Specific patch level
kopi install "~21.0.2"  # Allows 21.0.x where x >= 2
```

## Best Practices

1. **Always pin versions** in production projects
2. **Use LTS versions** for stability
3. **Specify distributions** explicitly in CI/CD
4. **Document version requirements** in README
5. **Test with multiple JDKs** using shell overrides

## Troubleshooting

### Version Not Found

```bash
# Update metadata cache
kopi cache update

# Search with different criteria
kopi search
```

### Version Conflicts

```bash
# Check version resolution
kopi current --verbose

# Clear shell override
kopi shell --unset

# Verify environment
env | grep JAVA
env | grep KOPI
```

## Next Steps

- [Working with Projects](projects.md) - Project configuration
- [JDK Distributions](distributions.md) - Understanding distributions
- [Configuration](../reference/configuration.md) - Advanced configuration