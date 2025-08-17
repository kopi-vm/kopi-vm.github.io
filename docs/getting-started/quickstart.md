# Quick Start

Get up and running with Kopi in just a few minutes.

## Installation

If you haven't installed Kopi yet, see the [Installation Guide](installation.md).

## Basic Commands

### List Available JDKs

```bash
# Search for available JDK versions
kopi search

# Search for specific version
kopi search 21

# Search for specific distribution
kopi search temurin
```

### Install a JDK

```bash
# Install latest JDK 21
kopi install 21

# Install specific distribution
kopi install temurin@21

# Install exact version
kopi install temurin@21.0.2
```

### Switch JDK Versions

```bash
# Use a specific version globally
kopi global 21

# Set project-specific version
kopi local 21

# Check current version
kopi current
```

## Project Setup

### Set Project JDK Version

```bash
# In your project directory
cd my-java-project

# Set JDK version for this project
kopi local 17

# This creates a .kopi-version file
cat .kopi-version
# temurin@17
```

### Automatic Version Switching

When you enter a project directory with a `.kopi-version` file, Kopi automatically switches to the specified JDK version.

```bash
cd my-java-project
java --version  # Automatically uses JDK 17

cd ../another-project
java --version  # Switches to this project's JDK version
```

## What's Next?

- [First Steps](first-steps.md) - Learn more commands and features
- [Managing JDK Versions](../guide/managing-versions.md) - Advanced version management
- [Working with Projects](../guide/projects.md) - Project configuration best practices
