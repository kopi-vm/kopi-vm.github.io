# Version Files

Understanding how Kopi uses version files to manage JDK versions in projects.

## Overview

Version files tell Kopi which JDK version to use for a specific project or directory. They enable automatic version switching and ensure consistency across development environments.

## File Formats

### .kopi-version (Native Format)

The native Kopi format provides full control over distribution and version:

```bash
# Format: distribution@version
temurin@21
corretto@17.0.9
graalvm@21.0.1

# Version only (uses default distribution)
21
17.0.9

# JRE format: jre@distribution@version
jre@temurin@21
jre@corretto@17
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
```

### .java-version (Compatibility Format)

For compatibility with other tools (jEnv, etc.):

```bash
# Simple version number only
17
21
11.0.2
```

**Kopi's interpretation:**

- Reads numeric versions only
- Does not support distribution prefix in .java-version
- Auto-selects best distribution based on configuration

## Version File Priority

When multiple version files exist, Kopi uses this priority:

1. `KOPI_JAVA_VERSION` environment variable (highest priority)
2. `.kopi-version` in current directory
3. `.java-version` in current directory
4. Parent directory files (searched recursively up to root)
5. Global configuration (`~/.kopi/version`)

```
project/
├── .kopi-version (21)        # Takes priority
├── .java-version (17)        # Ignored when .kopi-version exists
└── src/
    └── Main.java             # Uses JDK 21
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

# Pin with JRE package type
kopi local jre@temurin@21
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
```

## Directory Hierarchy

### Inheritance

Version files are inherited from parent directories:

```
workspace/
├── .kopi-version (21)              # Workspace default
├── project-a/
│   ├── .kopi-version (17)         # Override for project-a
│   └── submodule/
│       └── Main.java               # Uses JDK 17
└── project-b/
    └── Main.java                   # Uses JDK 21 (inherited)
```

### Search Order

Kopi searches for version files from current directory up to root:

```bash
# Current directory: /home/user/projects/myapp/src/main/java

# Search order:
1. /home/user/projects/myapp/src/main/java/.kopi-version
2. /home/user/projects/myapp/src/main/java/.java-version
3. /home/user/projects/myapp/src/main/.kopi-version
4. /home/user/projects/myapp/src/main/.java-version
5. /home/user/projects/myapp/src/.kopi-version
6. /home/user/projects/myapp/src/.java-version
7. /home/user/projects/myapp/.kopi-version
8. /home/user/projects/myapp/.java-version
9. /home/user/projects/.kopi-version
10. /home/user/projects/.java-version
# ... continues to root
```

## Use Cases

### Monorepo Management

Structure for different JDK requirements:

```
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

```
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

```
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
