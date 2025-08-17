# First Steps

Learn the essential Kopi commands and concepts to manage your JDK versions effectively.

## Understanding Kopi

Kopi manages JDK versions at three levels:

1. **Global** - Default JDK version for your system
2. **Project** - JDK version for a specific project directory
3. **Shell** - Temporary JDK version for current shell session

## Essential Commands

### Viewing JDK Information

```bash
# Show current JDK version
kopi current

# List installed JDKs
kopi list

# Search available JDKs
kopi search

# Get detailed JDK information
kopi which java
```

### Installing JDKs

```bash
# Install with automatic selection
kopi install 21  # Installs recommended distribution

# Install specific distribution
kopi install corretto@21
kopi install temurin@17
kopi install graalvm@21

# Install exact version
kopi install temurin@21.0.2+13
```

### Managing Versions

```bash
# Set global default
kopi global 21

# Set project version
kopi local 17

# Temporary shell override
kopi shell 11
```

### Removing JDKs

```bash
# Uninstall a JDK
kopi uninstall temurin@17

# List installed JDKs to see what can be removed
kopi list

# Uninstall specific versions
kopi uninstall corretto@11
```

## Working with Projects

### Creating Version Files

Kopi supports two version file formats:

```bash
# Native Kopi format (.kopi-version)
echo "temurin@21" > .kopi-version

# Compatibility format (.java-version)
echo "17" > .java-version
```

### Version File Priority

When multiple version files exist, Kopi uses this priority:

1. `.kopi-version` (highest priority)
2. `.java-version`
3. Global default

## Shell Integration

### Automatic Switching

Once shell integration is configured, Kopi automatically switches JDK versions when you:

- Enter a directory with a version file
- Run Java commands
- Start a new shell session

### Manual Override

```bash
# Temporary override for current shell
kopi shell 11

# Run single command with different JDK
KOPI_JAVA_VERSION=17 java --version
```

## Tips and Best Practices

1. **Use version files** - Always use `kopi local` to set versions in projects for consistency
2. **Specify distributions** - Be explicit about which JDK distribution you need
3. **Regular updates** - Keep your metadata cache updated with `kopi cache refresh`
4. **Clean up** - Regularly check installed JDKs with `kopi list` and remove unused ones with `kopi uninstall`

## Next Steps

- [Managing JDK Versions](../guide/managing-versions.md) - Advanced version management
- [Working with Projects](../guide/projects.md) - Project configuration
- [Shell Integration](../guide/shell-integration.md) - Configure your shell
