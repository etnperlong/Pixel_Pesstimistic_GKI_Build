# Android GKI Kernel Build System - Project Index

## Project Overview

This project provides a comprehensive GitHub Actions-based build system for creating customized Android Generic Kernel Image (GKI) kernels with integrated security enhancements.

### Core Features
- 🔧 **Multi-Android Version Support**: Android 12-15 (Kernel 5.10-6.6)
- 🛡️ **Security Integration**: KernelSU + SUSFS (Stealthy Universal SPoofing FileSystem)
- 🚀 **Automated CI/CD**: GitHub Actions workflows with parallel builds
- 📱 **Pixel Device Compatibility**: GKI-compatible Android devices
- 🔄 **Multiple Output Formats**: AnyKernel3 flashable packages

## Project Structure

```
.
├── .github/workflows/          # GitHub Actions workflows
│   ├── build-kernel-release.yml   # Master workflow orchestrator
│   ├── gki-kernel.yml             # Core kernel build workflow
│   ├── kernel-a12-5.10.yml       # Android 12 (5.10) specific
│   ├── kernel-a13-5.10.yml       # Android 13 (5.10) specific
│   ├── kernel-a13-5.15.yml       # Android 13 (5.15) specific
│   ├── kernel-a14-5.15.yml       # Android 14 (5.15) specific
│   ├── kernel-a14-6.1.yml        # Android 14 (6.1) specific
│   └── kernel-a15-6.6.yml        # Android 15 (6.6) specific
└── README.md                   # Chinese documentation
```

## Build Matrix Support

| Android Version | Kernel Version | Status | Features |
|----------------|---------------|--------|----------|
| Android 12     | 5.10          | ✅ Active | KernelSU, SUSFS, BBR |
| Android 13     | 5.10/5.15     | ✅ Active | Enhanced security hooks |
| Android 14     | 5.15/6.1      | ✅ Active | Bazel build system |
| Android 15     | 6.6           | ✅ Active | Latest kernel features |

## Key Components

### 1. Master Orchestration Workflow
📄 **File**: `.github/workflows/build-kernel-release.yml`
- **Purpose**: Central dispatcher for all kernel builds
- **Triggers**: Manual workflow dispatch with build selection
- **Features**: 
  - Parallel build execution
  - Conditional builds per Android version
  - Automated release creation
  - Telegram notifications

### 2. Core Build Engine
📄 **File**: `.github/workflows/gki-kernel.yml`
- **Purpose**: Main kernel compilation workflow
- **Key Processes**:
  - Source synchronization from Android repositories
  - KernelSU integration (SukiSU variant)
  - SUSFS patch application
  - Security hook implementation
  - Kernel configuration optimization
  - Multi-format output generation

### 3. Version-Specific Workflows
📄 **Files**: `kernel-a{12-15}-{5.10-6.6}.yml`
- **Purpose**: Version-specific build matrices
- **Configuration**: Sub-level and OS patch level combinations
- **Build Strategy**: Matrix builds for multiple patch levels

## Security Features

### KernelSU Integration
- **Variant**: SukiSU (Enhanced KernelSU)
- **Branches**: Stable, Dev, Custom
- **Features**: Root management with stealth capabilities

### SUSFS (Stealthy Universal Spoofing FileSystem)
- **Purpose**: Advanced root hiding and filesystem spoofing
- **Components**:
  - Path spoofing (`CONFIG_KSU_SUSFS_SUS_PATH`)
  - Mount hiding (`CONFIG_KSU_SUSFS_SUS_MOUNT`)
  - Kernel statistics spoofing (`CONFIG_KSU_SUSFS_SUS_KSTAT`)
  - Command line spoofing (`CONFIG_KSU_SUSFS_SPOOF_CMDLINE`)

### Security Enhancements
- Manual syscall hooks
- Hide kernel modifications
- Anti-detection mechanisms
- Bypass root detection methods

## Build Process Architecture

### Phase 1: Environment Setup
1. **Resource Optimization**: Maximize build space, configure ccache
2. **Toolchain Management**: Download and cache Android build tools
3. **Dependency Resolution**: Clone required repositories

### Phase 2: Source Preparation
1. **Kernel Source Sync**: Initialize repo with specific Android branch
2. **Patch Application**: Apply SUSFS and security patches
3. **Configuration**: Optimize kernel config for performance and security

