# Commands Reference

Complete reference for all Kopi commands and options.

## Global Options

Kopi supports several global options that can be used with any command:

| Option                      | Short | Description                                                |
| --------------------------- | ----- | ---------------------------------------------------------- |
| `--help`                    | `-h`  | Display help information for any command                   |
| `--version`                 | `-V`  | Show the current Kopi version                              |
| `--verbose`                 | `-v`  | Increase output verbosity (-v info, -vv debug, -vvv trace) |
| `--no-progress`             |       | Disable progress indicators                                |
| `--lock-timeout <DURATION>` |       | Override lock acquisition timeout (seconds or "infinite")  |

### Lock Timeout Configuration

The `--lock-timeout` flag controls how long Kopi waits to acquire locks before timing out. This global flag overrides all other timeout sources including environment variables and configuration files.

**Accepted values:**

- Positive integers representing seconds: `30`, `120`, `600`
- The literal string `infinite` to wait indefinitely

**Timeout resolution order (highest to lowest precedence):**

1. CLI flag: `--lock-timeout`
2. Environment variable: `KOPI_LOCK_TIMEOUT`
3. Configuration file: `locking.timeout` in `~/.kopi/config.toml`
4. Built-in default: 600 seconds (10 minutes)

**Usage examples:**

```bash
# Wait up to 30 seconds for installation lock
kopi --lock-timeout 30 install temurin@21

# Wait indefinitely for cache refresh
kopi --lock-timeout infinite cache refresh

# Apply to any operation that acquires locks
kopi --lock-timeout 120 uninstall corretto@17
```

**When timeout errors occur:**

Kopi reports the effective timeout value and where it was configured, making it easy to adjust settings. Lock timeouts indicate another Kopi process is holding the lock. Common solutions include waiting for the other operation to complete, increasing the timeout value, or checking for stuck processes.

For more details about the locking system, see the [Locking concept page](../concepts/locking.md).

## Core Commands

### kopi install

Install a JDK version. The install command accepts a version specification such as "21" for the latest JDK 21, "temurin\@21" for a specific distribution, or "corretto\@17.0.9" for an exact version.

| Option                | Short | Description                                                                |
| --------------------- | ----- | -------------------------------------------------------------------------- |
| `--force`             | `-f`  | Force reinstall even if the version already exists                         |
| `--dry-run`           |       | Preview what would be installed without performing the actual installation |
| `--timeout <SECONDS>` |       | Specify download timeout in seconds                                        |

The command has an alias "i" for convenience.

#### Install Examples

**Install latest JDK 21:**

```bash
kopi install 21
```

**Install specific distribution:**

```bash
kopi install temurin@21
```

**Install exact version:**

```bash
kopi install corretto@17.0.9
```

**Preview installation:**

```bash
kopi install --dry-run 21
```

**Force reinstallation:**

```bash
kopi install --force temurin@17
```

**Install with custom timeout:**

```bash
kopi install --timeout 300 graalvm@21
```

### kopi uninstall

Remove an installed JDK. Provide a version or distribution filter to decide what gets removed. Use `--cleanup` on its own only when you want to clean up partial uninstall artifacts; running the command with no version and no cleanup flag results in an error.

| Option      | Short | Description                                           |
| ----------- | ----- | ----------------------------------------------------- |
| `--force`   | `-f`  | Skip confirmation prompts                             |
| `--dry-run` |       | Preview what would be removed without actual deletion |
| `--all`     |       | Uninstall all versions of a distribution              |
| `--cleanup` |       | Clean up failed or partial uninstall operations       |

The command has aliases "u" and "remove" for convenience.

#### Uninstall Examples

**Uninstall specific version:**

```bash
kopi uninstall temurin@21
```

**Skip confirmation:**

```bash
kopi uninstall --force corretto@17
```

**Preview removal:**

```bash
kopi uninstall --dry-run 11
```

**Remove all versions of a distribution:**

```bash
kopi uninstall --all temurin
```

**Clean up failed installations:**

```bash
kopi uninstall --cleanup
```

### kopi global

Set the global default JDK version that will be used system-wide. Pass a version specification such as "21" or "temurin\@17".

The command has aliases "g" and "default" for convenience.

#### Global Examples

**Set JDK 21 as global default:**

```bash
kopi global 21
```

**Set specific distribution as global:**

```bash
kopi global temurin@17
```

**Quick set using alias:**

```bash
kopi g 11
```

### kopi local

Set the JDK version for the current project by creating a .kopi-version file in the current directory. This version takes precedence over the global setting when working within the project directory.

