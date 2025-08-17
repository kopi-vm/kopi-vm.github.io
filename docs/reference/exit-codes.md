# Exit Codes

Reference for Kopi exit codes and their meanings.

## Standard Exit Codes

### 0 - Success

The command completed successfully.

```bash
kopi install 21
echo $?  # 0
```

### 1 - General Error

An unspecified error occurred.

```bash
kopi invalid-command
echo $?  # 1
```

**Common causes**:

- Unknown error
- Unhandled exception
- Generic failure

**Resolution**:

- Check error message
- Enable debug mode (`KOPI_DEBUG=1`)
- Report issue if persistent

## Command and Argument Errors

### 2 - Invalid Command or Arguments

The command syntax is incorrect or arguments are invalid.

```bash
kopi install
# Error: Missing required argument: VERSION
echo $?  # 2

kopi install --invalid-flag 21
# Error: Unknown option: --invalid-flag
echo $?  # 2
```

**Common causes**:

- Missing required arguments
- Unknown command
- Invalid option flags
- Incorrect argument format

**Resolution**:

- Check command syntax with `--help`
- Verify argument format
- Update Kopi if command should exist

### 3 - Version File Not Found

No version file found in current or parent directories.

```bash
cd /tmp/no-version-file
kopi current
# Error: No .kopi-version or .java-version file found
echo $?  # 3
```

**Common causes**:

- No `.kopi-version` or `.java-version` file
- Not in a project directory
- File deleted or renamed

**Resolution**:

- Create version file: `kopi pin 21`
- Navigate to project directory
- Set global default: `kopi use 21`

### 4 - JDK Not Installed

The requested JDK version is not installed.

```bash
kopi use temurin@99
# Error: JDK temurin@99 is not installed
echo $?  # 4
```

**Common causes**:

- JDK not yet installed
- Typo in version specification
- JDK was uninstalled

**Resolution**:

- Install JDK: `kopi install <version>`
- List installed: `kopi list`
- Check available: `kopi search`

## Installation and Download Errors

### 5 - Installation Failed

JDK installation failed.

```bash
kopi install invalid@version
# Error: Failed to install JDK
echo $?  # 5
```

**Common causes**:

- Invalid version specified
- Download failed
- Extraction failed
- Verification failed

**Resolution**:

- Verify version exists: `kopi search`
- Check network connection
- Clear cache: `kopi cache clear`
- Retry with `--force`

### 6 - Download Failed

Failed to download JDK archive.

```bash
# With network issues
kopi install 21
# Error: Failed to download JDK archive
echo $?  # 6
```

**Common causes**:

- Network connectivity issues
- Proxy configuration problems
- Server unavailable
- Timeout

**Resolution**:

- Check internet connection
- Configure proxy if needed
- Increase timeout: `KOPI_NETWORK_TIMEOUT=300`
- Try different mirror/distribution

### 7 - Checksum Verification Failed

Downloaded file failed checksum verification.

```bash
kopi install 21
# Error: Checksum verification failed
echo $?  # 7
```

**Common causes**:

- Corrupted download
- Man-in-the-middle attack
- Wrong checksum in metadata

**Resolution**:

- Retry download
- Clear downloads: `kopi cache clear --downloads`
- Update metadata: `kopi cache update`
- Report if persistent

### 8 - Extraction Failed

Failed to extract JDK archive.

```bash
kopi install 21
# Error: Failed to extract archive
echo $?  # 8
```

**Common causes**:

- Corrupted archive
- Insufficient disk space
- Permission issues
- Unsupported archive format

**Resolution**:

- Check disk space: `df -h`
- Clear downloads and retry
- Check permissions
- Update Kopi for format support

## Configuration and System Errors

### 10 - Configuration Error

Configuration file is invalid or cannot be loaded.

```bash
kopi config validate
# Error: Invalid configuration
echo $?  # 10
```

**Common causes**:

- Syntax error in config file
- Invalid values
- Missing required fields
- Corrupted file

**Resolution**:

- Fix syntax errors
- Reset config: `kopi config reset`
- Check with: `kopi config validate`
- Edit manually: `kopi config edit`

