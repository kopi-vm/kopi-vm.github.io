# Version Files

Understanding how Kopi uses version files to manage JDK versions in projects.

## Overview

Version files tell Kopi which JDK version to use for a specific project or directory. They enable automatic version switching and ensure consistency across development environments.

## File Formats

### .kopi-version (Native Format)

The native Kopi format provides full control over distribution and version:

```bash
# Format: [package@]distribution@version[+fx]
temurin@21
corretto@17.0.9
graalvm@21.0.1

# Version only (uses default distribution)
21
17.0.9

# JRE package type (JDK is default):
jre@temurin@21      # JRE with distribution
jre@17              # JRE without distribution

# JavaFX bundled versions (add +fx suffix):
21+fx               # JDK with JavaFX
zulu@17+fx          # Zulu JDK with JavaFX
jre@liberica@21+fx  # Liberica JRE with JavaFX
```

**Examples:**

```bash
# Latest patch of Temurin 21
temurin@21

# Specific version of Corretto
corretto@17.0.9.8.1

# GraalVM with exact version
graalvm@21.0.1

# Just major version (auto-selects distribution)
21

# JDK with JavaFX bundled
zulu@21+fx
liberica@17.0.9+fx
```

### .java-version (Compatibility Format)

For compatibility with other tools (jEnv, etc.):

```bash
# Simple version numbers only
17
21
11.0.2
```

**Important notes:**

- Only supports numeric version formats
- Does not support distribution specification (no `temurin@21` format)
- Does not support package type specification (no `jre@` prefix)
- Auto-selects distribution based on availability and configuration

## Version Resolution Priority

Kopi resolves the JDK version to use in this order:

1. `KOPI_JAVA_VERSION` environment variable (highest priority)
2. `.kopi-version` file (searched from current directory up to root)
3. `.java-version` file (searched from current directory up to root)
4. Global default (`~/.kopi/version`)

**Note:** When both `.kopi-version` and `.java-version` exist in the same directory, `.kopi-version` takes precedence.

```text
project/
├── .kopi-version (temurin@21)  # Takes priority
├── .java-version (17)           # Ignored when .kopi-version exists
└── src/
    └── Main.java                # Uses Temurin JDK 21
```

## Creating Version Files

### Using Commands

```bash
# Pin specific version (creates .kopi-version)
kopi local 21
# or using the alias
kopi pin 21

# Pin with distribution
kopi local temurin@21

# Pin JRE instead of JDK
kopi local jre@temurin@21
# or without distribution
kopi local jre@21
```

### Manual Creation

```bash
# Create .kopi-version
echo "temurin@21" > .kopi-version

# Create .java-version
echo "17" > .java-version

# Using heredoc for complex versions
cat > .kopi-version << EOF
graalvm@21.0.1
EOF
```

## Version Specification

### Basic Versions

```bash
# Major version only
21
17
11

# Major.minor
21.0
17.0
11.0

# Full version
21.0.2
17.0.9
11.0.21
```

### Distribution Specification

```bash
# Distribution with major version
temurin@21
corretto@17
graalvm@21

# Distribution with full version
temurin@21.0.2+13
corretto@17.0.9.8.1
graalvm@21.0.1

# JRE package type with distribution
jre@temurin@21
jre@corretto@17.0.9

# JRE package type without distribution
jre@21
jre@17.0.9
```

### JavaFX Support

The `+fx` suffix specifies that you want a JDK with JavaFX bundled:

```bash
# JavaFX with version only
21+fx               # Auto-selects distribution with JavaFX
17.0.9+fx

# JavaFX with specific distribution
zulu@21+fx          # Zulu includes JavaFX builds
liberica@17+fx      # Liberica Full includes JavaFX

# JavaFX with JRE package type
jre@21+fx           # JRE with JavaFX
jre@liberica@21+fx  # Liberica JRE with JavaFX
```

**Note:** Not all distributions provide JavaFX-bundled builds. Common distributions with JavaFX support include:

- Zulu (Azul)
- Liberica (BellSoft)
- Oracle (certain versions)

## Directory Hierarchy

### Inheritance

Version files are inherited from parent directories:

```text
workspace/
├── .kopi-version (21)              # Workspace default
├── project-a/
│   ├── .kopi-version (17)         # Override for project-a
│   └── submodule/
│       └── Main.java               # Uses JDK 17
└── project-b/
    └── Main.java                   # Uses JDK 21 (inherited)
```

### Search Behavior

Kopi searches for version files from the current directory up to the root:

1. In each directory, `.kopi-version` is checked before `.java-version`
2. If a version file is found, the search stops
3. If no version file is found, the search continues to the parent directory

```bash
# Example: Current directory is /home/user/projects/myapp/src/main/java

# Search progression:
/home/user/projects/myapp/src/main/java/
├── .kopi-version?  # Check first
└── .java-version?  # Check if no .kopi-version

/home/user/projects/myapp/src/main/
├── .kopi-version?  # Check first
└── .java-version?  # Check if no .kopi-version

/home/user/projects/myapp/
├── .kopi-version?  # Check first
└── .java-version?  # Check if no .kopi-version

# Continues up to root directory
# Finally checks ~/.kopi/version if no project file found
```

## Use Cases

### Monorepo Management

Structure for different JDK requirements:

```text
monorepo/
├── .kopi-version (21)              # Default for monorepo
├── services/
│   ├── legacy-api/
│   │   └── .kopi-version (8)      # Legacy service
│   ├── modern-api/
│   │   └── .kopi-version (21)     # Modern service
│   └── data-processor/
│       └── .kopi-version (17)     # Specific requirement
└── libraries/
    ├── core/
    │   └── .kopi-version (11)     # Minimum supported
    └── utils/
        └── .kopi-version (17)     # Using newer features
```

### Multi-Module Projects

Maven/Gradle multi-module setup:

```text
my-app/
├── .kopi-version (17)              # Project default
├── modules/
│   ├── core/
│   │   └── src/                   # Uses JDK 17
│   ├── legacy-support/
│   │   ├── .kopi-version (8)      # Needs JDK 8
│   │   └── src/
│   └── experimental/
│       ├── .kopi-version (21)     # Testing new features
│       └── src/
└── pom.xml
```

### Development vs Production

Different versions for different environments:

```text
project/
├── .kopi-version (21)              # Production version
├── dev/
│   └── .kopi-version (22)         # Development version
└── test/
    └── .kopi-version (21)         # Match production
```

## Next Steps

- [Shims](shims.md) - How shims work
- [Metadata System](metadata.md) - JDK discovery
- [Working with Projects](../guide/projects.md) - Project setup
