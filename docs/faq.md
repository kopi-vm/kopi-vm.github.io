# Frequently Asked Questions

Common questions about Kopi and JDK version management.

## General Questions

### What is Kopi?

Kopi is a fast, lightweight JDK version manager written in Rust. It allows you to:
- Install multiple JDK versions from various distributions
- Switch between versions automatically based on project requirements
- Manage JDKs without manual PATH configuration
- Work with different JDK distributions (Temurin, Corretto, GraalVM, etc.)

### How does Kopi differ from other version managers?

| Feature | Kopi | SDKMAN! | jEnv | Jabba |
|---------|------|---------|------|-------|
| **Written in** | Rust | Bash | Bash | Go |
| **Performance** | Very Fast | Moderate | Slow | Fast |
| **Auto-switching** | ✓ | ✓ | ✓ | ✓ |
| **Multiple distributions** | ✓ | ✓ | ✗ | ✓ |
| **Windows support** | ✓ | WSL only | ✗ | ✓ |
| **Offline mode** | ✓ | ✗ | ✓ | ✗ |
| **Shell integration** | Native | Source | Shims | Shims |

### Which operating systems are supported?

Kopi supports:
- **Linux** (x64, ARM64, x86) - All major distributions
- **macOS** (Intel and Apple Silicon)
- **Windows** (x64, ARM64) - Native support, no WSL required

### Which shells are supported?

- **Bash** - Full support
- **Zsh** - Full support  
- **Fish** - Full support
- **PowerShell** - Full support (Windows)
- **Command Prompt** - Basic support (Windows)

## Installation Questions

### How do I install Kopi?

**Recommended methods**:

```bash
# macOS (Homebrew)
brew install kopi-vm/tap/kopi

# Ubuntu/Debian (PPA)
curl -fsSL https://kopi-vm.github.io/install.sh | bash

# Windows (winget)
winget install kopi

# All platforms (Cargo)
cargo install kopi
```

### Where does Kopi install JDKs?

By default, JDKs are installed in:
- **Unix/Linux/macOS**: `~/.kopi/jdks/`
- **Windows**: `%USERPROFILE%\.kopi\jdks\`

Example structure:
```
~/.kopi/jdks/
├── temurin-21.0.2/
├── corretto-17.0.9/
└── graalvm-21.0.1/
```

### Can I change the installation directory?

Yes, set the `KOPI_HOME` environment variable:

```bash
export KOPI_HOME=/opt/kopi
```

Or configure in `~/.kopi/config.toml`:

```toml
[paths]
jdks = "/opt/kopi/jdks"
```

### How much disk space do I need?

- **Kopi itself**: ~10 MB
- **Per JDK**: 200-500 MB (depending on distribution)
- **Metadata cache**: ~5 MB
- **Recommended free space**: 2 GB for multiple JDKs

## Usage Questions

### How do I install a specific JDK version?

```bash
# Install latest patch of major version
kopi install 21

# Install specific distribution
kopi install temurin@21
kopi install corretto@17

# Install exact version
kopi install temurin@21.0.2+13.0.LTS
```

### How do I switch between JDK versions?

**Global (system-wide default)**:
```bash
kopi use 21
```

**Project (directory-specific)**:
```bash
kopi pin 21  # Creates .kopi-version file
```

**Shell (current session only)**:
```bash
kopi shell 21
```

### How do I list available JDK versions?

```bash
# List installed JDKs
kopi list

# Search available JDKs
kopi search

# Search specific version
kopi search 21

