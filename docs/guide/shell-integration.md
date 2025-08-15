# Shell Integration

Configure your shell to work seamlessly with Kopi for automatic JDK switching.

## How Shell Integration Works

Kopi uses **shims** - lightweight executable scripts that intercept Java commands and route them to the correct JDK version. When you run `java`, `javac`, or other JDK tools, Kopi's shims:

1. Detect the required JDK version
2. Set up the environment
3. Execute the actual JDK binary

## Supported Shells

- **Bash** (Linux, macOS)
- **Zsh** (macOS default, Linux)
- **Fish** (Linux, macOS)
- **PowerShell** (Windows)
- **Command Prompt** (Windows)

## Installation

### Bash

Add to your `~/.bashrc`:

```bash
# Add shims to PATH
export PATH="$HOME/.kopi/shims:$PATH"

# Set JAVA_HOME
eval "$(kopi env)"
```

### Zsh

Add to your `~/.zshrc`:

```bash
# Add shims to PATH
export PATH="$HOME/.kopi/shims:$PATH"

# Set JAVA_HOME
eval "$(kopi env)"
```

### Fish

Add to your `~/.config/fish/config.fish`:

```fish
# Add shims to PATH
set -gx PATH $HOME/.kopi/shims $PATH

# Set JAVA_HOME
kopi env | source
```

### PowerShell

Add to your PowerShell profile:

```powershell
# Add shims to PATH
$env:Path = "$env:USERPROFILE\.kopi\shims;$env:Path"

# Set JAVA_HOME
kopi env | Invoke-Expression
```

### Command Prompt (Windows)

Set system environment variables:

```batch
:: Add to system PATH
setx PATH "%USERPROFILE%\.kopi\shims;%PATH%"
```

## Verification

After configuring your shell:

```bash
# Reload shell configuration
source ~/.bashrc  # or ~/.zshrc

# Verify shims are in PATH
which java
# Should show: /home/user/.kopi/shims/java

# Test automatic switching
cd project-with-kopi-version
java --version  # Should use project's JDK
```

## Features

### Automatic Version Switching

Kopi automatically switches JDK when:

```bash
# Entering a directory with version file
cd my-project  # Has .kopi-version
java --version  # Uses project's JDK

# Moving to parent directory
cd ..
java --version  # Uses global or inherited JDK
```

### Command Interception

All JDK commands are intercepted:

```bash
# These all use Kopi's version management
java MyApp
javac MyApp.java
jar cf myapp.jar *.class
jshell
```

### Environment Variables

Kopi manages these environment variables:

```bash
# Automatically set by Kopi
echo $JAVA_HOME  # Points to active JDK
echo $PATH       # Includes JDK bin directory

# Manual override
export KOPI_VERSION=17  # Force JDK 17
```

## Advanced Configuration

### Shell Hooks

#### Bash/Zsh - Directory Change Hook

```bash
# Auto-switch on directory change
kopi_auto_switch() {
    if [ -f .kopi-version ] || [ -f .java-version ]; then
        kopi shell $(cat .kopi-version 2>/dev/null || cat .java-version)
    fi
}

# Add to PROMPT_COMMAND (Bash)
PROMPT_COMMAND="kopi_auto_switch; $PROMPT_COMMAND"

# Or use chpwd hook (Zsh)
autoload -U add-zsh-hook
add-zsh-hook chpwd kopi_auto_switch
```

#### Fish - Directory Change Event

```fish
function kopi_auto_switch --on-variable PWD
    if test -f .kopi-version
        kopi shell (cat .kopi-version)
    else if test -f .java-version
        kopi shell (cat .java-version)
    end
end
```

### Custom Prompts

Show active JDK in your prompt:

#### Bash

```bash
# Add to PS1
PS1='[$(kopi current --short)] \u@\h:\w\$ '
```

#### Zsh

```zsh
# Add to PROMPT
PROMPT='[$(kopi current --short)] %n@%m:%~%# '
```

#### Fish

```fish
function fish_prompt
    echo -n "[$(kopi current --short)] "
    echo -n (whoami)@(hostname):$(pwd)'$ '
end
```

### Aliases and Functions

Create helpful shortcuts:

```bash
# Quick JDK switching aliases
alias j8='kopi use 8'
alias j11='kopi use 11'
alias j17='kopi use 17'
alias j21='kopi use 21'

# Project setup function
setup_java_project() {
    kopi install $1
    kopi pin $1
    echo "Project configured with JDK $1"
}

# List Java processes with their JDK
java_ps() {
    ps aux | grep java | while read line; do
        pid=$(echo $line | awk '{print $2}')
        if [ -e /proc/$pid/exe ]; then
            jdk=$(readlink -f /proc/$pid/exe | sed 's|/bin/java||')
            echo "$line | JDK: $jdk"
        fi
    done
}
```

## Troubleshooting

### Shims Not Working

```bash
# Check PATH order
echo $PATH
# Ensure ~/.kopi/shims comes before system Java

# Verify shim installation
ls -la ~/.kopi/shims/

# Reinstall shims
kopi setup
```

### Wrong JDK Version

```bash
# Check version resolution
kopi current --verbose

# Clear any overrides
unset JAVA_HOME
unset KOPI_VERSION

# Verify shell integration
kopi doctor
```

### Performance Issues

```bash
# Check shim execution time
time java --version

# Update metadata cache
kopi cache update

# Optimize shell startup
# Move Kopi init to .bashrc instead of .bash_profile
```

### Shell-Specific Issues

#### Zsh

```bash
# If commands are not found
rehash

# For oh-my-zsh compatibility
# Add Kopi init after oh-my-zsh
```

#### Fish

```fish
# Update universal variables
set -U fish_user_paths $HOME/.kopi/shims $fish_user_paths
```

#### Windows

```powershell
# Refresh environment variables
refreshenv

# Or restart PowerShell
```

## Shell Integration Options

### Minimal Setup

Just add shims to PATH:

```bash
export PATH="$HOME/.kopi/shims:$PATH"
```

### With Environment Variables

Add shims and set JAVA_HOME:

```bash
# Add to PATH
export PATH="$HOME/.kopi/shims:$PATH"

# Set JAVA_HOME for current JDK
eval "$(kopi env)"
```

### Custom Integration

Build your own integration:

```bash
# Get Kopi paths
KOPI_SHIMS="$HOME/.kopi/shims"
KOPI_JDKS="$HOME/.kopi/jdks"

# Add to PATH
export PATH="$KOPI_SHIMS:$PATH"

# Set JAVA_HOME dynamically
export JAVA_HOME=$(kopi which --home)
```

## Best Practices

1. **PATH order matters** - Ensure shims come before system Java
2. **Avoid conflicts** - Remove other JDK version managers
3. **Use version files** - Let Kopi handle automatic switching
4. **Set environment early** - Add PATH and JAVA_HOME near the top of shell config
5. **Keep shells updated** - Update shell configs after Kopi updates

## Next Steps

- [CI/CD Integration](ci-cd.md) - Automated builds
- [Working with Projects](projects.md) - Project configuration
- [Troubleshooting](../troubleshooting.md) - Common issues