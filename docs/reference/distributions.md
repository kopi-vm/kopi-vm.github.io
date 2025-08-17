# Supported Distributions

Complete list of JDK distributions supported by Kopi.

## Distribution Overview

| Distribution       | Vendor             | License     | LTS Support   | Best For                      |
| ------------------ | ------------------ | ----------- | ------------- | ----------------------------- |
| **Temurin**        | Eclipse Foundation | GPL v2 + CE | Yes           | General use, production       |
| **Corretto**       | Amazon             | GPL v2 + CE | Yes           | AWS deployments               |
| **Zulu**           | Azul Systems       | GPL v2 + CE | Yes           | Enterprise applications       |
| **GraalVM**        | Oracle             | GPL v2 + CE | Yes           | Native compilation, polyglot  |
| **Microsoft**      | Microsoft          | GPL v2 + CE | Yes           | Azure deployments             |
| **Semeru**         | IBM                | GPL v2 + CE | Yes           | OpenJ9 JVM, memory efficiency |
| **Liberica**       | BellSoft           | GPL v2 + CE | Yes           | Full JDK with JavaFX          |
| **Dragonwell**     | Alibaba            | GPL v2 + CE | Yes           | Optimized for Alibaba Cloud   |
| **SapMachine**     | SAP                | GPL v2 + CE | Yes           | SAP applications              |
| **Oracle OpenJDK** | Oracle             | GPL v2 + CE | No (6 months) | Latest features               |

\*GPL v2 + CE = GPLv2 with Classpath Exception

## Distribution Details

### Eclipse Temurin

**Identifier**: `temurin`  
**Vendor**: Eclipse Foundation (formerly AdoptOpenJDK)  
**Website**: https://adoptium.net/

```bash
# Installation
kopi install temurin@21
kopi install temurin@17.0.9
kopi install temurin@11.0.21
```

**Features**:

- High-quality TCK-tested builds
- Wide platform support
- Regular security updates
- Strong community support
- Default choice for Kopi

**Available Versions**: 8, 11, 17, 18, 19, 20, 21, 22

### Amazon Corretto

**Identifier**: `corretto`  
**Vendor**: Amazon Web Services  
**Website**: https://aws.amazon.com/corretto/

```bash
# Installation
kopi install corretto@21
kopi install corretto@17.0.9.8.1
kopi install corretto@11.0.21.9.1
```

**Features**:

- Production-ready builds
- Long-term support from AWS
- Performance improvements
- Optimized for Amazon Linux
- No-cost commercial support

**Available Versions**: 8, 11, 17, 21, 22

### Azul Zulu

**Identifier**: `zulu`  
**Vendor**: Azul Systems  
**Website**: https://www.azul.com/downloads/

```bash
# Installation
kopi install zulu@21
kopi install zulu@17.46.19
kopi install zulu@11.68.17
```

**Features**:

- Certified OpenJDK builds
- Wide platform support
- Commercial support available
- Zulu Prime for performance
- Cloud-optimized builds

**Available Versions**: 6, 7, 8, 11, 13, 15, 17, 18, 19, 20, 21, 22

### GraalVM

**Identifier**: `graalvm`, `graalvm_community`  
**Vendor**: Oracle  
**Website**: https://www.graalvm.org/

```bash
# Installation
kopi install graalvm@21
kopi install graalvm_community@21
kopi install graalvm@17.0.9
```

**Features**:

- Native image compilation
- Polyglot programming (JavaScript, Python, Ruby, R)
- Advanced JIT compiler
- Ahead-of-time compilation
- Superior performance for specific workloads

**Available Versions**: 17, 21, 22

### Microsoft OpenJDK

**Identifier**: `microsoft`  
**Vendor**: Microsoft  
**Website**: https://www.microsoft.com/openjdk

```bash
# Installation
kopi install microsoft@21
kopi install microsoft@17.0.9
kopi install microsoft@11.0.21
```

**Features**:

- Optimized for Azure
- Container-ready builds
- Quarterly updates
- Windows native integration
- Support for Alpine Linux

**Available Versions**: 11, 17, 21

### IBM Semeru

**Identifier**: `semeru`, `semeru_certified`  
**Vendor**: IBM  
**Website**: https://developer.ibm.com/languages/java/semeru-runtimes/

```bash
# Installation
kopi install semeru@21
kopi install semeru_certified@17
kopi install semeru@11.0.21
```

**Features**:

- OpenJ9 JVM (not HotSpot)
- Superior memory efficiency
- Fast startup times
- Cloud-optimized
- Certified edition available

**Available Versions**: 8, 11, 17, 18, 19, 20, 21, 22

### BellSoft Liberica

**Identifier**: `liberica`, `liberica_native`  
**Vendor**: BellSoft  
**Website**: https://bell-sw.com/

