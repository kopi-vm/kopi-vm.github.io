# Troubleshooting

Solutions for common issues with Kopi.

## Installation Issues

### Kopi Command Not Found

After installation, the kopi command might not be recognized in your terminal. This typically happens when Kopi isn't added to your system's PATH environment variable.

To fix this issue, add Kopi to your PATH based on your shell:

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

If the issue persists, verify the installation by checking if the kopi binary exists:

```bash
ls ~/.cargo/bin/kopi
~/.cargo/bin/kopi --version
```

As a last resort, reinstall Kopi:

```bash
cargo install kopi --force
```

### Shell Integration Not Working

When Java commands don't use Kopi-managed versions and instead use the system Java, the shell integration needs configuration.

First, ensure the shims directory is in your PATH:

```bash
export PATH="$HOME/.kopi/shims:$PATH"
echo 'export PATH="$HOME/.kopi/shims:$PATH"' >> ~/.bashrc
```

Check that the shims directory appears before any system Java in your PATH:

```bash
echo $PATH
```

If needed, regenerate the shims:

```bash
kopi setup --force
```

Run the built-in diagnostic tool to identify configuration issues:

```bash
kopi doctor
```

## Version Management Issues

### Wrong JDK Version Active

When the wrong JDK version is being used, you need to check how Kopi is resolving versions.

Start by checking what version Kopi thinks should be active:

```bash
kopi current --verbose
env | grep KOPI_JAVA_VERSION
```

Clear any environment overrides that might be forcing a specific version:

```bash
unset KOPI_JAVA_VERSION
```

Check for version files in the current and parent directories:

```bash
cat .kopi-version .java-version 2>/dev/null
find . -name ".kopi-version" -o -name ".java-version"
```

Set the correct version for your needs:

```bash
# For current project
kopi local 21

# For all projects
kopi global 21

# For current shell session only
kopi shell 21
```

### Version File Not Detected

Kopi might not detect version files due to permission issues or incorrect formatting.

Verify the file exists and has proper permissions:

```bash
ls -la .kopi-version .java-version
```

Check the file contents are properly formatted:

```bash
cat .kopi-version
# Should contain: temurin@21 or similar

# Fix Windows line endings if needed
dos2unix .kopi-version
```

If problems persist, recreate the version file:

```bash
rm .kopi-version
kopi local 21
```

## Installation and Download Issues

### JDK Installation Fails

Installation failures can occur due to network issues, outdated metadata, or insufficient disk space.

First, update the metadata cache:

```bash
kopi cache refresh
```

Verify the requested version exists:

```bash
kopi search 21
```

If the issue persists, clear the cache completely:

```bash
kopi cache clear
kopi install 21
```

Try an alternative JDK distribution if one fails:

```bash
kopi install corretto@21
```

Check available disk space (each JDK requires approximately 500MB):

```bash
df -h ~/.kopi
```

Enable verbose mode for detailed error information:

```bash
kopi -v install 21
```

### Download Timeouts

For slow connections or large downloads, increase the timeout:

```bash
kopi install --timeout 600 21
```

If you're behind a corporate firewall, configure proxy settings:

```bash
export HTTP_PROXY=http://proxy:8080
export HTTPS_PROXY=http://proxy:8080
kopi install 21
```

### Checksum Verification Failed

Checksum failures indicate corrupted downloads or outdated metadata.

Clear the cache and retry:

```bash
kopi cache clear
kopi install 21
```

Update metadata to get the latest checksums:

```bash
kopi cache refresh
kopi install 21
```

## Network and Proxy Issues

### Behind Corporate Proxy

Configure proxy settings for your corporate network:

```bash
export HTTP_PROXY=http://user:pass@proxy.corp.com:8080
export HTTPS_PROXY=http://user:pass@proxy.corp.com:8080
export NO_PROXY=localhost,*.corp.com,10.0.0.0/8
```

### SSL Certificate Errors

Update system certificates to resolve SSL verification issues:

```bash
# Ubuntu/Debian
sudo apt-get update && sudo apt-get install ca-certificates

# macOS
brew install ca-certificates
```

