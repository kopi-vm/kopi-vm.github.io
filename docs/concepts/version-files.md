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
```

**Examples:**

```bash
# Latest patch of Temurin 21
temurin@21

# Specific version of Corretto
corretto@17.0.9.8.1

# GraalVM with exact version
graalvm@21.0.1+12.1

# Just major version (auto-selects distribution)
21
```

### .java-version (Compatibility Format)

For compatibility with other tools (jEnv, etc.):

```bash
# Simple version number
17
21
11.0.2

# Some tools support distribution prefix
temurin-17
corretto-21
```

**Kopi's interpretation:**
- Reads numeric versions
- Auto-selects best distribution
- Maintains compatibility with jEnv

## Version File Priority

When multiple version files exist, Kopi uses this priority:

1. `.kopi-version` (highest priority)
2. `.java-version`
3. Parent directory files (recursive)
4. Global configuration

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
# Pin current JDK version
kopi pin

# Pin specific version
kopi pin 21

# Pin with distribution
kopi pin temurin@21
```

### Manual Creation

```bash
# Create .kopi-version
echo "temurin@21" > .kopi-version

# Create .java-version
echo "17" > .java-version

# Using heredoc for complex versions
cat > .kopi-version << EOF
graalvm@21.0.1+12.1
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
temurin@21.0.2+13.0.LTS
corretto@17.0.9.8.1
graalvm@21.0.1+12.1
```

### Version Constraints

```bash
# Minimum version (future feature)
>=21.0.1

# Version range (future feature)
>=17.0.0 <18.0.0

# Latest LTS (future feature)
lts

# Latest version (future feature)
latest
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

## Best Practices

### 1. Always Specify Distribution

```bash
# Good - explicit and reproducible
temurin@21
corretto@17.0.9

# Avoid - may vary between systems
21
17
```

### 2. Use Exact Versions for Production

```bash
# Good for production
temurin@21.0.2+13.0.LTS

# Good for development
temurin@21
```

### 3. Document Version Requirements

```markdown
<!-- In README.md -->
## Prerequisites

This project requires JDK 21 (Eclipse Temurin).

The exact version is specified in `.kopi-version`:
- Production: `temurin@21.0.2+13.0.LTS`
- Development: Any Temurin 21.x version
```

### 4. Commit Version Files

```bash
# Always commit version files
git add .kopi-version
git commit -m "Set JDK version to Temurin 21"
```

### 5. Validate in CI/CD

```yaml
# In CI pipeline
- name: Validate JDK version
  run: |
    expected=$(cat .kopi-version)
    actual=$(kopi current)
    if [ "$actual" != "$expected" ]; then
      echo "Version mismatch!"
      exit 1
    fi
```

## Migration from Other Formats

### From .sdkmanrc

```bash
# SDKMAN! format
java=17.0.2-tem

# Convert to Kopi
temurin@17.0.2
```

### From .tool-versions

```bash
# asdf format
java adoptopenjdk-17.0.2

# Convert to Kopi
temurin@17.0.2
```

### From .nvmrc Style

```bash
# Node.js style version file
17

# Works as-is with Kopi (.java-version)
17
```

## Troubleshooting

### Version File Not Detected

```bash
# Check current directory
pwd
ls -la .kopi-version .java-version

# Check version resolution
kopi current --verbose

# Verify file contents
cat .kopi-version
```

### Wrong Version Active

```bash
# Check for overrides
env | grep KOPI_VERSION

# Check shell override
kopi shell --status

# Clear overrides
unset KOPI_VERSION
kopi shell --unset
```

### Invalid Version Format

```bash
# Validate version file
kopi validate .kopi-version

# Common issues:
# - Windows line endings (use dos2unix)
# - Extra whitespace (trim with sed)
# - Invalid distribution name
```

## Advanced Features

### Dynamic Version Files

Generate version files based on environment:

```bash
#!/bin/bash
# generate-version.sh

if [ "$ENV" = "production" ]; then
    echo "temurin@21.0.2+13.0.LTS" > .kopi-version
else
    echo "temurin@22" > .kopi-version
fi
```

### Version File Templates

Use templates for consistency:

```bash
# .kopi-version.template
${DISTRIBUTION}@${VERSION}

# Generate from template
envsubst < .kopi-version.template > .kopi-version
```

### Conditional Versions

Script-based version selection:

```bash
#!/bin/bash
# Set JDK based on branch

branch=$(git branch --show-current)
case "$branch" in
    main|master)
        kopi pin temurin@21
        ;;
    develop)
        kopi pin temurin@22
        ;;
    release/*)
        kopi pin temurin@21.0.2+13.0.LTS
        ;;
esac
```

## Next Steps

- [Shims](shims.md) - How shims work
- [Metadata System](metadata.md) - JDK discovery
- [Working with Projects](../guide/projects.md) - Project setup