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

# JRE package type (JDK is default)
kopi install jre@21
kopi install jre@temurin@21

# JavaFX bundled versions (+fx suffix)
kopi install 21+fx
kopi install zulu@21+fx
kopi install jre@liberica@17+fx
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

# Set with distribution
kopi global temurin@21

# View current version
kopi current
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
2. Project version files searched from current directory up to root:
   - `.kopi-version` takes precedence over `.java-version` in the same directory
   - Search stops when the first version file is found
3. Global default (`~/.kopi/version` set by `kopi global`)

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

# JRE package type (JDK is default)
jre@temurin@21   # JRE with distribution
jre@21           # JRE without distribution

# JavaFX bundled versions (+fx suffix)
21+fx            # JDK with JavaFX
zulu@17+fx       # Zulu JDK with JavaFX
jre@liberica@21+fx  # Liberica JRE with JavaFX
```

#### Compatibility Format (.java-version)

```bash
# Simple version numbers only
17
21
11.0.2

# Note: .java-version does not support:
# - Distribution specification (no temurin@21)
# - Package type specification (no jre@ prefix)
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

### JavaFX Support

Install JDKs with JavaFX bundled using the `+fx` suffix:

```bash
# Install JDK with JavaFX
kopi install 21+fx
kopi install zulu@17+fx
kopi install liberica@21.0.2+fx

# Set JavaFX version for project
kopi local zulu@21+fx

# Use JavaFX version in new shell
kopi shell liberica@17+fx
```

**Available distributions with JavaFX:**

- **Zulu** (Azul) - Provides JavaFX builds
- **Liberica** (BellSoft) - Full edition includes JavaFX
- **Oracle** - Some versions include JavaFX

**Note:** Not all distributions provide JavaFX-bundled builds. If a JavaFX build is not available for your requested version, Kopi will show available alternatives.

## Next Steps

- [Working with Projects](projects.md) - Project configuration
- [Configuration](../reference/configuration.md) - Advanced configuration