## Performance Issues

### Slow Startup

Java commands might have slow startup times due to shim overhead or cache issues.

Measure the overhead:

```bash
time kopi current
# Should be < 50ms

time java --version
# Should be < 100ms overhead
```

Refresh the cache to improve performance:

```bash
kopi cache refresh
```

Enable verbose logging to identify bottlenecks:

```bash
kopi -vvv current
```

### High Disk Usage

Check how much space Kopi is using:

```bash
du -sh ~/.kopi/*
du -sh ~/.kopi/jdks/*
```

Remove unused JDK installations:

```bash
kopi list
kopi uninstall temurin@11
kopi uninstall --all
```

Clear the download cache:

```bash
kopi cache clear
```

## Platform-Specific Issues

### Windows Issues

#### Command Prompt vs PowerShell

Different shells on Windows require different configuration approaches.

For PowerShell, add to your profile and set JAVA_HOME:

```bash
# Add shims to PATH
$env:Path = "$env:USERPROFILE\.kopi\shims;$env:Path"

# Set JAVA_HOME
kopi env | Invoke-Expression
```

For Command Prompt, set the PATH environment variable:

```bash
# Add to system PATH
setx PATH "%USERPROFILE%\.kopi\shims;%PATH%"
```

Note that PowerShell uses semicolons as path separators and dollar signs for variables, while Command Prompt uses percent signs for variables.

#### Path Too Long

Windows has a path length limit that can cause issues with deeply nested directories.

Use a shorter KOPI_HOME location:

```bash
$env:KOPI_HOME = "C:\kopi"
```

On Windows 10 and later, enable long path support (requires Administrator):

```bash
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" `
  -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

## Diagnostic Tools

### Kopi Doctor

The doctor command provides comprehensive diagnostics for your Kopi installation. It runs various checks across different categories to identify potential issues:

```bash
# Run full diagnostic
kopi doctor

# Check specific category
kopi doctor --check installation
kopi doctor --check shell
kopi doctor --check jdks
kopi doctor --check permissions
kopi doctor --check network
kopi doctor --check cache

# Get JSON output for automation
kopi doctor --json
```

The available check categories are:

- **installation**: Verifies Kopi binary, version, directories, and configuration
- **shell**: Checks shell detection, PATH configuration, and shim functionality
- **jdks**: Validates JDK installations, integrity, disk space, and version consistency
- **permissions**: Examines directory and binary permissions
- **network**: Tests API connectivity, DNS resolution, proxy configuration, and TLS
- **cache**: Inspects cache files, permissions, format, staleness, and size

### Debug Output

Enable detailed logging using Rust's environment logger:

```bash
RUST_LOG=debug kopi install 21
RUST_LOG=trace kopi current

# Component-specific debugging
RUST_LOG=kopi::cache=debug kopi cache refresh
RUST_LOG=kopi::shim=trace kopi shim list
```

### Manual Checks

Perform manual verification of your Kopi installation:

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

Kopi provides structured error messages with helpful suggestions. Error messages follow this format:

- Error type and brief description
- Detailed explanation of what went wrong
- Suggestion for how to fix the issue

### Reporting Issues

When reporting issues, gather the following information:

Run diagnostics to get system and configuration status:

```bash
kopi doctor
```

Capture error output with debug logging:

```bash
RUST_LOG=debug kopi [command] 2>&1 | tee error.log
```

Cache state:

```bash
kopi cache info
```

Include in your report:

- Exact commands you ran
- Expected behavior
- Actual behavior you observed
- The debug output and system information

### Community Support

Get help and support through these channels:

- **GitHub Issues**: Report bugs and technical issues at the Kopi repository
- **Discussions**: Ask questions and share experiences with other users
- **Documentation**: Find comprehensive guides and references at the Kopi documentation site

## Next Steps

- [FAQ](faq.md) - Frequently asked questions
- [Commands Reference](reference/commands.md) - All commands
- [Configuration](reference/configuration.md) - Configuration options