The command has aliases "l" and "pin" for convenience.

#### Local Examples

**Set project to use JDK 21:**

```bash
kopi local 21
```

**Pin specific distribution for project:**

```bash
kopi local corretto@17.0.9
```

**Set project JDK using alias:**

```bash
kopi l temurin@11
```

### kopi shell

Set the JDK version for the current shell session only. This temporary override affects only the current shell and its child processes.

| Option            | Short | Description                                      |
| ----------------- | ----- | ------------------------------------------------ |
| `--shell <SHELL>` |       | Specify the shell type instead of auto-detection |

The command has an alias "use" for convenience.

#### Shell Examples

**Temporarily use JDK 17:**

```bash
kopi shell 17
```

**Test with specific version:**

```bash
kopi use temurin@21
```

**Override shell detection:**

```bash
kopi shell --shell bash 11
```

### kopi current

Display the currently active JDK version and its source (shell, local, global, or system).

| Option    | Short | Description                       |
| --------- | ----- | --------------------------------- |
| `--quiet` | `-q`  | Show only the version number      |
| `--json`  |       | Output information in JSON format |

#### Current Examples

**Check current JDK:**

```bash
kopi current
```

**Get version only:**

```bash
kopi current --quiet
```

**Get JSON output:**

```bash
kopi current --json
```

### kopi env

Output environment variables for shell evaluation. This command generates the necessary environment variable settings (primarily JAVA_HOME) that can be evaluated by your shell to configure the JDK environment.

| Option            | Short | Description                                                    |
| ----------------- | ----- | -------------------------------------------------------------- |
| `<VERSION>`       |       | Use a specific JDK version instead of the current one          |
| `--shell <SHELL>` |       | Specify the shell type instead of auto-detection               |
| `--export`        |       | Control whether export statements are included (default: true) |

Usage varies by shell:

- For Bash and Zsh, use eval with command substitution
- For Fish, pipe the output to source
- For PowerShell, pipe the output to Invoke-Expression

#### Env Examples

**Set environment for current JDK (Bash/Zsh):**

```bash
eval "$(kopi env)"
```

**Set environment for specific version (Fish):**

```bash
kopi env 21 | source
```

**Set environment in PowerShell:**

```powershell
kopi env | Invoke-Expression
```

**Get raw environment variables:**

```bash
kopi env --export false
```

## Listing Commands

### kopi list

List all installed JDK versions along with their disk usage. Each entry reports the distribution and version (including any `+fx` suffix) and the size of that installation. The command does not highlight which JDK is currently active; use `kopi current` for that information.

The command has an alias "ls" for convenience.

#### List Examples

**List all installed JDKs:**

```bash
kopi list
```

**Quick list using alias:**

```bash
kopi ls
```

### kopi search

Search for available JDK versions in the metadata cache. This is a hidden alias for the cache search command. You can search by version number, distribution name, or combined specifications like "corretto\@21".

| Option       | Short | Description                                       |
| ------------ | ----- | ------------------------------------------------- |
| `--compact`  | `-c`  | Minimal output showing only version numbers       |
| `--detailed` | `-d`  | Comprehensive information including download URLs |
| `--json`     |       | Machine-readable output                           |
| `--lts-only` |       | Show only Long Term Support versions              |

The command has a visible alias "s" and hidden aliases "ls-remote" and "list-remote" for convenience.

#### Search Examples

**Search for JDK 21 versions:**

```bash
kopi search 21
```

**Search specific distribution:**

```bash
kopi search corretto
```

**Find LTS versions only:**

```bash
kopi search --lts-only
```

**Get compact listing:**

```bash
kopi search -c temurin
```

**Get detailed information:**

```bash
kopi search --detailed graalvm@21
```

## Utility Commands

### kopi which

Show the installation path for a JDK version. Without arguments, shows the path for the currently active JDK. With a version argument, shows the path for that specific version.

| Option          | Short | Description                                                |
| --------------- | ----- | ---------------------------------------------------------- |
| `--tool <TOOL>` |       | Show the path for a specific JDK tool (defaults to "java") |
| `--home`        |       | Show the JDK home directory instead of the executable path |
| `--json`        |       | Machine-readable output                                    |

The command has an alias "w" for convenience.

#### Which Examples

**Find current JDK path:**

```bash
kopi which
```

**Find specific version path:**

```bash
kopi which 21
```

**Get JAVA_HOME directory:**

```bash
kopi which --home
```

**Find javac path:**

```bash
kopi which --tool javac
```