```bash
# Installation
kopi install liberica@21
kopi install liberica_native@21
kopi install liberica@17.0.9
```

**Features**:

- Full JDK including JavaFX
- Lite builds available
- Native image kit
- Wide platform support
- Embedded systems support

**Available Versions**: 8, 11, 17, 18, 19, 20, 21, 22

### Alibaba Dragonwell

**Identifier**: `dragonwell`  
**Vendor**: Alibaba  
**Website**: https://dragonwell-jdk.io/

```bash
# Installation
kopi install dragonwell@21
kopi install dragonwell@17.0.9.0.9
kopi install dragonwell@11.0.20.17
```

**Features**:

- Optimized for Alibaba Cloud
- Enhanced for multi-tenant environments
- Improved GC performance
- Coroutine support
- Production-proven at scale

**Available Versions**: 8, 11, 17, 21

### SAP SapMachine

**Identifier**: `sap_machine`  
**Vendor**: SAP  
**Website**: https://sap.github.io/SapMachine/

```bash
# Installation
kopi install sap_machine@21
kopi install sap_machine@17.0.9
kopi install sap_machine@11.0.21
```

**Features**:

- Optimized for SAP applications
- Regular releases
- Commercial support from SAP
- PowerPC support
- Container images available

**Available Versions**: 11, 17, 19, 20, 21, 22

### Oracle OpenJDK

**Identifier**: `oracle_open_jdk`  
**Vendor**: Oracle  
**Website**: https://openjdk.org/

```bash
# Installation
kopi install oracle_open_jdk@22
kopi install oracle_open_jdk@21
kopi install oracle_open_jdk@20
```

**Features**:

- Reference implementation
- Latest Java features first
- Short support cycle (6 months)
- Good for development
- Not recommended for production

**Available Versions**: 19, 20, 21, 22

## Additional Distributions

### Oracle JDK

**Identifier**: `oracle`  
**License**: Oracle No-Fee Terms and Conditions  
**Note**: Commercial license may be required

```bash
kopi install oracle@21
```

### Red Hat OpenJDK

**Identifier**: `redhat`  
**Vendor**: Red Hat  
**Note**: Primarily for RHEL systems

```bash
kopi install redhat@17
```

### JetBrains Runtime

**Identifier**: `jetbrains`  
**Vendor**: JetBrains  
**Note**: Optimized for JetBrains IDEs

```bash
kopi install jetbrains@21
```

### Trava OpenJDK

**Identifier**: `trava`  
**Vendor**: Trava  
**Note**: Focused on security

```bash
kopi install trava@11
```

### Kona JDK

**Identifier**: `kona`  
**Vendor**: Tencent  
**Note**: Optimized for Tencent Cloud

```bash
kopi install kona@17
```

## Platform Support Matrix

### Linux x64

| Distribution | 8   | 11  | 17  | 21  | 22  |
| ------------ | --- | --- | --- | --- | --- |
| Temurin      | ✓   | ✓   | ✓   | ✓   | ✓   |
| Corretto     | ✓   | ✓   | ✓   | ✓   | ✓   |
| Zulu         | ✓   | ✓   | ✓   | ✓   | ✓   |
| GraalVM      | -   | -   | ✓   | ✓   | ✓   |
| Microsoft    | -   | ✓   | ✓   | ✓   | -   |
| Semeru       | ✓   | ✓   | ✓   | ✓   | ✓   |
| Liberica     | ✓   | ✓   | ✓   | ✓   | ✓   |
| Dragonwell   | ✓   | ✓   | ✓   | ✓   | -   |

### Linux ARM64

| Distribution | 8   | 11  | 17  | 21  | 22  |
| ------------ | --- | --- | --- | --- | --- |
| Temurin      | ✓   | ✓   | ✓   | ✓   | ✓   |
| Corretto     | ✓   | ✓   | ✓   | ✓   | ✓   |
| Zulu         | ✓   | ✓   | ✓   | ✓   | ✓   |
| GraalVM      | -   | -   | ✓   | ✓   | ✓   |
| Microsoft    | -   | ✓   | ✓   | ✓   | -   |
| Semeru       | -   | ✓   | ✓   | ✓   | -   |
| Liberica     | ✓   | ✓   | ✓   | ✓   | ✓   |

### macOS x64

| Distribution | 8   | 11  | 17  | 21  | 22  |
| ------------ | --- | --- | --- | --- | --- |
| Temurin      | ✓   | ✓   | ✓   | ✓   | ✓   |
| Corretto     | ✓   | ✓   | ✓   | ✓   | ✓   |
| Zulu         | ✓   | ✓   | ✓   | ✓   | ✓   |
| GraalVM      | -   | -   | ✓   | ✓   | ✓   |
| Microsoft    | -   | ✓   | ✓   | ✓   | -   |
| Semeru       | ✓   | ✓   | ✓   | ✓   | -   |
| Liberica     | ✓   | ✓   | ✓   | ✓   | ✓   |

