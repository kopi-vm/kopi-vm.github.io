# Shims

Understanding how Kopi's shim system enables transparent JDK version management.

## What are Shims?

Shims are lightweight executables that act as proxies between your shell and the actual JDK binaries. When you run a Java command like `java` or `javac`, you're actually running a Kopi shim that:

1. Determines which JDK version to use (from `.kopi-version`, environment variables, or config)
2. Validates security and permissions
3. Optionally auto-installs missing JDKs
4. Executes the real JDK binary with your arguments

```mermaid
graph LR
    A[User runs: java MyApp] --> B[Kopi Shim]
    B --> C[Version Resolution]
    C --> D[Find/Install JDK]
    D --> E[Execute Real Binary]
    E --> F[Output to User]
```

## How Shims Work

### The Shim Binary

At the core of Kopi's shim system is `kopi-shim`, a high-performance binary written in Rust. This binary:

- Extracts the tool name from argv\[0] (how it was invoked)
- Resolves the appropriate JDK version to use
- Validates security constraints
- Finds or auto-installs the matching JDK
- Executes the real JDK tool

### Platform-Specific Implementation

#### Unix/Linux/macOS

On Unix-like systems, shims are **symbolic links** pointing to the `kopi-shim` binary. The shims directory contains symbolic links for each JDK tool (java, javac, jar, jshell, etc.), all pointing to the same kopi-shim binary located in the bin directory.

When invoked, `kopi-shim` uses the exec system call to replace itself with the real JDK binary, maintaining zero overhead after resolution.

#### Windows

On Windows, shims are **copies** of `kopi-shim.exe`. The shims directory contains individual executable files (java.exe, javac.exe, jar.exe, jshell.exe, etc.), each being an identical copy of the kopi-shim.exe binary.

Since Windows doesn't support process replacement, the shim spawns the JDK tool as a child process and exits with its status code.

## Version Resolution

The shim resolves JDK versions using the `VersionResolver` in this priority order:

1. **Tool-specific environment variable** (e.g., `KOPI_JAVA_VERSION`)
2. **General environment variable** (`KOPI_VERSION`)
3. **Project-specific file** (`.kopi-version` in current or parent directories)
4. **Global default** (`~/.kopi/version`)
5. **Configuration default** (from `config.toml`)

The version resolver creates a new instance with the current configuration, then determines both the requested version and its source (environment variable, project file, or global default).

## Auto-Installation

If a required JDK isn't installed, Kopi can automatically install it. When auto-install is enabled and you run a Java command with a missing JDK, Kopi will prompt you to install it immediately. After confirmation, it downloads and installs the required JDK version, then continues executing your original command.

This feature is controlled by the `KOPI_AUTO_INSTALL` environment variable or configuration setting.

## Security Features

The shim system includes multiple security layers:

### Path Validation

Kopi validates all JDK paths to prevent directory traversal attacks and ensure security. The validation process:

- Resolves the JDK path to its canonical form, eliminating symbolic links and relative path components
- Verifies that the resolved path is within the Kopi home directory
- Rejects any attempts to access JDKs outside the designated installation directory
- Prevents malicious actors from manipulating version files or environment variables to execute arbitrary binaries

This sandboxing approach ensures that Kopi only executes JDK binaries from trusted locations that it manages.

### Tool Name Validation

Tool names are validated against known JDK tools to prevent arbitrary command execution.

### Permission Checks

File permissions are verified before execution to ensure binaries haven't been tampered with.

## Performance Optimization

### Native Binary Performance

The `kopi-shim` binary is written in Rust for minimal overhead:

- **Version resolution**: < 10ms
- **Tool execution**: Zero overhead (process replacement on Unix)
- **Memory usage**: < 5MB

### Efficient Process Replacement

On Unix systems, the exec system call replaces the shim process entirely with the target JDK binary. This approach eliminates any runtime overhead since the shim process completely transforms into the Java process, maintaining the same process ID and system resources.

## Supported Tools

Kopi manages shims for all standard JDK tools, organized by category:

### Core Tools

Essential tools available in all JDK versions:

- `java` - Java application launcher
- `javac` - Java compiler
- `javadoc` - Documentation generator
- `jar` - Archive tool
- `javap` - Class file disassembler

### Debug Tools

- `jdb` - Java debugger
- `jconsole` - Monitoring console
- `jstack` - Stack trace tool
- `jmap` - Memory map tool
- `jhsdb` - HotSpot debugger (JDK 9+)

### Monitoring Tools

- `jps` - Process status
- `jstat` - Statistics monitoring
- `jinfo` - Configuration info
- `jcmd` - Diagnostic commands (JDK 7+)
- `jfr` - Flight Recorder (JDK 11+)

### Security Tools

- `keytool` - Key and certificate management
- `jarsigner` - JAR signing and verification

### Utility Tools

- `jshell` - Interactive shell (JDK 9+)
- `jlink` - Custom runtime creator (JDK 9+)
- `jmod` - Module tool (JDK 9+)
- `jdeps` - Dependency analyzer (JDK 8+)
- `jpackage` - Packaging tool (JDK 14+)
- `jwebserver` - Simple web server (JDK 18+)

### Distribution-Specific Tools

Some tools are only available in specific distributions:

#### GraalVM

- `native-image` - Native compilation
- `native-image-configure` - Configuration tool
- `native-image-inspect` - Inspection tool
- `js` - JavaScript interpreter (removed in GraalVM 23+)

#### SAP Machine

- `asprof` - Async profiler

## Shim Management

### Installation

Shims are created during Kopi setup. Running `kopi setup` creates the default shims for core tools like java, javac, javadoc, jar, and jshell.

### Creating Additional Shims

When you install a JDK, Kopi automatically creates shims for all its tools. The `kopi install` command detects all available tools in the JDK and creates corresponding shims.

You can manage shims using these commands:

- `kopi setup --force` - Force recreate all shims even if they exist
- `kopi shim add <tool>` - Add a shim for a specific tool
- `kopi shim remove <tool>` - Remove a specific shim

### Verification

Check shim integrity and functionality using these verification methods:

- `kopi doctor --check shell` - Run shell diagnostics including shim checks
- `kopi shim verify` - Check all shims for integrity issues
- `kopi shim verify --fix` - Automatically repair any detected problems
- `kopi shim list` - Display all installed shims

You can test a specific shim by running it directly with the version flag, or on Unix systems, use the readlink command to verify that the symbolic link points to the correct kopi-shim binary.

### Repair

Fix broken or corrupted shims using these repair commands:

- `kopi setup --force` - Force regenerate all shims from scratch
- `kopi shim verify --fix` - Detect and automatically repair issues

The repair process performs three steps: checking all shim integrity, removing any broken shims, and recreating them correctly with proper permissions and links.

## Platform-Specific Details

### macOS Bundle Structure

On macOS, Kopi handles both standard and bundle JDK structures. Standard JDKs place binaries directly in the bin directory, while macOS bundle JDKs use the Contents/Home/bin path structure. Kopi automatically detects and handles both layouts.

### Windows Path Handling

Windows shims handle path separators and `.exe` extensions automatically. The tool resolution system appends the .exe extension when running on Windows, allowing you to use the same commands across all platforms without worrying about platform-specific file extensions.

## Next Steps

- [Projects](../guide/projects.md) - Configure project-specific versions
- [Shell Integration](../guide/shell-integration.md) - Set up your shell
- [Configuration](../reference/configuration.md) - Configure Kopi settings