**Get path for specific tool and version:**

```bash
kopi which --tool jar temurin@17
```

### kopi doctor

Run diagnostics on your Kopi installation. This command checks for common issues across various categories including installation, shell configuration, JDKs, permissions, network, and cache.

| Option               | Short | Description                                                                                   |
| -------------------- | ----- | --------------------------------------------------------------------------------------------- |
| `--json`             |       | Output results in JSON format for automation                                                  |
| `--check <CATEGORY>` |       | Run only specific category of checks (installation, shell, jdks, permissions, network, cache) |

Available check categories:

- **installation**: Verifies Kopi binary, version, directories, configuration files, and shims in PATH
- **shell**: Checks shell detection, PATH configuration, shell configuration, and shim functionality
- **jdks**: Validates JDK installations, integrity, disk space usage, and version consistency
- **permissions**: Examines directory and binary file permissions
- **network**: Tests API connectivity, DNS resolution, proxy configuration, and TLS verification
- **cache**: Inspects cache files, permissions, format validity, staleness, and size

#### Detailed Check Information

**Installation Checks:**

- Kopi Binary Check: Verifies the kopi executable exists and is accessible
- Installation Directory Check: Confirms all required Kopi directories exist and are writable
- Config File Check: Validates configuration file syntax and contents
- Shims In Path Check: Ensures the shims directory is in your PATH

**Shell Checks:**

- Shell Detection Check: Verifies Kopi can detect your current shell
- Path Check: Confirms PATH variable contains the shims directory
- Shell Configuration Check: Validates shell-specific setup
- Shim Functionality Check: Tests that shims can execute successfully

**JDK Checks:**

- JDK Installation Check: Verifies all installed JDKs have valid directory structures
- JDK Integrity Check: Confirms JDK binaries are present and executable
- JDK Version Consistency Check: Validates version metadata matches actual installations
- JDK Disk Space Check: Reports disk usage for each installed JDK

**Permission Checks:**

- Directory Permissions Check: Validates Kopi directories have correct permissions
- Binary Permissions Check: Ensures executables have appropriate execute permissions

**Network Checks:**

- API Connectivity Check: Tests connection to metadata sources
- DNS Resolution Check: Verifies DNS can resolve required hostnames
- Proxy Configuration Check: Validates proxy settings if configured
- TLS Verification Check: Tests HTTPS certificate validation

**Cache Checks:**

- Cache File Check: Verifies cache files exist and are accessible
- Cache Permissions Check: Confirms cache files have correct permissions
- Cache Format Check: Validates cache file structure and format
- Cache Staleness Check: Reports cache age and staleness
- Cache Size Check: Reports disk space used by cache

#### Performance Characteristics

The doctor command is designed for predictable execution:

- **Sequential execution**: Runs checks category by category while updating progress output
- **Network timeouts**: Network probes use a 5-second timeout per request to avoid long stalls
- **Progress indicator**: Shows check progress in interactive terminals
- **Resilient reporting**: Continues even if individual checks fail so you see every result

#### Result Interpretation

Check results are displayed with color coding:

- **Green (Pass)**: Check completed successfully, no issues found
- **Yellow (Warning)**: Check completed but found potential issues
- **Red (Fail)**: Check failed or found critical problems

Each failed or warning check includes:

- Description of what was checked
- Specific issue(s) found
- Actionable recommendations for resolution

#### Exit Codes

- `0`: All checks passed
- `1`: One or more checks failed
- `2`: Warnings detected (no failures)

Use verbose mode for detailed diagnostic information:

```bash
kopi -v doctor         # Show info-level logs
kopi -vv doctor        # Show debug-level logs
kopi -vvv doctor       # Show trace-level logs
```

#### Doctor Examples

**Run full diagnostic:**

```bash
kopi doctor
```

**Check specific category:**

```bash
kopi doctor --check shell
kopi doctor --check jdks
kopi doctor --check network
```

**Get JSON output for automation:**

```bash
kopi doctor --json
kopi doctor --check cache --json
```

### kopi setup

Perform initial setup and configuration of Kopi. This command creates necessary directories, installs shims, and configures shell integration.

| Option    | Short | Description                               |
| --------- | ----- | ----------------------------------------- |
| `--force` | `-f`  | Recreate shims even if they already exist |

#### Setup Examples

**Initial Kopi setup:**

```bash
kopi setup
```

**Recreate all shims:**

```bash
kopi setup --force
```

## Cache Commands

### kopi cache

Manage the JDK metadata cache. This command has several subcommands for different cache operations.