### 11 - Metadata Error

Metadata is corrupted or unavailable.

```bash
kopi search
# Error: Failed to load metadata
echo $?  # 11
```

**Common causes**:

- Corrupted cache
- Network issues
- Invalid metadata format

**Resolution**:

- Update metadata: `kopi cache update --force`
- Clear cache: `kopi cache clear`
- Check network connectivity
- Try offline mode if available

### 12 - Cache Error

Cache operations failed.

```bash
kopi cache update
# Error: Failed to update cache
echo $?  # 12
```

**Common causes**:

- Permission issues
- Disk full
- Corrupted cache
- I/O errors

**Resolution**:

- Check permissions on `~/.kopi/cache`
- Free disk space
- Clear cache: `kopi cache clear --all`
- Check filesystem health

## Permission and Access Errors

### 13 - Permission Denied

Insufficient permissions for operation.

```bash
sudo kopi install 21  # If KOPI_HOME is protected
# Error: Permission denied
echo $?  # 13
```

**Common causes**:

- Protected directories
- File ownership issues
- Read-only filesystem
- SELinux/AppArmor restrictions

**Resolution**:

- Check file permissions
- Use user installation (not root)
- Fix ownership: `chown -R $USER ~/.kopi`
- Check security policies

### 14 - Directory Not Found

Required directory does not exist.

```bash
KOPI_HOME=/nonexistent kopi list
# Error: Directory not found
echo $?  # 14
```

**Common causes**:

- KOPI_HOME misconfigured
- Directory deleted
- Mount point not available

**Resolution**:

- Create directory: `mkdir -p ~/.kopi`
- Fix KOPI_HOME variable
- Run setup: `kopi setup`

### 15 - File Not Found

Required file does not exist.

```bash
kopi validate /nonexistent/file
# Error: File not found
echo $?  # 15
```

**Common causes**:

- File deleted
- Wrong path specified
- File not created yet

**Resolution**:

- Verify file path
- Create file if needed
- Check working directory

## Network Errors

### 20 - Network Error

General network communication failure.

```bash
# With no internet
kopi cache update
# Error: Network error
echo $?  # 20
```

**Common causes**:

- No internet connection
- DNS resolution failure
- Firewall blocking
- SSL/TLS issues

**Resolution**:

- Check connectivity: `ping api.foojay.io`
- Check DNS: `nslookup api.foojay.io`
- Configure proxy if needed
- Use offline mode: `KOPI_OFFLINE=1`

### 21 - Timeout

Operation timed out.

```bash
KOPI_NETWORK_TIMEOUT=1 kopi install 21
# Error: Operation timed out
echo $?  # 21
```

**Common causes**:

- Slow network
- Server not responding
- Timeout too short

**Resolution**:

- Increase timeout: `KOPI_NETWORK_TIMEOUT=300`
- Try again later
- Use different mirror
- Check server status

### 22 - Proxy Error

Proxy configuration or connection issues.

```bash
KOPI_HTTP_PROXY=invalid:proxy kopi install 21
# Error: Proxy error
echo $?  # 22
```

**Common causes**:

- Invalid proxy settings
- Proxy authentication required
- Proxy server down

**Resolution**:

- Verify proxy settings
- Include authentication: `http://user:pass@proxy:port`
- Test proxy: `curl -x $KOPI_HTTP_PROXY https://api.foojay.io`
- Bypass proxy: `KOPI_NO_PROXY=api.foojay.io`

## Resource Errors

### 28 - Insufficient Disk Space

Not enough disk space for operation.

```bash
# On full disk
kopi install 21
# Error: Insufficient disk space
echo $?  # 28
```

**Common causes**:

- Disk full
- Quota exceeded
- Temporary directory full

**Resolution**:

- Free disk space
- Clean downloads: `kopi cache clear --downloads`
- Prune old JDKs: `kopi prune`
- Use different disk: `KOPI_HOME=/other/disk/kopi`

### 29 - Memory Error

Insufficient memory for operation.

```bash
# On low memory system
kopi install 21
# Error: Out of memory
echo $?  # 29
```

**Common causes**:

