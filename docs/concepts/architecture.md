# Architecture

Kopi's internal architecture and core components.

## Overview

Kopi is a Rust-based JDK version manager that intercepts Java commands and automatically manages JDK versions:

```mermaid
graph TB
    User["User runs:<br/>java MyApp.java"]
    Shims["Kopi Shims<br/>(Command Interception)"]
    Resolver["Version Resolver<br/>(Find Required JDK)"]
    JDK["Correct JDK<br/>(~/.kopi/jdks/)"]

    User --> Shims
    Shims --> Resolver
    Resolver --> JDK
    JDK --> User

    VersionFiles[".kopi-version<br/>.java-version"]
    Metadata["JDK Metadata<br/>(Available Versions)"]
    Config["Configuration<br/>(config.toml)"]

    Resolver -.-> VersionFiles
    Resolver -.-> Config
    Shims -.-> Metadata

    style Shims fill:#f9f,stroke:#333,stroke-width:2px
    style Resolver fill:#bbf,stroke:#333,stroke-width:2px
```

Key components:

- **Shims**: Intercept Java commands and route to correct JDK
- **Version Resolver**: Determines which JDK version to use
- **Metadata System**: Tracks available JDK distributions and versions
- **Storage Manager**: Manages installed JDKs in `~/.kopi/jdks/`

## Core Components

### 1. Shim System (`src/shim/`)

Shims are Rust binaries that intercept Java tool invocations:

```mermaid
flowchart LR
    User["User executes:<br/>java MyApp.java"]
    Shim["Shim Binary<br/>(~/.kopi/shims/java)"]
    Validate["Security Validation<br/>(tool name & path)"]
    Resolve["Version Resolution<br/>(find required JDK)"]
    AutoInstall{"JDK<br/>Installed?"}
    Prompt["Auto-install<br/>Prompt"]
    Execute["Execute Tool<br/>(JDK/bin/java)"]
    Output["Output to User"]

    User --> Shim
    Shim --> Validate
    Validate --> Resolve
    Resolve --> AutoInstall
    AutoInstall -->|No| Prompt
    AutoInstall -->|Yes| Execute
    Prompt -->|Approved| Execute
    Execute --> Output

    style Shim fill:#f9f,stroke:#333,stroke-width:2px
    style AutoInstall fill:#ffd,stroke:#333,stroke-width:2px
```

**Key features:**

- Security validation for tool names and paths
- Auto-installation capability with user prompts
- Platform-specific executable resolution
- Tool discovery and validation
- Minimal overhead through Rust implementation

### 2. Version Resolution (`src/version/resolver.rs`)

Hierarchical version resolution with distribution support:

```mermaid
flowchart TD
    Start["Version Resolution"]
    EnvVar{"KOPI_JAVA_VERSION<br/>set?"}
    KopiFile{".kopi-version<br/>exists?"}
    JavaFile{".java-version<br/>exists?"}
    Parent{"Parent dir<br/>has version?"}
    Global{"Global default<br/>~/.kopi/version?"}
    Error["Error: No version"]
    Resolved["Version Resolved"]

    Start --> EnvVar
    EnvVar -->|Yes| Resolved
    EnvVar -->|No| KopiFile
    KopiFile -->|Yes| Resolved
    KopiFile -->|No| JavaFile
    JavaFile -->|Yes| Resolved
    JavaFile -->|No| Parent
    Parent -->|Yes| Resolved
    Parent -->|No| Global
    Global -->|Yes| Resolved
    Global -->|No| Error

    style EnvVar fill:#ffd,stroke:#333,stroke-width:2px
    style Resolved fill:#dfd,stroke:#333,stroke-width:2px
    style Error fill:#fdd,stroke:#333,stroke-width:2px
```

**Version format support:**

- Simple version: `21` or `17.0.9`
- Distribution-specific: `temurin@21` or `corretto@17`
- Semantic matching: `17` matches `17.0.15`
- File formats: `.kopi-version` (native) and `.java-version` (compatibility)

