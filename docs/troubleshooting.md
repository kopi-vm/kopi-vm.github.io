# Troubleshooting

Solutions for common issues with Kopi.

## Installation Issues

### Kopi Command Not Found

**Problem**: After installation, `kopi` command is not recognized.

```bash
$ kopi --version
bash: kopi: command not found
```

**Solutions**:

1. **Add Kopi to PATH**:
```bash
# For bash
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# For zsh
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# For fish
set -U fish_user_paths $HOME/.cargo/bin $fish_user_paths
```

2. **Verify installation**:
```bash
# Check if kopi exists
ls ~/.cargo/bin/kopi

# Try absolute path
~/.cargo/bin/kopi --version
```

3. **Reinstall if needed**:
```bash
cargo install kopi --force
```

### Shell Integration Not Working

**Problem**: Java commands don't use Kopi-managed versions.

```bash
$ java --version
# Shows system Java instead of Kopi-managed version
```

**Solutions**:

1. **Configure shell integration**:
```bash
# Add shims to PATH
export PATH="$HOME/.kopi/shims:$PATH"

# Add to shell config file
echo 'export PATH="$HOME/.kopi/shims:$PATH"' >> ~/.bashrc
```

2. **Check PATH order**:
```bash
echo $PATH
# Ensure ~/.kopi/shims comes before system Java

# Fix PATH order
export PATH="$HOME/.kopi/shims:$PATH"
```

3. **Regenerate shims**:
```bash
kopi setup --force
```

4. **Run diagnostics**:
```bash
kopi doctor
```

## Version Management Issues

### Wrong JDK Version Active

**Problem**: Wrong JDK version is being used.

```bash
$ java --version
# Shows JDK 11 but should be JDK 21
```

**Solutions**:

1. **Check version resolution**:
```bash
# See what version Kopi thinks should be active
kopi current --verbose

# Check for overrides
env | grep KOPI_VERSION
```

2. **Clear overrides**:
```bash
# Clear environment override
unset KOPI_VERSION
```

3. **Verify version files**:
```bash
# Check current directory
cat .kopi-version .java-version 2>/dev/null

# Check parent directories
find . -name ".kopi-version" -o -name ".java-version"
```

4. **Set correct version**:
```bash
# For project
kopi local 21

# For global
kopi global 21

# For shell session
kopi shell 21
```

### Version File Not Detected

**Problem**: Kopi doesn't detect `.kopi-version` or `.java-version` file.

**Solutions**:

1. **Check file exists and is readable**:
```bash
ls -la .kopi-version .java-version
# Check permissions
```

2. **Verify file contents**:
```bash
cat .kopi-version
# Should contain: temurin@21 or similar

# Fix line endings (Windows)
dos2unix .kopi-version
```

3. **Create new version file**:
```bash
# Remove old file
rm .kopi-version

# Create new
kopi local 21
```

## Installation and Download Issues

### JDK Installation Fails

**Problem**: Cannot install JDK versions.

```bash
$ kopi install 21
Error: Failed to install JDK
```

**Solutions**:

1. **Update metadata cache**:
```bash
kopi cache refresh
```

2. **Check available versions**:
```bash
kopi search 21
# Verify version exists
```

3. **Clear cache and retry**:
```bash
kopi cache clear
kopi install 21
```

4. **Try different distribution**:
```bash
# If temurin fails, try corretto
kopi install corretto@21
```

5. **Check disk space**:
```bash
df -h ~/.kopi
# Need ~500MB per JDK
```

6. **Enable verbose mode**:
```bash
kopi -v install 21
```

### Download Timeouts

**Problem**: Downloads timeout or fail.

**Solutions**:

1. **Increase timeout**:
```bash
export KOPI_NETWORK_TIMEOUT=600
kopi install 21
```

2. **Configure proxy** (if behind firewall):
```bash
export HTTP_PROXY=http://proxy:8080
export HTTPS_PROXY=http://proxy:8080
kopi install 21
```

3. **Try with increased timeout**:
```bash
kopi install --timeout 600 21
```

### Checksum Verification Failed

**Problem**: Downloaded JDK fails checksum verification.

**Solutions**:

1. **Clear cache and retry**:
```bash
kopi cache clear
kopi install 21
```

2. **Update metadata** (checksums might be outdated):
```bash
kopi cache refresh
kopi install 21
```

## Network and Proxy Issues

### Behind Corporate Proxy

**Problem**: Cannot download JDKs behind corporate proxy.

**Solutions**:

1. **Configure proxy settings**:
```bash
# Set proxy
export HTTP_PROXY=http://user:pass@proxy.corp.com:8080
export HTTPS_PROXY=http://user:pass@proxy.corp.com:8080

# Exclude internal hosts
export NO_PROXY=localhost,*.corp.com,10.0.0.0/8
```

### SSL Certificate Errors

**Problem**: SSL certificate verification fails.

**Solutions**:

1. **Update system certificates**:
```bash
# Ubuntu/Debian
sudo apt-get update && sudo apt-get install ca-certificates

# macOS
brew install ca-certificates
```

2. **Use system certificates**:
```bash
# The system certificates should be configured correctly
```

## Performance Issues

### Slow Startup

**Problem**: Java commands have slow startup time.

**Solutions**:

