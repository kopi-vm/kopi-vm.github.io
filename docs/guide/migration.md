# Migrating from Other Tools

Guide for migrating from other JDK version management tools to Kopi.

## Migration Overview

Kopi is designed to be compatible with existing version management workflows while offering improved performance and features. Most migrations require minimal changes to your existing setup.

## From SDKMAN!

### Version File Migration

SDKMAN! uses `.sdkmanrc` files. Convert them to Kopi format:

```bash
# Convert .sdkmanrc to .kopi-version
if [ -f .sdkmanrc ]; then
    grep java .sdkmanrc | cut -d'=' -f2 | \
    sed 's/^/temurin@/' > .kopi-version
fi
```

### Command Mapping

| SDKMAN! Command | Kopi Equivalent |
|-----------------|-----------------|
| `sdk install java 17.0.2-tem` | `kopi install temurin@17.0.2` |
| `sdk use java 17.0.2-tem` | `kopi use temurin@17.0.2` |
| `sdk default java 17.0.2-tem` | `kopi use temurin@17.0.2` |
| `sdk list java` | `kopi search` |
| `sdk list java --installed` | `kopi list` |
| `sdk current java` | `kopi current` |
| `sdk uninstall java 17.0.2-tem` | `kopi uninstall temurin@17.0.2` |

### Migration Script

```bash
#!/bin/bash
# migrate-from-sdkman.sh

# List installed JDKs from SDKMAN!
echo "Migrating SDKMAN! JDKs to Kopi..."

# Parse SDKMAN! installations
for dir in ~/.sdkman/candidates/java/*; do
    if [ -d "$dir" ]; then
        version=$(basename "$dir")
        echo "Found: $version"
        
        # Map SDKMAN! identifiers to Kopi distributions
        case "$version" in
            *-tem) 
                kopi_version="temurin@${version%-tem}"
                ;;
            *-amzn)
                kopi_version="corretto@${version%-amzn}"
                ;;
            *-zulu)
                kopi_version="zulu@${version%-zulu}"
                ;;
            *-graal)
                kopi_version="graalvm@${version%-graal}"
                ;;
            *)
                kopi_version="temurin@$version"
                ;;
        esac
        
        echo "Installing $kopi_version with Kopi..."
        kopi install "$kopi_version"
    fi
done

# Set default version
current=$(sdk current java | grep -oP '\d+\.\d+\.\d+')
if [ -n "$current" ]; then
    kopi use "$current"
fi

echo "Migration complete!"
```

### Uninstalling SDKMAN!

After migrating to Kopi:

```bash
# Remove SDKMAN! (optional)
rm -rf ~/.sdkman
# Remove from shell config
sed -i '/SDKMAN/d' ~/.bashrc ~/.zshrc
```

## From jEnv

### Version File Compatibility

Good news! Kopi automatically reads `.java-version` files used by jEnv:

```bash
# No conversion needed - Kopi reads .java-version
cat .java-version
# 17.0.2

# Works with Kopi immediately
kopi current
# temurin@17.0.2
```

### Command Mapping

| jEnv Command | Kopi Equivalent |
|--------------|-----------------|
| `jenv add /path/to/jdk` | `kopi install temurin@17` |
| `jenv global 17.0.2` | `kopi use 17.0.2` |
| `jenv local 17.0.2` | `kopi pin 17.0.2` |
| `jenv shell 17.0.2` | `kopi shell 17.0.2` |
| `jenv versions` | `kopi list` |
| `jenv version` | `kopi current` |
| `jenv rehash` | Not needed (automatic) |

### Migration Script

```bash
#!/bin/bash
# migrate-from-jenv.sh

# Get jEnv versions
echo "Migrating jEnv configuration to Kopi..."

# Global version
if [ -f ~/.jenv/version ]; then
    global_version=$(cat ~/.jenv/version)
    echo "Setting global version: $global_version"
    kopi use "$global_version"
fi

# Project versions are already compatible (.java-version)
echo "Project .java-version files are automatically compatible"

# Install matching JDKs
jenv versions --bare | while read version; do
    echo "Installing JDK $version with Kopi..."
    kopi install "$version"
done

echo "Migration complete!"
```

## From Jabba

### Version File Migration

Convert `.jabbarc` to Kopi format:

```bash
# Convert .jabbarc (JSON) to .kopi-version
if [ -f .jabbarc ]; then
    jdk=$(cat .jabbarc | jq -r '.jdk')
    
    # Map Jabba format to Kopi
    case "$jdk" in
        adopt@*) 
            echo "temurin@${jdk#adopt@}" > .kopi-version
            ;;
        amazon-corretto@*)
            echo "corretto@${jdk#amazon-corretto@}" > .kopi-version
            ;;
        *)
            echo "$jdk" > .kopi-version
            ;;
    esac
fi
```

### Command Mapping

| Jabba Command | Kopi Equivalent |
|---------------|-----------------|
| `jabba install adopt@17.0.2` | `kopi install temurin@17.0.2` |
| `jabba use adopt@17.0.2` | `kopi use temurin@17.0.2` |
| `jabba use-project` | Automatic with version files |
| `jabba ls` | `kopi list` |
| `jabba ls-remote` | `kopi search` |
| `jabba current` | `kopi current` |
| `jabba uninstall adopt@17.0.2` | `kopi uninstall temurin@17.0.2` |

