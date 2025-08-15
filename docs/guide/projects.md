# Working with Projects

Learn how to configure JDK versions for your projects and ensure consistency across your team.

## Project Configuration

### Creating Version Files

Set a JDK version for your project:

```bash
# Navigate to project root
cd my-project

# Set specific version
kopi local 21

# Set with distribution
kopi local temurin@21.0.2
```

This creates a `.kopi-version` file in your project root.

### Version File Formats

#### Kopi Native Format (.kopi-version)

```bash
# Recommended format
temurin@21

# Version only (uses default distribution)
21

# Exact version
temurin@21.0.2+13.0.LTS
```

#### Compatibility Format (.java-version)

For compatibility with other tools:

```bash
# Simple version
17

# Some tools support distribution prefix
temurin-17
corretto-21
```

## Project Hierarchy

### Version Inheritance

Kopi searches for version files up the directory tree:

```
/home/user/
├── .kopi-version (21)
└── projects/
    ├── project-a/
    │   └── .kopi-version (17)  # Uses JDK 17
    └── project-b/              # Inherits JDK 21
        └── submodule/
            └── .kopi-version (11)  # Uses JDK 11
```

### Monorepo Support

Configure different JDK versions for different parts of a monorepo:

```
monorepo/
├── .kopi-version (21)          # Default for monorepo
├── services/
│   ├── legacy-service/
│   │   └── .kopi-version (8)   # Legacy service needs JDK 8
│   └── modern-service/
│       └── .kopi-version (21)  # Modern service uses JDK 21
└── libraries/
    └── .kopi-version (17)       # Libraries target JDK 17
```

## Team Collaboration

### Sharing Configuration

Commit version files to version control:

```bash
# Add to Git
git add .kopi-version
git commit -m "Set JDK version to 21"
```

### Documentation

Document JDK requirements in README:

```markdown
## Prerequisites

This project requires JDK 21. Install using Kopi:

\`\`\`bash
kopi install $(cat .kopi-version)
\`\`\`
```

### Automated Setup

Create setup scripts for team members:

```bash
#!/bin/bash
# setup.sh

# Install Kopi if not present
if ! command -v kopi &> /dev/null; then
    echo "Installing Kopi..."
    brew install kopi-vm/tap/kopi
fi

# Install required JDK
if [ -f .kopi-version ]; then
    VERSION=$(cat .kopi-version)
    echo "Installing JDK $VERSION..."
    kopi install $VERSION
fi
```

## Build Tool Integration

### Maven

Configure Maven to use Kopi-managed JDK:

```xml
<properties>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
</properties>
```

### Gradle

Configure Gradle to use Kopi-managed JDK:

```gradle
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}
```

### IDE Configuration

Most IDEs automatically detect the JDK from PATH, which Kopi manages through shims.

## CI/CD Integration

### GitHub Actions

```yaml
name: Build
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Install Kopi
        run: |
          curl -fsSL https://kopi-vm.github.io/install.sh | bash
          echo "$HOME/.kopi/shims" >> $GITHUB_PATH
      
      - name: Install JDK
        run: |
          kopi install $(cat .kopi-version)
      
      - name: Build
        run: ./gradlew build
```

### GitLab CI

```yaml
build:
  image: ubuntu:latest
  before_script:
    - apt-get update && apt-get install -y curl
    - curl -fsSL https://kopi-vm.github.io/install.sh | bash
    - export PATH="$HOME/.kopi/shims:$PATH"
    - kopi install $(cat .kopi-version)
  script:
    - ./gradlew build
```

## Best Practices

### Version Setting

1. **Always set versions** in production projects using `kopi local`
2. **Use exact versions** for reproducible builds
3. **Include distribution** to avoid ambiguity

### Version File Placement

1. **Project root** - One version for entire project
2. **Module level** - Different versions for different modules
3. **Test directories** - Specific version for tests

### Documentation

1. **Document JDK requirements** in README
2. **Include setup instructions** for new developers
3. **Note any JDK-specific features** used

## Migration

### From SDKMAN

```bash
# Convert .sdkmanrc to .kopi-version
if [ -f .sdkmanrc ]; then
    grep java .sdkmanrc | cut -d'=' -f2 > .kopi-version
fi
```

### From jEnv

```bash
# Convert .java-version (jEnv uses same format)
# Kopi reads .java-version automatically
```

### From Jabba

```bash
# Convert .jabbarc to .kopi-version
if [ -f .jabbarc ]; then
    cat .jabbarc | jq -r '.jdk' > .kopi-version
fi
```

## Troubleshooting

### Version Not Installing

```bash
# Check version availability
kopi search $(cat .kopi-version)

# Update metadata cache
kopi cache update

# Try without distribution
kopi install 21
```

### Wrong Version Active

```bash
# Check version resolution
kopi current --verbose

# Verify version file
cat .kopi-version

# Check for overrides
env | grep KOPI_JAVA_VERSION
```

## Next Steps

- [CI/CD Integration](ci-cd.md) - Automated builds
- [Shell Integration](shell-integration.md) - Shell configuration
- [Migration Guide](migration.md) - Migrating from other tools