# Search by distribution
kopi search graalvm
```

### What are version files?

Version files tell Kopi which JDK to use in a directory:

- **`.kopi-version`** - Kopi native format
- **`.java-version`** - Compatible with jEnv and other tools

Example `.kopi-version`:
```
temurin@21
```

### How does automatic version switching work?

When you run a Java command, Kopi:
1. Checks for `KOPI_VERSION` environment variable
2. Looks for `.kopi-version` or `.java-version` in current directory
3. Searches parent directories for version files
4. Falls back to global default
5. Uses system JDK as last resort

## Distribution Questions

### Which JDK distributions are supported?

Major distributions include:
- **Eclipse Temurin** (default, recommended)
- **Amazon Corretto**
- **Azul Zulu**
- **GraalVM**
- **Microsoft OpenJDK**
- **IBM Semeru** (OpenJ9)
- **BellSoft Liberica**
- **Oracle OpenJDK**

See [Supported Distributions](reference/distributions.md) for complete list.

### Which distribution should I use?

**For most users**: Eclipse Temurin (default)
- High quality, TCK-tested
- Wide platform support
- Regular updates

**Special cases**:
- **AWS**: Amazon Corretto
- **Azure**: Microsoft OpenJDK
- **Native compilation**: GraalVM
- **Memory-constrained**: IBM Semeru (OpenJ9)
- **Need JavaFX**: BellSoft Liberica

### Can I use Oracle JDK?

Yes, but be aware of licensing:

```bash
kopi install oracle@21
```

**Note**: Oracle JDK requires accepting license terms and may require a commercial license for production use.

### How do I install GraalVM?

```bash
# Install GraalVM
kopi install graalvm@21

# Or community edition
kopi install graalvm_community@21
```

## Configuration Questions

### How do I configure Kopi?

Edit `~/.kopi/config.toml`:

```bash
# Open in editor
kopi config edit

# Or set specific values
kopi config set defaults.distribution corretto
```

### Can I use Kopi offline?

Yes, Kopi supports offline mode:

```bash
# Enable offline mode
export KOPI_OFFLINE=1

# Works with cached metadata
kopi search  # Uses cached data
kopi install 21   # Works if JDK is cached
```

### How do I use Kopi behind a proxy?

Set proxy environment variables:

```bash
export KOPI_HTTP_PROXY=http://proxy:8080
export KOPI_HTTPS_PROXY=http://proxy:8080
export KOPI_NO_PROXY=localhost,internal.company.com
```

Or configure in `~/.kopi/config.toml`:

```toml
[network]
http_proxy = "http://proxy:8080"
https_proxy = "http://proxy:8080"
no_proxy = "localhost,internal.company.com"
```

### How do I speed up downloads?

1. **Increase parallel downloads**:
```bash
export KOPI_PARALLEL_DOWNLOADS=8
```

2. **Use closer mirrors** (automatic by default)

3. **Enable compression**:
```toml
[cache]
compress = true
```

## Troubleshooting Questions

### Why isn't Kopi detecting my version file?

Common causes:
1. **Wrong filename**: Must be `.kopi-version` or `.java-version`
2. **Wrong format**: Check file contents
3. **Line endings**: Use Unix line endings (LF, not CRLF)
4. **Permissions**: File must be readable

Fix:
```bash
# Validate file
kopi validate .kopi-version

# Recreate file
kopi pin 21
```

### Why is the wrong JDK version active?

Check version resolution:

```bash
# See active version and why
kopi current --verbose

# Check for overrides
env | grep KOPI_VERSION

# Clear overrides
unset KOPI_VERSION
kopi shell --unset
```

### How do I uninstall Kopi?

1. **Remove Kopi binary**:
```bash
# If installed with Cargo
cargo uninstall kopi

# If installed with Homebrew
brew uninstall kopi

# Manual removal
rm ~/.cargo/bin/kopi
```

2. **Remove Kopi data** (optional):
```bash
rm -rf ~/.kopi
```

3. **Remove shell integration**:
```bash
# Remove from ~/.bashrc, ~/.zshrc, etc.
# Lines containing: eval "$(kopi init ...)"
```

### How do I report a bug?

1. **Check existing issues**: https://github.com/kopi-vm/kopi/issues

2. **Gather information**:
```bash
kopi doctor --system-info > system-info.txt
KOPI_DEBUG=1 kopi [command] 2>&1 > debug.log
```

3. **Create issue** with:
- System information
- Steps to reproduce
- Expected vs actual behavior
- Debug output

## Migration Questions

### How do I migrate from SDKMAN!?

```bash
# List SDKMAN! JDKs
ls ~/.sdkman/candidates/java/