- System out of memory
- Process limits too low
- Memory leak

**Resolution**:

- Free memory
- Increase limits: `ulimit -m unlimited`
- Reduce parallel downloads: `KOPI_PARALLEL_DOWNLOADS=1`
- Report issue if persistent

## Shell and Environment Errors

### 126 - Command Found but Not Executable

Command exists but cannot be executed.

```bash
chmod -x ~/.kopi/shims/java
java --version
# Error: Permission denied
echo $?  # 126
```

**Common causes**:

- Missing execute permission
- Wrong file type
- Corrupted shim

**Resolution**:

- Fix permissions: `chmod +x ~/.kopi/shims/*`
- Regenerate shims: `kopi setup --regenerate-shims`
- Check file type: `file ~/.kopi/shims/java`

### 127 - Command Not Found

Command or required program not found.

```bash
# If shims not in PATH
java --version
# Error: Command not found
echo $?  # 127
```

**Common causes**:

- Shims not in PATH
- Shell not configured
- Required tool missing

**Resolution**:

- Add to PATH: `export PATH="$HOME/.kopi/shims:$PATH"`
- Configure shell: `eval "$(kopi init bash)"`
- Install missing tools
- Run setup: `kopi setup`

### 130 - Interrupted (Ctrl+C)

Operation interrupted by user.

```bash
kopi install 21
# Press Ctrl+C
echo $?  # 130
```

**Common causes**:

- User pressed Ctrl+C
- SIGINT signal received

**Resolution**:

- Retry operation
- Clean partial downloads
- Use `--force` to restart

## Using Exit Codes in Scripts

### Bash Script Example

```bash
#!/bin/bash
set -e  # Exit on error

install_jdk() {
    local version=$1

    if kopi install "$version"; then
        echo "Successfully installed $version"
    else
        case $? in
            4)
                echo "JDK already installed"
                ;;
            5)
                echo "Installation failed, retrying..."
                kopi install --force "$version"
                ;;
            20)
                echo "Network error, trying offline"
                KOPI_OFFLINE=1 kopi use "$version"
                ;;
            28)
                echo "Disk full, cleaning up..."
                kopi prune --force
                kopi install "$version"
                ;;
            *)
                echo "Unknown error: $?"
                exit 1
                ;;
        esac
    fi
}

install_jdk "21"
```

### CI/CD Example

```yaml
# GitHub Actions
- name: Install JDK
  run: |
    kopi install $(cat .kopi-version) || {
      code=$?
      if [ $code -eq 4 ]; then
        echo "::warning::JDK already installed"
      else
        echo "::error::Installation failed with code $code"
        exit $code
      fi
    }
```

### Error Handling Function

```bash
handle_kopi_error() {
    local exit_code=$1

    case $exit_code in
        0) return 0 ;;
        2) echo "Check command syntax with: kopi --help" ;;
        3) echo "Create version file with: kopi pin" ;;
        4) echo "Install JDK with: kopi install" ;;
        13) echo "Fix permissions: sudo chown -R $USER ~/.kopi" ;;
        20) echo "Check network or use offline mode" ;;
        28) echo "Free disk space with: kopi prune" ;;
        127) echo "Configure shell: eval \"\$(kopi init bash)\"" ;;
        *) echo "Error code $exit_code - check logs" ;;
    esac

    return $exit_code
}

# Usage
kopi current || handle_kopi_error $?
```

## Debugging Exit Codes

### Enable Debug Output

```bash
# See detailed error information
KOPI_DEBUG=1 kopi install 21
echo "Exit code: $?"
```

### Check System Errors

```bash
# Check system error
strerror() {
    python3 -c "import os; print(os.strerror($1))"
}

kopi install 21
strerror $?
```

### Log Exit Codes

```bash
# Log all Kopi commands and exit codes
kopi() {
    command kopi "$@"
    local code=$?
    echo "[$(date)] kopi $@ = $code" >> ~/.kopi/commands.log
    return $code
}
```

## Next Steps

- [Commands](commands.md) - Command reference
- [Troubleshooting](../troubleshooting.md) - Common issues
- [Environment Variables](environment.md) - Environment settings