## From asdf-java

### Version File Compatibility

Convert `.tool-versions` to Kopi format:

```bash
# Extract Java version from .tool-versions
if [ -f .tool-versions ]; then
    java_line=$(grep "^java " .tool-versions)
    version=$(echo "$java_line" | cut -d' ' -f2)
    
    # Map asdf format to Kopi
    case "$version" in
        adoptopenjdk-*)
            echo "temurin@${version#adoptopenjdk-}" > .kopi-version
            ;;
        corretto-*)
            echo "corretto@${version#corretto-}" > .kopi-version
            ;;
        *)
            echo "$version" > .kopi-version
            ;;
    esac
fi
```

### Command Mapping

| asdf Command | Kopi Equivalent |
|--------------|-----------------|
| `asdf install java latest:17` | `kopi install 17` |
| `asdf global java 17.0.2` | `kopi use 17.0.2` |
| `asdf local java 17.0.2` | `kopi pin 17.0.2` |
| `asdf list java` | `kopi list` |
| `asdf list-all java` | `kopi search` |
| `asdf current java` | `kopi current` |

## From System JDK

### Using System-Installed JDK

If you have JDK installed via apt, yum, brew, etc.:

```bash
# Check current system JDK
java --version

# Install same version with Kopi
kopi install 17  # or matching version

# Gradually migrate projects
cd my-project
kopi pin 17
```

### Maintaining System JDK

Keep system JDK as fallback:

```bash
# Configure Kopi to fall back to system JDK
# In ~/.kopi/config.toml
[fallback]
use_system_jdk = true
```

## General Migration Tips

### 1. Inventory Current Setup

```bash
# List all Java installations
echo "=== System JDK ==="
/usr/bin/java --version 2>&1 | head -1

echo "=== SDKMAN! ==="
ls ~/.sdkman/candidates/java/ 2>/dev/null

echo "=== jEnv ==="
jenv versions 2>/dev/null

echo "=== Jabba ==="
jabba ls 2>/dev/null

echo "=== asdf ==="
asdf list java 2>/dev/null
```

### 2. Install Kopi

```bash
# Install via package manager (recommended)
brew install kopi-vm/tap/kopi  # macOS
# or
curl -fsSL https://kopi-vm.github.io/install.sh | bash
```

### 3. Migrate Projects

```bash
# Find all version files
find . -name ".sdkmanrc" -o -name ".java-version" \
       -o -name ".jabbarc" -o -name ".tool-versions" | \
while read file; do
    dir=$(dirname "$file")
    echo "Migrating $dir"
    cd "$dir"
    
    # Create .kopi-version based on existing file
    # ... conversion logic ...
done
```

### 4. Update CI/CD

Replace version manager installation in CI:

```yaml
# Before (SDKMAN!)
- curl -s "https://get.sdkman.io" | bash
- source "$HOME/.sdkman/bin/sdkman-init.sh"
- sdk install java 17.0.2-tem

# After (Kopi)
- curl -fsSL https://kopi-vm.github.io/install.sh | bash
- kopi install temurin@17.0.2
```

### 5. Update Documentation

Update your project README:

```markdown
## Prerequisites

This project requires JDK 17. Install using Kopi:

\`\`\`bash
# Install Kopi (if not already installed)
brew install kopi-vm/tap/kopi

# Install required JDK
kopi install $(cat .kopi-version)
\`\`\`
```

## Feature Comparison

| Feature | SDKMAN! | jEnv | Jabba | asdf | Kopi |
|---------|---------|------|-------|------|------|
| Auto-switching | ✓ | ✓ | ✓ | ✓ | ✓ |
| Multiple distributions | ✓ | - | ✓ | ✓ | ✓ |
| Version file | .sdkmanrc | .java-version | .jabbarc | .tool-versions | .kopi-version/.java-version |
| Shell support | Bash/Zsh | Bash/Zsh/Fish | Bash/Zsh/Fish | Bash/Zsh/Fish | Bash/Zsh/Fish/PowerShell |
| Windows support | WSL | - | ✓ | - | ✓ |
| Offline mode | - | ✓ | - | - | ✓ |
| Written in | Bash | Bash | Go | Bash | Rust |
| Performance | Moderate | Slow | Fast | Moderate | Very Fast |

## Rollback Plan

If you need to rollback to your previous tool:

```bash
# Keep both tools installed initially
# Use environment variable to switch

# Use Kopi
export PATH="$HOME/.kopi/shims:$PATH"

# Use SDKMAN!
export PATH="$HOME/.sdkman/candidates/java/current/bin:$PATH"
```

## Getting Help

- [Troubleshooting](../troubleshooting.md) - Common issues
- [FAQ](../faq.md) - Frequently asked questions
- [GitHub Issues](https://github.com/kopi-vm/kopi/issues) - Report problems

## Next Steps

- [Quick Start](../getting-started/quickstart.md) - Get started with Kopi
- [Shell Integration](shell-integration.md) - Configure your shell
- [Working with Projects](projects.md) - Project setup