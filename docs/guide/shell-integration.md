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

## Advanced Configuration

### Shell Hooks

#### Bash/Zsh - Directory Change Hook

For automatic Java version switching when entering project directories:

```bash
# Add to ~/.bashrc or ~/.zshrc
kopi_auto_env() {
    eval "$(kopi env)"
}

# Hook into directory changes
if [[ -n "$ZSH_VERSION" ]]; then
    # Zsh
    autoload -U add-zsh-hook
    add-zsh-hook chpwd kopi_auto_env
    kopi_auto_env  # Run on shell start
else
    # Bash
    PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}kopi_auto_env"
fi
```

#### Fish - Directory Change Event

```fish
# Add to ~/.config/fish/functions/kopi_auto_env.fish
function kopi_auto_env --on-variable PWD
    kopi env | source
end

# Run on shell start
kopi_auto_env
```

#### PowerShell - Directory Detection

```powershell
# Add to your PowerShell profile ($PROFILE)
# Auto-switch Java version on directory change
$global:KopiLastPwd = $PWD

function Invoke-KopiAutoEnv {
    if ($PWD.Path -ne $global:KopiLastPwd) {
        $global:KopiLastPwd = $PWD.Path
        if (Get-Command kopi -ErrorAction SilentlyContinue) {
            kopi env | Invoke-Expression
        }
    }
}

# Hook into prompt function for directory change detection
$global:OriginalPromptFunction = $function:prompt

function prompt {
    Invoke-KopiAutoEnv
    & $global:OriginalPromptFunction
}

# Run on shell start
Invoke-KopiAutoEnv
```

### Custom Prompts

Show active JDK in your prompt:

#### Bash

```bash
# Function to get current Java version
kopi_version() {
    if command -v kopi &> /dev/null; then
        kopi current 2>/dev/null | grep -oE '[0-9]+(\.[0-9]+)*' | head -1
    fi
}

# Add to PS1
PS1='[\u@\h \W$(kopi_version && echo " java:$(kopi_version)")]$ '
```

#### Zsh

```zsh
# Function to get current Java version
kopi_version() {
    if command -v kopi &> /dev/null; then
        kopi current 2>/dev/null | grep -oE '[0-9]+(\.[0-9]+)*' | head -1
    fi
}

# Add to PROMPT
PROMPT='%n@%m %1~ $(kopi_version >/dev/null && echo "java:$(kopi_version) ")%# '
```

#### Fish

```fish
function fish_prompt
    set -l java_version (kopi current 2>/dev/null | string match -r '\d+(\.\d+)*' | head -1)
    if test -n "$java_version"
        set_color yellow
        echo -n "java:$java_version "
        set_color normal
    end
    # ... rest of your prompt
    echo -n (whoami)@(hostname):$(pwd)'$ '
end
```

#### PowerShell

```powershell
function prompt {
    $javaVersion = & kopi current 2>$null | Select-String -Pattern '\d+(\.\d+)*' | ForEach-Object { $_.Matches[0].Value }
    if ($javaVersion) {
        Write-Host "[java:$javaVersion] " -NoNewline -ForegroundColor Yellow
    }
    # ... rest of your prompt
    return "> "
}
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

## Next Steps

- [Working with Projects](projects.md) - Project configuration
- [Managing Versions](managing-versions.md) - Version management
- [Troubleshooting](../troubleshooting.md) - Common issues