# Install same versions with Kopi
kopi install temurin@17
kopi install temurin@21

# Convert .sdkmanrc files
grep java .sdkmanrc | cut -d'=' -f2 > .kopi-version
```

### How do I migrate from jEnv?

Good news! Kopi automatically reads `.java-version` files:

```bash
# No conversion needed
cat .java-version  # Works with Kopi

# Install matching JDKs
kopi install $(cat .java-version)
```

### Can I use Kopi alongside other version managers?

Yes, but configure PATH carefully:

```bash
# Kopi first (recommended)
export PATH="$HOME/.kopi/shims:$PATH"

# Or other tool first
export PATH="$PATH:$HOME/.kopi/shims"
```

## Advanced Questions

### Can I create custom JDK distributions?

Yes, using custom metadata:

```toml
# ~/.kopi/metadata/custom.toml
[[distributions]]
name = "custom-jdk"
vendor = "Internal"

[[distributions.versions]]
version = "17.0.0-internal"
url = "https://internal.company.com/jdk.tar.gz"
checksum = "sha256:..."
```

### How do I use different JDKs in CI/CD?

**GitHub Actions**:
```yaml
- name: Setup JDK
  run: |
    kopi install $(cat .kopi-version)
```

**GitLab CI**:
```yaml
before_script:
  - kopi install $(cat .kopi-version)
```

**Jenkins**:
```groovy
sh 'kopi install $(cat .kopi-version)'
```

### Can I use Kopi in Docker?

Yes, example Dockerfile:

```dockerfile
FROM ubuntu:22.04

# Install Kopi
RUN apt-get update && apt-get install -y curl && \
    curl -fsSL https://kopi-vm.github.io/install.sh | bash

ENV PATH="/root/.kopi/shims:$PATH"

# Install JDK
COPY .kopi-version .
RUN kopi install $(cat .kopi-version)

# Your application
COPY . /app
WORKDIR /app
CMD ["java", "-jar", "app.jar"]
```

### How do I contribute to Kopi?

1. **Fork repository**: https://github.com/kopi-vm/kopi
2. **Set up development environment**:
```bash
git clone https://github.com/YOUR_USERNAME/kopi
cd kopi
cargo build
cargo test
```
3. **Make changes** and submit pull request
4. **See** [Contributing Guide](https://github.com/kopi-vm/kopi/blob/main/CONTRIBUTING.md)

## Performance Questions

### Why is Kopi fast?

- **Written in Rust**: Compiled to native code
- **Efficient shims**: Minimal overhead (<10ms)
- **Smart caching**: Multiple cache layers
- **Lazy loading**: Only loads what's needed
- **Parallel operations**: Downloads and extractions

### How can I make Kopi even faster?

1. **Keep cache updated**:
```bash
kopi cache update
```

2. **Disable auto-switching** if not needed:
```bash
export KOPI_AUTO_SWITCH=false
```

3. **Use local metadata**:
```bash
export KOPI_OFFLINE=1  # When possible
```

## Security Questions

### Is Kopi secure?

Yes, Kopi implements several security measures:
- **HTTPS only** for downloads
- **Checksum verification** for all JDKs
- **Certificate validation**
- **Path validation** to prevent directory traversal
- **No arbitrary code execution**

### How does Kopi verify downloads?

1. Downloads over HTTPS
2. Verifies SHA256 checksum
3. Validates archive structure
4. Checks file permissions

### Can I disable security checks?

Not recommended, but possible for development:

```bash
# Disable checksum verification
export KOPI_VERIFY_CHECKSUMS=false

# Disable certificate verification
export KOPI_VERIFY_CERTIFICATES=false
```

**Never disable in production!**

## Next Steps

- [Getting Started](getting-started/installation.md) - Installation guide
- [Quick Start](getting-started/quickstart.md) - Basic usage
- [Troubleshooting](troubleshooting.md) - Solve problems
- [Commands Reference](reference/commands.md) - All commands