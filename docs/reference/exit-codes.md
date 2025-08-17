# Exit Codes

Reference for Kopi exit codes and their meanings based on the actual implementation.

## Standard Exit Codes

### 0 - Success

The command completed successfully without any errors.

### 1 - General Error

An unspecified error occurred during command execution. This is the default exit code for errors that don't have a specific code assigned.

**Common causes**:

- I/O errors
- JSON parsing errors
- ZIP archive errors
- File system walk errors
- Thread panics
- Cache not found
- General system errors
- Not implemented features

**Resolution**:

- Check the error message for specific details
- Enable debug mode by setting KOPI_DEBUG environment variable to 1
- Report the issue if it persists

## Command and Configuration Errors

### 2 - Invalid Format or Configuration

The command received invalid input or configuration.

**Common causes**:

- Invalid version format specification
- Invalid configuration values
- Validation errors in input parameters

**Resolution**:

- Check the version format (use proper syntax like "21", "temurin@21", etc.)
- Verify configuration file syntax and values
- Review command arguments for correct format

### 3 - No Local Version

No JDK version configured for the current project.

**Common causes**:

- No .kopi-version or .java-version file found in current or parent directories
- Not in a project directory with version configuration

**Resolution**:

- Create a version file using the pin command
- Navigate to a project directory with a version file
- Set a global default using the use command

### 4 - JDK Not Installed

The requested JDK version is not installed on the system.

**Common causes**:

- JDK has not been installed yet
- Specified version doesn't exist
- JDK was previously uninstalled
- Auto-install was declined by user
- Auto-install failed

**Resolution**:

- Install the JDK using the install command
- List installed JDKs with the list command
- Check available versions with the search command
- Enable auto-install if desired

### 5 - Tool Not Found

A required tool was not found in the JDK installation.

**Common causes**:

- Tool doesn't exist in the specified JDK
- JDK installation is corrupted or incomplete
- Looking for a tool that's not part of the JDK

**Resolution**:

- Verify the tool name is correct
- Check which tools are available in the JDK
- Reinstall the JDK if installation appears corrupted

## Shell Integration Errors

### 6 - Shell Detection Error

Failed to detect the current shell environment.

**Common causes**:

- Unable to determine shell type
- Shell environment variables not set properly
- Running in an unsupported environment

**Resolution**:

- Manually specify the shell type
- Check shell environment variables
- Run Kopi from a supported shell

### 7 - Unsupported Shell

The detected shell is not supported by Kopi.

**Common causes**:

- Using a shell that Kopi doesn't support
- Shell version too old or too new
- Custom or uncommon shell

**Resolution**:

- Switch to a supported shell (bash, zsh, fish, etc.)
- Check Kopi documentation for supported shells
- Use manual PATH configuration as a workaround

## File System Errors

### 13 - Permission Denied

Insufficient permissions for the requested operation.

**Common causes**:

- Protected directories
- File ownership issues
- Read-only filesystem
- SELinux or AppArmor restrictions

**Resolution**:

- Check file and directory permissions
- Use user installation instead of root
- Fix ownership of .kopi directory in home folder
- Check security policy configurations

### 17 - Already Exists

The resource already exists and cannot be created again.

**Common causes**:

- JDK version already installed
- Configuration file already exists
- Shim already created

**Resolution**:

- Use the existing resource
- Remove the existing resource if replacement is needed
- Use force flag if available to override

## Network and Resource Errors

### 20 - Network Error

Network-related operations failed.

**Common causes**:

- No internet connection
- HTTP request failed
- Metadata fetch failed
- DNS resolution issues
- Proxy configuration problems
- SSL/TLS certificate issues

**Resolution**:

- Check network connectivity
- Verify proxy settings if behind a firewall
- Check if metadata servers are accessible
- Try using offline mode if available
- Retry the operation

### 28 - Insufficient Disk Space

Not enough disk space available for the operation.

**Common causes**:

- Disk full
- User quota exceeded
- Temporary directory full
- Not enough space for JDK installation

**Resolution**:

- Free disk space on the system
- Clean downloads using cache clear command
- Remove old JDKs using the prune command
- Use different disk by setting KOPI_HOME environment variable

## Command Not Found Errors

### 127 - Command Not Found

Standard POSIX exit code for command not found.

**Common causes**:

- Kopi binary not found in PATH
- Shell not found in PATH
- Shims directory not in PATH
- Installation incomplete

**Resolution**:

- Ensure Kopi is properly installed
- Add Kopi to PATH environment variable
- Configure shell using the init command
- Reinstall Kopi if necessary