1. **Check shim performance**:
```bash
time kopi current
# Should be < 50ms

time java --version
# Should be < 100ms overhead
```

2. **Update cache**:
```bash
kopi cache refresh
```

3. **Profile execution**:
```bash
kopi -vvv current
```

### High Disk Usage

**Problem**: Kopi using too much disk space.

**Solutions**:

1. **Check disk usage**:
```bash
du -sh ~/.kopi/*
du -sh ~/.kopi/jdks/*
```

2. **Remove unused JDKs**:
```bash
# List installed
kopi list

# Remove specific version
kopi uninstall temurin@11

# Remove specific versions
kopi uninstall --all
```

3. **Clear cache**:
```bash
kopi cache clear
```


## Platform-Specific Issues

### Windows Issues

#### Command Prompt vs PowerShell

**Problem**: Different behavior in CMD vs PowerShell.

**Solutions**:

1. **Use appropriate initialization**:
```powershell
# PowerShell
kopi init powershell | Invoke-Expression

# Command Prompt
kopi init cmd
```

2. **Check PATH format**:
```powershell
# PowerShell uses semicolon
$env:Path = "$env:USERPROFILE\.kopi\shims;$env:Path"

# CMD uses percent signs
set PATH=%USERPROFILE%\.kopi\shims;%PATH%
```

#### Path Too Long

**Problem**: Windows path length limit exceeded.

**Solutions**:

1. **Use shorter KOPI_HOME**:
```powershell
$env:KOPI_HOME = "C:\kopi"
```

2. **Enable long path support** (Windows 10+):
```powershell
# Run as Administrator
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" `
  -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

### macOS Issues

#### Quarantine Attribute

**Problem**: macOS blocks downloaded JDKs.

**Solutions**:

1. **Remove quarantine**:
```bash
xattr -r -d com.apple.quarantine ~/.kopi/jdks/
```

2. **Configure auto-removal**:
```toml
# ~/.kopi/config.toml
[platform.macos]
remove_quarantine = true
```

#### Rosetta on Apple Silicon

**Problem**: Need to run x64 JDK on Apple Silicon.

**Solutions**:

1. **Install Rosetta**:
```bash
softwareupdate --install-rosetta
```

2. **Install x64 JDK**:
```bash
kopi install --arch x64 temurin@21
```

### Linux Issues

#### Permission Denied

**Problem**: Permission errors during installation.

**Solutions**:

1. **Fix ownership**:
```bash
sudo chown -R $USER:$USER ~/.kopi
```

2. **Check directory permissions**:
```bash
chmod -R u+rwX ~/.kopi
```

3. **SELinux/AppArmor**:
```bash
# Check SELinux
getenforce

# Temporarily disable (if needed)
sudo setenforce 0
```

#### GLIBC vs MUSL

**Problem**: Wrong C library for distribution.

**Solutions**:

1. **Check system library**:
```bash
ldd --version  # Shows glibc version
# or
apk info musl  # On Alpine
```

2. **Configure preference**:
```toml
# ~/.kopi/config.toml
[preferences]
libc = "musl"  # For Alpine
# or
libc = "glibc"  # For most Linux
```

## Diagnostic Tools

### Kopi Doctor

Run comprehensive diagnostics:

```bash
# Full diagnostic
kopi doctor

# Check specific component
kopi doctor --check shell
kopi doctor --check shims
kopi doctor --check path
kopi doctor --check metadata

# Auto-fix issues
kopi doctor --fix
```

### Debug Output

Enable detailed logging:

```bash
# General debug
export KOPI_DEBUG=1

# Component-specific
export KOPI_DEBUG_METADATA=1
export KOPI_DEBUG_CACHE=1
export KOPI_DEBUG_SHIM=1

# Trace level
export KOPI_TRACE=all
```

### Manual Checks

```bash
# Check installation
which kopi
kopi --version

# Check shims
ls -la ~/.kopi/shims/
~/.kopi/shims/java --version

# Check PATH
echo $PATH | tr ':' '\n' | grep kopi

# Check environment
env | grep -E "(KOPI|JAVA)"

# Check cache
kopi cache info
ls -la ~/.kopi/cache/
```

## Getting Help

### Error Messages

Understanding error messages:

```bash
# Error format
Error: <error type>
  <detailed message>
  
Suggestion: <helpful hint>

# Example
Error: JDK not found: temurin@25
  The requested JDK version is not available.
  
Suggestion: Run 'kopi search 25' to see available versions
```

### Reporting Issues

When reporting issues, include:

1. **System information**:
```bash
kopi doctor --system-info
```

2. **Error output**:
```bash
KOPI_DEBUG=1 kopi [command] 2>&1 | tee error.log
```

3. **Cache information**:
```bash
kopi cache info
```

4. **Reproduction steps**:
- Exact commands run
- Expected behavior
- Actual behavior

### Community Support

- **GitHub Issues**: https://github.com/kopi-vm/kopi/issues
- **Discussions**: https://github.com/kopi-vm/kopi/discussions
- **Documentation**: https://kopi-vm.github.io

## Next Steps

- [FAQ](faq.md) - Frequently asked questions
- [Commands Reference](reference/commands.md) - All commands
- [Configuration](reference/configuration.md) - Configuration options