### 3. Metadata System (`src/metadata/`)

Multi-source metadata provider with intelligent fallback:

```mermaid
flowchart TD
    Request["Metadata Request"]
    HTTP{"HTTP Source<br/>Available?"}
    Foojay{"Foojay API<br/>Available?"}
    Local{"Local Scan<br/>Available?"}
    Cache{"Cache Valid?<br/>(< 30 days)"}
    StaleCache{"Stale Cache<br/>Exists?"}

    HTTPFetch["Fetch from<br/>GitHub Pages"]
    FoojayFetch["Query<br/>Disco API"]
    LocalScan["Scan Local<br/>JDKs"]
    LoadCache["Load from<br/>Cache"]
    UseStale["Use Stale<br/>Cache"]
    UpdateCache["Update<br/>Cache"]
    Return["Return<br/>Metadata"]
    Error["Error"]

    Request --> Cache
    Cache -->|Yes| LoadCache
    Cache -->|No| HTTP

    HTTP -->|Yes| HTTPFetch
    HTTP -->|No| Foojay

    HTTPFetch --> UpdateCache

    Foojay -->|Yes| FoojayFetch
    Foojay -->|No| Local

    FoojayFetch --> UpdateCache

    Local -->|Yes| LocalScan
    Local -->|No| StaleCache

    LocalScan --> UpdateCache

    StaleCache -->|Yes| UseStale
    StaleCache -->|No| Error

    LoadCache --> Return
    UpdateCache --> Return
    UseStale --> Return

    style Cache fill:#bbf,stroke:#333,stroke-width:2px
    style UpdateCache fill:#bfb,stroke:#333,stroke-width:2px
    style Error fill:#fdd,stroke:#333,stroke-width:2px
```

**Metadata features:**

- Lazy loading of package details
- Platform-specific filtering
- Distribution-aware caching
- Batch processing capabilities
- 30-day cache TTL with stale fallback

### 4. Configuration System (`src/config.rs`)

Hierarchical configuration with environment overrides:

```mermaid
graph TD
    subgraph "Configuration Sources"
        TOML["config.toml<br/>(User Config)"]
        ENV["Environment<br/>(KOPI_* vars)"]
        Defaults["Built-in<br/>Defaults"]
    end

    subgraph "Configuration Structure"
        Storage["Storage Config<br/>(paths, locations)"]
        AutoInstall["Auto-install<br/>(enabled, timeout)"]
        Shims["Shim Config<br/>(tools, paths)"]
        Cache["Cache Config<br/>(TTL, location)"]
        Metadata["Metadata Config<br/>(sources, URLs)"]
    end

    ENV -->|Override| TOML
    TOML -->|Override| Defaults

    Defaults --> Storage
    Defaults --> AutoInstall
    Defaults --> Shims
    Defaults --> Cache
    Defaults --> Metadata

    style ENV fill:#ffd,stroke:#333,stroke-width:2px
    style TOML fill:#bbf,stroke:#333,stroke-width:2px
```

### 5. Storage Layout

```
~/.kopi/                        # KOPI_HOME
├── shims/                      # Executable shims
│   ├── java                    # Shim for java command
│   ├── javac                   # Shim for javac command
│   └── [other JDK tools]
├── jdks/                       # Installed JDKs
│   ├── temurin-21.0.2/         # Distribution-version format
│   │   └── bin/
│   │       ├── java
│   │       └── javac
│   ├── corretto-17.0.9/
│   └── graalvm-21.0.1/
├── cache/                      # Metadata cache
│   ├── distributions/          # Per-distribution caches
│   │   ├── temurin.json
│   │   └── corretto.json
│   └── metadata.json           # Combined metadata
├── downloads/                  # Temporary download directory
├── version                     # Global default version
└── config.toml                 # User configuration
```

## Next Steps

- [Version Files](version-files.md) - Version configuration
- [Shims](shims.md) - Command interception details
- [Metadata System](metadata.md) - JDK discovery and metadata
- [Caching](caching.md) - Cache management
