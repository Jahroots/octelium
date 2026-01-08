# Octelium Build System - Multi-Architecture Support

This document describes the multi-architecture build system for Octelium, which supports both AMD64 (x86-64) and ARM64 (aarch64) architectures.

## Architecture Support

### Supported Architectures

- **AMD64 (x86-64)**: Intel and AMD processors (standard Linux/macOS/Windows servers)
- **ARM64 (aarch64)**: Apple Silicon (M1/M2/M3+), AWS Graviton, Azure Ampere, and other ARM64-based systems

### Build Platform Compatibility

The Go compiler enables cross-compilation, so you can build binaries for any target architecture from any host platform (AMD64 or ARM64).

## Building Binaries

### Build CLI Tools

Build all CLI tools for a specific architecture:

```bash
# Build for AMD64
make build-cli-octelium-amd64
make build-cli-octeliumctl-amd64
make build-cli-octops-amd64

# Build for ARM64
make build-cli-octelium-arm64
make build-cli-octeliumctl-arm64
make build-cli-octops-arm64

# Build all CLI tools for both architectures
make build-cli-all-arch
```

### Build Cluster Components

Build individual cluster components for specific architectures:

```bash
# Build apiserver for AMD64
make build-apiserver-amd64

# Build apiserver for ARM64
make build-apiserver-arm64

# Build all cluster components for both architectures
make build-cluster-all-arch
```

### Available Cluster Components

- apiserver
- authserver
- dnsserver
- e2e
- genesis
- gwagent
- ingress
- nocturne
- nodeinit
- octovigil
- portal
- rscserver
- vigil

Each component has explicit AMD64 and ARM64 targets.

### Build All Components

```bash
# Build everything for both architectures
make build-all-arch
```

Binaries will be output to the `bin/` directory with architecture suffixes:

```
bin/
├── octelium-amd64
├── octelium-arm64
├── octeliumctl-amd64
├── octeliumctl-arm64
├── octops-amd64
├── octops-arm64
├── octelium-apiserver-amd64
├── octelium-apiserver-arm64
├── ... and so on
```

## Build Flags

All builds use the following compilation flags to ensure portability:

```bash
CGO_ENABLED=0       # Disable C bindings for cross-platform compatibility
GOOS=linux          # Target operating system (linux, darwin, windows)
GOARCH=amd64|arm64  # Target architecture
```

## Release Artifacts

### GitHub Releases

The CI/CD pipeline automatically builds and publishes release artifacts for both architectures:

- `octelium-linux-amd64.tar.gz`
- `octelium-linux-arm64.tar.gz` ← NEW
- `octelium-darwin-amd64.tar.gz`
- `octelium-darwin-arm64.tar.gz` ← NEW (Apple Silicon)
- `octelium-windows-amd64.exe`

### Docker Images

Multi-platform Docker images are built and published to GitHub Container Registry:

```bash
# Pull latest version (auto-detects your architecture)
docker pull ghcr.io/octelium/octelium-apiserver:latest

# Pull specific architecture
docker pull ghcr.io/octelium/octelium-apiserver:v1.0.0-amd64
docker pull ghcr.io/octelium/octelium-apiserver:v1.0.0-arm64
```

All 15 components (12 cluster + 3 client) support both architectures:

**Cluster Components:**
- octelium-apiserver
- octelium-authserver
- octelium-dnsserver
- octelium-e2e
- octelium-genesis
- octelium-gwagent
- octelium-ingress
- octelium-nocturne
- octelium-nodeinit
- octelium-octovigil
- octelium-portal
- octelium-rscserver
- octelium-vigil

**Client Components:**
- octelium
- octeliumctl
- octops

## Development

### Local Build on Different Platforms

**On Apple Silicon (M1/M2/M3+):**
```bash
# Build for your native ARM64 architecture
make build-cli

# Cross-compile for AMD64
make build-cli-octelium-amd64 build-cli-octeliumctl-amd64 build-cli-octops-amd64
```

**On Linux/Windows (AMD64):**
```bash
# Build for your native AMD64 architecture
make build-cli

# Cross-compile for ARM64
make build-cli-octelium-arm64 build-cli-octeliumctl-arm64 build-cli-octops-arm64
```

### Verify Binary Architecture

Check the binary architecture using the `file` command:

```bash
# Should show "x86-64"
file bin/octelium-amd64

# Should show "ARM aarch64"
file bin/octelium-arm64
```

## CI/CD Pipelines

### Release Pipeline

**File:** `.github/workflows/release.yaml`

- Triggers on version tags (`v*.*.*`)
- Builds all components for AMD64 and ARM64 in parallel
- Produces release artifacts for:
  - Linux (AMD64, ARM64)
  - macOS (AMD64, ARM64 - Apple Silicon)
  - Windows (AMD64)

### Container Image Pipelines

**Cluster Components:** `.github/workflows/cluster-components.yaml`
- Uses Docker buildx for multi-platform builds
- Produces images for linux/amd64 and linux/arm64
- Publishes to GitHub Container Registry (GHCR)

**Client Components:** `.github/workflows/client-components.yaml`
- Uses Docker buildx for multi-platform builds
- Produces images for linux/amd64 and linux/arm64
- Publishes to GitHub Container Registry (GHCR)

## Troubleshooting

### Cross-Compilation Issues

If you encounter cross-compilation issues:

1. Ensure Go 1.20+ is installed: `go version`
2. Check that dependencies support your target architecture
3. Build without CGO: `CGO_ENABLED=0` is already set in Makefile targets

### Binary Execution

- **Wrong architecture:** If you get "exec format error", you're trying to run a binary for a different architecture
- **Test execution:** For native testing, use `file bin/octelium-*` to verify architecture

### Docker Build Issues

If Docker multi-platform builds fail:

1. Ensure Docker buildx is available: `docker buildx version`
2. Check that your system has build caching enabled
3. Review CI/CD logs for platform-specific errors

## See Also

- [README.md](README.md) - Project overview
- [Makefile](Makefile) - Build system implementation
- [.github/workflows/](/.github/workflows/) - CI/CD pipeline definitions
