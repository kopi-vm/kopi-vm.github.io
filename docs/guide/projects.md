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
temurin@21.0.2+13
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

```text
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

```text
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

This project requires JDK 21.

If you have Kopi installed, the required JDK will be automatically installed when you run any Java command in this project directory. Kopi will detect the `.kopi-version` file and prompt to install the missing JDK.
```

## Next Steps

- [Shell Integration](shell-integration.md) - Shell configuration
- [Managing Versions](managing-versions.md) - Version management
- [Configuration](../reference/configuration.md) - Advanced configuration
