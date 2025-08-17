# Installation

Kopi can be installed on macOS, Linux, and Windows using various package managers or from source.

## System Requirements

- **Operating System**: macOS, Linux, or Windows
- **Architecture**: x64 or ARM64
- **Shell**: Bash, Zsh, Fish, or PowerShell

## Installation Methods

### macOS

#### Using Homebrew

```bash
brew install kopi-vm/tap/kopi
```

### Linux

#### Ubuntu/Debian (using PPA)

```bash
# Import GPG key
curl -fsSL https://keyserver.ubuntu.com/pks/lookup?op=get\&search=0xD2AC04A5A34E9BE3A8B32784F507C6D3DB058848 | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/kopi-archive-keyring.gpg > /dev/null

# Add repository
echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/kopi-archive-keyring.gpg] \
  https://kopi-vm.github.io/ppa-kopi $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/kopi.list > /dev/null

# Install kopi
sudo apt update && sudo apt install kopi
```

### Windows

#### Using Windows Package Manager

```bash
winget install kopi
```

#### Using Scoop

```bash
# Add the kopi bucket
scoop bucket add kopi https://github.com/kopi-vm/scoop-kopi

# Install kopi
scoop install kopi
```

### All Platforms

#### Using Cargo

```bash
cargo install kopi
```

## Post-Installation Setup

After installing Kopi, you need to set up shell integration:

```bash
# Initialize Kopi
kopi setup
```

## Verify Installation

```bash
kopi --version
```

## Next Steps

- [Quick Start](quickstart.md) - Get up and running with Kopi
- [First Steps](first-steps.md) - Learn the basic commands