#### kopi cache refresh

Refresh metadata from the foojay.io API. This updates the local cache with the latest available JDK versions and distributions.

##### Cache Refresh Examples

**Update cache:**

```bash
kopi cache refresh
```

#### kopi cache info

Display information about the current cache state, including last update time, cache location, and statistics.

##### Cache Info Examples

**Check cache status:**

```bash
kopi cache info
```

#### kopi cache clear

Clear all cached metadata. This removes all cached data and forces a fresh download on the next operation that requires metadata.

##### Cache Clear Examples

**Clear entire cache:**

```bash
kopi cache clear
```

#### kopi cache search

Search for available JDK versions in the cache. Accepts a query string to search for specific versions or distributions.

| Option                   | Short | Description                                   |
| ------------------------ | ----- | --------------------------------------------- |
| `--compact`              |       | Minimal output                                |
| `--detailed`             |       | Comprehensive information                     |
| `--json`                 |       | Machine-readable output                       |
| `--lts-only`             |       | Show only Long Term Support versions          |
| `--java-version`         |       | Force searching by java_version field         |
| `--distribution-version` |       | Force searching by distribution_version field |

##### Cache Search Examples

**Search cached versions:**

```bash
kopi cache search 21
```

**Search by Java version field:**

```bash
kopi cache search --java-version 21.0.1
```

**Search by distribution version:**

```bash
kopi cache search --distribution-version 17.0.9+9
```

#### kopi cache list-distributions

List all available distributions in the cache, showing which vendors and versions are available for installation.

##### Cache List-Distributions Examples

**List all distributions:**

```bash
kopi cache list-distributions
```

### kopi refresh

Hidden command that serves as an alias for cache refresh. Refreshes the JDK metadata from the foojay.io API.

The command has a visible alias "r" for convenience.

#### Refresh Examples

**Quick cache refresh:**

```bash
kopi refresh
```

**Using alias:**

```bash
kopi r
```

## Shim Commands

### kopi shim

Manage tool shims that enable automatic JDK switching. This command has several subcommands for shim operations.

#### kopi shim add

Add a shim for a specific JDK tool. Pass the tool name (such as "java", "javac", "jar") to create a shim for that tool.

| Option    | Short | Description                                |
| --------- | ----- | ------------------------------------------ |
| `--force` | `-f`  | Force creation even if shim already exists |

##### Shim Add Examples

**Add java shim:**

```bash
kopi shim add java
```

**Add javac shim:**

```bash
kopi shim add javac
```

**Force recreate shim:**

```bash
kopi shim add --force jar
```

#### kopi shim remove

Remove a shim for a specific JDK tool. Pass the tool name to remove its shim.

##### Shim Remove Examples

**Remove javadoc shim:**

```bash
kopi shim remove javadoc
```

**Remove jshell shim:**

```bash
kopi shim remove jshell
```

#### kopi shim list

List all installed shims.

| Option                  | Short | Description                                        |
| ----------------------- | ----- | -------------------------------------------------- |
| `--available`           |       | Show available tools that could have shims created |
| `--distribution <DIST>` | `-d`  | Filter by distribution                             |

##### Shim List Examples

**List installed shims:**

```bash
kopi shim list
```

**Show available tools:**

```bash
kopi shim list --available
```

**Filter by distribution:**

```bash
kopi shim list -d temurin
```

#### kopi shim verify

Verify and repair shims. Checks that all shims are properly installed and functioning.

| Option  | Short | Description                           |
| ------- | ----- | ------------------------------------- |
| `--fix` |       | Automatically repair any issues found |

##### Shim Verify Examples

**Check shim integrity:**

```bash
kopi shim verify
```

**Auto-repair shims:**

```bash
kopi shim verify --fix
```

## Command Aliases

Many Kopi commands have shorter aliases for convenience:

| Command     | Aliases                         | Description                       |
| ----------- | ------------------------------- | --------------------------------- |
| `install`   | `i`                             | Install a JDK version             |
| `uninstall` | `u`, `remove`                   | Uninstall a JDK                   |
| `list`      | `ls`                            | List installed JDK versions       |
| `search`    | `s`, `ls-remote`, `list-remote` | Search available JDK versions     |
| `global`    | `g`, `default`                  | Set global default JDK            |
| `local`     | `l`, `pin`                      | Set project-specific JDK          |
| `shell`     | `use`                           | Set JDK for current shell session |
| `which`     | `w`                             | Show JDK installation path        |
| `refresh`   | `r`                             | Refresh metadata cache            |
