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
kopi install temurin@21.0.2+13

# Latest version
kopi install latest
kopi install temurin@latest

# Package type (JRE or JDK)
kopi install jre@21
kopi install jdk@temurin@21
```

### Distribution Selection

When no distribution is specified, Kopi uses the default distribution configured in `~/.kopi/config.toml`:

```toml
# Default configuration
default_distribution = "temurin"
```

You can change the default distribution by editing the configuration file or using environment variables.

## Version Scopes

### Global Version

Set the system-wide default JDK:

```bash
# Set global version
kopi global 21

# View current version
kopi current

# Reset to system JDK
kopi global system
```

### Project Version

Set JDK version for a project:

```bash
# Set specific version
kopi local 17

# Set with distribution
kopi local temurin@17

# Remove project version
rm .kopi-version
```

### Shell Version

Launch a new shell with a specific JDK:

```bash
# Launch new shell with JDK 11
kopi shell 11

# Or use the alias
kopi use 11

# The new shell will have JAVA_HOME and PATH configured
java --version
```

Note: `kopi shell` and `kopi use` spawn a new subshell with the specified JDK. Exit the shell to return to your previous environment.

## Version Discovery

### Listing Versions

```bash
# List all installed versions
kopi list

# Or use the alias
kopi ls
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

1. Environment variable (`KOPI_JAVA_VERSION`)
2. Project version file (`.kopi-version` takes precedence over `.java-version`)
3. Parent directory version files (recursive up to root, checking `.kopi-version` first, then `.java-version`)
4. Global default (`~/.kopi/version` set by `kopi global`)

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

# Package type prefix (JRE or JDK)
jre@21
jdk@temurin@21
```

#### Compatibility Format (.java-version)

```bash
# Simple version only
17
21
11.0.2

# Note: .java-version does not support distribution@version format
```

## Advanced Usage

### Multiple JDK Installations

Install multiple versions of the same major version:

```bash
# Install different distributions
kopi install temurin@21
kopi install corretto@21
kopi install graalvm@21

# Switch between them globally
kopi global temurin@21
kopi global graalvm@21

# Or launch a new shell with specific JDK
kopi shell temurin@21
kopi use graalvm@21  # alias for shell
```

## Best Practices

1. **Always set versions** in production projects using `kopi local`
2. **Use LTS versions** for stability
3. **Specify distributions** explicitly in CI/CD
4. **Document version requirements** in README
5. **Test with multiple JDKs** using `kopi shell` or `kopi use` to launch temporary test shells

## Troubleshooting

### Version Not Found

```bash
# Update metadata cache
kopi cache refresh

# Search with different criteria
kopi search
```

### Version Conflicts

```bash
# Check current version
kopi current

# Verify environment
env | grep JAVA
env | grep KOPI

# Clear environment override
unset KOPI_JAVA_VERSION
```

## Next Steps

- [Working with Projects](projects.md) - Project configuration
- [JDK Distributions](distributions.md) - Understanding distributions
- [Configuration](../reference/configuration.md) - Advanced configuration
