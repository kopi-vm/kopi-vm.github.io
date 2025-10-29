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

### Already Installed

When trying to install a JDK version that's already installed, you'll see an error message.

```bash
Error: temurin 21 is already installed
```

To reinstall or update an existing JDK installation, use the `--force` flag:

```bash
kopi install 21 --force
```

### Comprehensive Uninstall Issues

#### Multiple JDKs Match

When multiple JDK installations match your uninstall criteria:

```bash
Error: Multiple JDKs match version '21'
```

Be more specific with your uninstall command:

```bash
kopi uninstall temurin@21.0.5+11   # Specify exact version
kopi uninstall corretto@21          # Specify distribution
```

#### Files in Use (Windows)

On Windows, antivirus software or running processes may lock JDK files:

```bash
Error: Files may be held by antivirus software
```

To resolve:

- Close any running Java applications
- Temporarily disable real-time antivirus protection
- Add kopi directory to antivirus exclusions
- Use `--force` flag to attempt forced removal

#### Permission Errors

When you lack permissions to remove JDK files:

```bash
Error: Permission denied removing JDK
```

Solutions:

- Run as Administrator (Windows) or with sudo (Unix)
- Check file permissions in `~/.kopi/jdks/`
- Ensure no files are read-only or locked

#### Partial Removal Cleanup

If a previous uninstall was interrupted:

```bash
Error: Partial removal detected
```

Clean up partial removals:

```bash
kopi uninstall --cleanup
kopi uninstall --cleanup --force  # For stubborn files
```

Check for orphaned temporary directories and metadata files.

#### Orphaned Symlinks (Unix)

Broken symbolic links may be left behind:

```bash
Warning: Orphaned symlinks found
```

Symlinks are cleaned up automatically during uninstall. For manual cleanup:

```bash
find ~/.kopi -type l ! -exec test -e {} \; -delete
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
kopi uninstall --all temurin
```

Clear the download cache:

```bash
kopi cache clear
```

## Shim-Specific Issues

### Shim Not Working

If a tool like `java` isn't found in the JDK:

```bash
Error: Tool 'java' not found in JDK
```

Solutions:

- Ensure `~/.kopi/shims` is in your PATH
- Run `kopi shim verify` to check shim integrity
- Recreate the shim: `kopi shim add java --force`

### Version Not Switching

When the wrong Java version is used despite having a `.kopi-version` file:

Solutions:

- Check version file location: must be in current or parent directory
- Verify version format: `temurin@21` or just `21`
- Check environment variable: `KOPI_JAVA_VERSION` overrides files
- Enable debug logging: `RUST_LOG=kopi::shim=debug java -version`

### Shim Performance

If shim execution is slow:

Solutions:

- Run benchmarks: `cargo bench --bench shim_bench`
- Check for antivirus interference on Windows
- Ensure shims are built with release profile
- Verify no network delays in version resolution

### Security Validation Errors

When encountering security-related errors:

```bash
Error: Security error: Path contains directory traversal
```

Solutions:

- Check for suspicious patterns in version files
- Ensure no malformed symlinks in kopi directories
- Run `kopi shim verify --fix` to repair issues

## Shell Integration Problems

### PATH Not Updated

**Symptoms:**

- `java -version` shows wrong version
- `which java` doesn't point to kopi shims
- `kopi` command not found

**Solutions:**

1. Verify shims in PATH:

   ```bash
   echo $PATH | grep kopi/shims
   ```

2. Ensure correct PATH order:

   ```bash
   export PATH="$HOME/.kopi/shims:$PATH"
   ```

3. Check if kopi is installed:
   ```bash
   which kopi
   ```

### Version Not Detected

**Symptoms:**

- Kopi doesn't detect project version files
- Wrong version is used despite having `.kopi-version` or `.java-version`

**Solutions:**

1. Check file permissions:

   ```bash
   ls -la .kopi-version .java-version
   ```

2. Verify version format:

   ```bash
   cat .kopi-version  # Should be like: temurin@21
   cat .java-version  # Should be like: 21
   ```

3. Check version resolution:
   ```bash
   kopi current  # Shows which version is currently active
   kopi env  # Outputs environment variables for the current version
   ```

### Shell Detection Issues

**Symptoms:**

- `kopi env` fails to detect shell type
- Wrong shell syntax is generated

**Solutions:**

1. Explicitly specify shell:

   ```bash
   eval "$(kopi env --shell bash)"    # For bash
   kopi env --shell fish | source      # For fish
   kopi env --shell powershell | Invoke-Expression  # For PowerShell
   ```

2. Check your current shell:
   ```bash
   echo $SHELL
   echo $0
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

```powershell
$env:KOPI_HOME = "C:\kopi"
```

On Windows 10 and later, enable long path support (requires Administrator):

```powershell
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" `
  -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

#### PowerShell Execution Policy

Scripts may fail to execute in PowerShell due to execution policy restrictions:

```bash
Error: Scripts cannot execute in PowerShell
```

Check and update the execution policy:

```powershell
# Check current policy
Get-ExecutionPolicy

# Set policy for current user
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### macOS Issues

#### Gatekeeper Issues

When macOS prevents Kopi from running due to unverified developer:

```bash
Error: "kopi" cannot be opened because the developer cannot be verified
```

Remove the quarantine attribute:

```bash
xattr -d com.apple.quarantine ~/.kopi/bin/kopi
```

#### Shell Configuration

On macOS, ensure proper shell configuration:

- If using bash, ensure `.bash_profile` sources `.bashrc`
- For zsh (default on macOS 10.15+), use `~/.zshrc`

### Linux Issues

#### SELinux Contexts

Permission denied errors despite correct file permissions may be due to SELinux:

```bash
Error: Permission denied (even with correct file permissions)
```

Check and fix SELinux contexts:

```bash
# Check SELinux status
sestatus

# Set correct context
restorecon -Rv ~/.kopi
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

## Enhanced Error Messages

Kopi provides comprehensive error messages with helpful suggestions when something goes wrong. Error messages follow this structured format:

- Error type and brief description
- Detailed explanation of what went wrong
- Suggestion for how to fix the issue

```bash
# Example: Version not found
$ kopi install 999
Error: JDK version 'temurin 999' is not available

Details: Version lookup failed: temurin 999 not found

Suggestion: Run 'kopi cache search' to see available versions or 'kopi cache refresh' to update the list.
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

- [Commands Reference](reference/commands.md) - All commands
- [Configuration](reference/configuration.md) - Configuration options