### macOS ARM64 (Apple Silicon)

| Distribution | 8   | 11  | 17  | 21  | 22  |
| ------------ | --- | --- | --- | --- | --- |
| Temurin      | -   | ✓   | ✓   | ✓   | ✓   |
| Corretto     | -   | ✓   | ✓   | ✓   | ✓   |
| Zulu         | ✓   | ✓   | ✓   | ✓   | ✓   |
| GraalVM      | -   | -   | ✓   | ✓   | ✓   |
| Microsoft    | -   | ✓   | ✓   | ✓   | -   |
| Semeru       | -   | -   | ✓   | ✓   | -   |
| Liberica     | -   | ✓   | ✓   | ✓   | ✓   |

### Windows x64

| Distribution | 8   | 11  | 17  | 21  | 22  |
| ------------ | --- | --- | --- | --- | --- |
| Temurin      | ✓   | ✓   | ✓   | ✓   | ✓   |
| Corretto     | ✓   | ✓   | ✓   | ✓   | ✓   |
| Zulu         | ✓   | ✓   | ✓   | ✓   | ✓   |
| GraalVM      | -   | -   | ✓   | ✓   | ✓   |
| Microsoft    | -   | ✓   | ✓   | ✓   | -   |
| Semeru       | ✓   | ✓   | ✓   | ✓   | -   |
| Liberica     | ✓   | ✓   | ✓   | ✓   | ✓   |
| Dragonwell   | ✓   | ✓   | ✓   | ✓   | -   |

## Choosing a Distribution

### By Use Case

**General Development**:

- Temurin (recommended)
- Oracle OpenJDK
- Zulu

**Production Deployment**:

- Temurin
- Corretto (AWS)
- Microsoft (Azure)
- Zulu

**Memory-Constrained**:

- Semeru (OpenJ9)
- Liberica Lite
- GraalVM Native

**Native Compilation**:

- GraalVM
- Liberica Native
- Mandrel

**Full JDK with JavaFX**:

- Liberica Full
- Zulu with JavaFX

### By Platform

**Cloud Providers**:

- AWS → Corretto
- Azure → Microsoft
- Google Cloud → Temurin
- Alibaba Cloud → Dragonwell
- Tencent Cloud → Kona

**Operating Systems**:

- RHEL/CentOS → Red Hat
- Ubuntu/Debian → Temurin
- Alpine Linux → Liberica, Microsoft
- Windows → Any major distribution

### By Support

**Free Community Support**:

- Temurin
- Corretto
- Oracle OpenJDK

**Commercial Support Available**:

- Zulu (Azul)
- Liberica (BellSoft)
- Semeru (IBM)
- Oracle JDK

## Version Lifecycle

### LTS (Long Term Support) Versions

Current LTS versions with support timeline:

| Version | Release        | End of Support |
| ------- | -------------- | -------------- |
| 8       | 2014           | 2030+          |
| 11      | 2018           | 2026+          |
| 17      | 2021           | 2029+          |
| 21      | 2023           | 2031+          |
| 25      | 2025 (planned) | 2033+          |

### Non-LTS Versions

| Version | Release | End of Support |
| ------- | ------- | -------------- |
| 18      | 2022-03 | 2022-09        |
| 19      | 2022-09 | 2023-03        |
| 20      | 2023-03 | 2023-09        |
| 22      | 2024-03 | 2024-09        |
| 23      | 2024-09 | 2025-03        |

## Searching for Distributions

```bash
# Search all available distributions
kopi search

# Search specific distribution
kopi search temurin

# Find LTS versions
kopi search --lts-only

# Check distribution availability
kopi search "graalvm@21"
```

## Installation Examples

```bash
# Install default distribution (Temurin)
kopi install 21

# Install specific distribution
kopi install corretto@21
kopi install zulu@17.0.9
kopi install graalvm@21.0.1

# Install with exact version
kopi install temurin@21.0.2+13

# Install latest patch
kopi install temurin@21
```

## Troubleshooting

### Distribution Not Found

```bash
# Update metadata
kopi cache update

# Check available versions
kopi search <name>

# Try alternative name
kopi search | grep -i <name>
```

### Platform Not Supported

```bash
# Check platform support
kopi search <name> --detailed

# Try alternative distribution
kopi search <version>
```

## Next Steps

- [Commands](commands.md) - Command reference
- [JDK Distributions Guide](../guide/distributions.md) - Detailed comparison
- [Installation](../getting-started/installation.md) - Getting started