### Phase 3: Compilation
1. **Build Execution**: Compile with LTO optimization
2. **Image Processing**: Create multiple kernel image formats
3. **Packaging**: Generate AnyKernel3 flashable packages

### Phase 4: Release Management
1. **Artifact Collection**: Gather build outputs
2. **Release Creation**: Automated GitHub releases
3. **Asset Upload**: Deploy kernel packages
4. **Notification**: Telegram alerts for new builds

## Performance Optimizations

### Build Performance
- **ccache**: Compilation caching (2GB cache)
- **Parallel Jobs**: `nproc --all` concurrent compilation
- **LTO**: Link Time Optimization (thin mode)
- **Disk Cache**: Bazel disk cache for faster rebuilds

### Kernel Optimizations
- **BBR Congestion Control**: TCP performance enhancement
- **Network Features**: Advanced netfilter and ipset support
- **Filesystem**: TMPFS extended attributes and POSIX ACL
- **Performance**: Optimized scheduler and memory management

## Usage Instructions

### Manual Build Trigger
1. Navigate to Actions tab in GitHub repository
2. Select "Build and Release GKI Kernels" workflow
3. Configure build parameters:
   - Select Android versions to build
   - Choose KernelSU variant (SukiSU)
   - Select branch (Stable/Dev/Custom)
   - Enable release creation

### Installation Methods
- **Recommended**: HorizonKernelFlasher
- **Alternative**: SukiSU Ultra manager installation
- **Manual**: Flash AnyKernel3 packages via custom recovery

### Safety Precautions
⚠️ **Important**: Always `fastboot boot` before flashing permanently
- Test kernel compatibility with your device
- Have recovery method ready
- Backup existing boot image

## Output Artifacts

### Kernel Packages
- **AnyKernel3 ZIP**: Flashable packages for custom recovery
- **Boot Images**: Direct boot partition images (disabled by default)
- **Raw Kernels**: Uncompressed and compressed kernel images

### Naming Convention
```
android{version}-{kernel_version}.{sub_level}-{os_patch_level}-AnyKernel3.zip
```
Example: `android14-6.1.43-2024-05-AnyKernel3.zip`

## Dependencies and Requirements

### External Repositories
- **Android Kernel**: Google's official kernel sources
- **KernelSU**: SukiSU Ultra variant
- **SUSFS**: Stealth filesystem patches
- **AnyKernel3**: Universal kernel flasher template

### Build Requirements
- **Platform**: Ubuntu Latest (GitHub Actions)
- **Storage**: ~8GB build space
- **Memory**: 8GB+ recommended
- **Cache**: 2GB ccache storage

## Development and Customization

### Adding New Android Versions
1. Create version-specific workflow file
2. Update build matrix configurations
3. Modify core build workflow for version support
4. Test compatibility with SUSFS patches

### Kernel Configuration Customization
- **Base Config**: `arch/arm64/configs/gki_defconfig`
- **Security Features**: Predefined SUSFS and KernelSU options
- **Performance**: BBR, netfilter, filesystem optimizations

### Patch Management
- **SUSFS Patches**: Version-specific filesystem patches
- **Security Hooks**: Manual syscall hook implementations
- **Performance**: Kernel optimization patches

## Troubleshooting

### Common Issues
- **Build Failures**: Check patch compatibility
- **Boot Issues**: Verify device GKI compatibility
- **KernelSU Problems**: Ensure proper SUSFS configuration

### Debug Information
- **Build Logs**: Available in GitHub Actions
- **Kernel Version**: Check with `uname -a`
- **SUSFS Status**: Verify through KernelSU manager

## Project Maintenance

### Regular Updates
- **Security Patches**: Monthly Android security updates
- **SUSFS Updates**: Regular stealth capability enhancements
- **KernelSU**: Track upstream development

### Version Lifecycle
- **Support Period**: 2+ years for major Android versions
- **EOL Policy**: Deprecated when upstream support ends
- **Migration**: Automated upgrade paths for newer versions

## Contributing

### Development Workflow
1. Fork repository
2. Create feature branch
3. Test build compatibility
4. Submit pull request with detailed changes

### Testing Requirements
- Multi-device compatibility testing
- Security feature validation
- Performance regression testing

---

**Note**: This project is for educational and research purposes. Users assume all risks associated with custom kernel modifications.