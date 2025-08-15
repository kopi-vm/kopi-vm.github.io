# JDK Distributions

Understanding the different JDK distributions available through Kopi.

## What are JDK Distributions?

JDK distributions are different builds of OpenJDK provided by various vendors. While they all implement the same Java specifications, they may differ in:

- Support and maintenance policies
- Performance optimizations
- Additional tools and features
- Licensing terms
- Platform availability

## Available Distributions

### Eclipse Temurin

**Vendor**: Eclipse Foundation  
**License**: GPLv2 with Classpath Exception  
**Recommended for**: General purpose, production use

```bash
kopi install temurin@21
```

Features:
- High-quality TCK-tested builds
- Wide platform support
- Long-term support (LTS) versions
- Active community support

### Amazon Corretto

**Vendor**: Amazon  
**License**: GPLv2 with Classpath Exception  
**Recommended for**: AWS deployments, production use

```bash
kopi install corretto@21
```

Features:
- Production-ready builds
- Long-term support from Amazon
- Performance improvements
- Security patches
- No-cost support

### Azul Zulu

**Vendor**: Azul Systems  
**License**: GPLv2 with Classpath Exception  
**Recommended for**: Enterprise applications

```bash
kopi install zulu@21
```

Features:
- Certified OpenJDK builds
- Wide platform support
- LTS versions available
- Commercial support available

### GraalVM

**Vendor**: Oracle  
**License**: GPLv2 with Classpath Exception  
**Recommended for**: High-performance applications, native compilation

```bash
kopi install graalvm@21
```

Features:
- Native image compilation
- Polyglot programming
- Advanced JIT compiler
- Performance optimizations

### Microsoft OpenJDK

**Vendor**: Microsoft  
**License**: GPLv2 with Classpath Exception  
**Recommended for**: Azure deployments

```bash
kopi install microsoft@21
```

Features:
- Optimized for Azure
- Quarterly updates
- LTS support
- Container-ready

### Oracle OpenJDK

**Vendor**: Oracle  
**License**: GPLv2 with Classpath Exception  
**Recommended for**: Development, short-term projects

```bash
kopi install oracle_open_jdk@21
```

Features:
- Reference implementation
- Latest features
- Short support cycle (6 months)

### IBM Semeru

**Vendor**: IBM  
**License**: GPLv2 with Classpath Exception  
**Recommended for**: Enterprise applications, IBM platforms

```bash
kopi install semeru@21
```

Features:
- OpenJ9 JVM
- Memory efficiency
- Fast startup
- Cloud optimizations

### BellSoft Liberica

**Vendor**: BellSoft  
**License**: GPLv2 with Classpath Exception  
**Recommended for**: Embedded systems, full JDK features

```bash
kopi install liberica@21
```

Features:
- Full JDK including JavaFX
- Wide platform support
- Lightweight builds available
- Commercial support

## Choosing a Distribution

### For Production

Consider these factors:

1. **Support**: Long-term support availability
2. **Stability**: Track record of stable releases
3. **Performance**: Optimization for your workload
4. **Compliance**: TCK certification
5. **Cost**: Support and licensing costs

Recommended: Temurin, Corretto, Zulu

### For Development

Consider:

1. **Features**: Latest Java features
2. **Tools**: Development tools included
3. **Compatibility**: Match production environment

Recommended: Temurin, Oracle OpenJDK

### For Special Requirements

- **Native Compilation**: GraalVM
- **Memory Efficiency**: Semeru (OpenJ9)
- **JavaFX**: Liberica, Zulu
- **AWS**: Corretto
- **Azure**: Microsoft OpenJDK

## Platform Availability

Check distribution availability for your platform:

```bash
# List all distributions for current platform
kopi search

# Check specific distribution
kopi search corretto
```

### Platform Support Matrix

| Distribution | Linux x64 | Linux ARM64 | macOS x64 | macOS ARM64 | Windows x64 |
|-------------|-----------|-------------|-----------|-------------|-------------|
| Temurin     | ✓         | ✓           | ✓         | ✓           | ✓           |
| Corretto    | ✓         | ✓           | ✓         | ✓           | ✓           |
| Zulu        | ✓         | ✓           | ✓         | ✓           | ✓           |
| GraalVM     | ✓         | ✓           | ✓         | ✓           | ✓           |
| Microsoft   | ✓         | ✓           | ✓         | ✓           | ✓           |
| Semeru      | ✓         | ✓           | ✓         | ✓           | ✓           |
| Liberica    | ✓         | ✓           | ✓         | ✓           | ✓           |

## Version Comparison

### Listing Versions

Compare available versions across distributions:

```bash
# List all JDK 21 versions
kopi search 21

# Compare specific distributions
kopi search "temurin@21"
kopi search "corretto@21"
```

### Feature Differences

Most distributions include:
- Core JDK tools (javac, java, jar, etc.)
- Standard libraries
- JVM

Some distributions additionally include:
- JavaFX (Liberica, Zulu)
- Native image tools (GraalVM)
- Performance monitoring tools
- Cloud-specific optimizations

## Configuration

### Setting Default Distribution

Configure preferred distribution in `~/.kopi/config.toml`:

```toml
[defaults]
distribution = "temurin"

[preferences]
distributions = ["temurin", "corretto", "zulu"]
```

### Distribution Aliases

Create shortcuts for distributions:

```toml
[aliases]
java = "temurin@21"
graal = "graalvm@21"
aws = "corretto@21"
```

## Migration Between Distributions

### Switching Distributions

```bash
# Uninstall old distribution
kopi uninstall temurin@21

# Install new distribution
kopi install corretto@21

# Update project configuration
kopi pin corretto@21
```

### Testing Compatibility

```bash
# Test with different distributions
kopi shell temurin@21
./gradlew test

kopi shell corretto@21
./gradlew test
```

## Best Practices

1. **Standardize on one distribution** for production
2. **Test with target distribution** during development
3. **Document distribution requirements** in project
4. **Consider support lifecycle** for LTS planning
5. **Evaluate performance** for your specific workload

## Next Steps

- [Managing Versions](managing-versions.md) - Version management
- [Configuration](../reference/configuration.md) - Advanced configuration
- [Supported Distributions](../reference/distributions.md) - Complete list