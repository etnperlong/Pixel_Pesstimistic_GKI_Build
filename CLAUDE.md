# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Android GKI (Generic Kernel Image) build system that creates customized kernels with KernelSU and SUSFS integration using GitHub Actions. The repository builds kernels for Android 12-15 with corresponding kernel versions 5.10-6.6.

For project structure, view @PROJECT_INDEX.md

## Build System Architecture

### Workflow Hierarchy
```
build-kernel-release.yml (Master Orchestrator)
├── kernel-a12-5.10.yml (Android 12, Kernel 5.10)
├── kernel-a13-5.10.yml (Android 13, Kernel 5.10)
├── kernel-a13-5.15.yml (Android 13, Kernel 5.15)
├── kernel-a14-5.15.yml (Android 14, Kernel 5.15)
├── kernel-a14-6.1.yml  (Android 14, Kernel 6.1)
└── kernel-a15-6.6.yml  (Android 15, Kernel 6.6)
    └── All call → gki-kernel.yml (Core Build Engine)
```

### Core Build Process (gki-kernel.yml)
1. **Environment Setup**: ccache, build space optimization, toolchain caching
2. **Source Sync**: Android kernel repo initialization with specific branches
3. **Security Integration**: KernelSU (SukiSU variant) and SUSFS patches
4. **Build Execution**: Either `build/build.sh` (older) or `bazel build` (newer)
5. **Output Generation**: AnyKernel3 packages and kernel images

### Version-Specific Patterns
- Each Android version has its own workflow file with matrix builds
- Matrix defines `sub_level` and `os_patch_level` combinations
- Most matrices are commented out (disabled) except active build targets
- All version workflows use `workflow_call` and inherit secrets

## Key Build Commands

### Manual Workflow Trigger
Access GitHub Actions tab and run "Build and Release GKI Kernels" with these parameters:
- **Build Selection**: Choose Android versions to build (a12_510, a13_510, etc.)
- **KernelSU Variant**: Currently only "SukiSU" supported
- **KernelSU Branch**: Stable, Dev, or Other (custom)
- **Make Release**: Create GitHub release with artifacts

### Build Commands (Internal to CI)
```bash
# Legacy build system (Android 12-13)
LTO=thin BUILD_CONFIG=common/build.config.gki.aarch64 build/build.sh CC="/usr/bin/ccache clang"

# Modern build system (Android 14-15)  
tools/bazel build --disk_cache=/home/runner/.cache/bazel --config=fast --lto=thin //common:kernel_aarch64_dist
```

### Kernel Configuration
Kernel configs are dynamically modified in gki-kernel.yml:
```bash
echo "CONFIG_KSU=y" >> ./common/arch/arm64/configs/gki_defconfig
echo "CONFIG_KSU_SUSFS=y" >> ./common/arch/arm64/configs/gki_defconfig
# ... additional SUSFS and security configs
```

## Security Integration Details

### KernelSU Branch Logic
- **SukiSU + Stable**: Uses `susfs-main` branch
- **SukiSU + Dev**: Uses `susfs-test` branch (all kernel versions)
- **Other**: Uses custom branch specified in `kernelsu_branch_other`

### SUSFS Patch Application
1. Clone `susfs4ksu` with branch `gki-{android_version}-{kernel_version}`
2. Copy kernel patches and filesystem modifications
3. Apply patches in sequence: SUSFS → SukiSU hooks → Hide patches

### Key Repositories
- **KernelSU**: `SukiSU-Ultra/SukiSU-Ultra` (main branch)
- **SUSFS**: `ShirkNeko/susfs4ksu` (version-specific branches)
- **Patches**: `ShirkNeko/SukiSU_patch` (syscall hooks, hiding patches)
- **AnyKernel3**: `MiRinChan/AnyKernel3` (flasher template)

## Modifying Build Matrix

### Adding New Kernel Versions
To enable a kernel version build, uncomment the appropriate matrix entry:
```yaml
# In kernel-a15-6.6.yml
matrix:
  include:
    - sub_level: "98"          # Kernel sub-version
      os_patch_level: "2025-08" # Android security patch level
```

### Creating New Android Version Support
1. Create new workflow file: `kernel-a{version}-{kernel}.yml`
2. Add matrix with sub_level/os_patch_level combinations
3. Add conditional job in `build-kernel-release.yml`
4. Ensure SUSFS patches exist for the new version

## Release Management

### Automated Releases
- Triggered by `make_release: true` parameter
- Release name format: `{currentdate}` (YYYY-MM-DD-HH-MM-SS)
- Artifacts: AnyKernel3 ZIP files for each built version
- Telegram notification to configured chat

### Release Notes Template
Includes:
- KernelSU variant and branch information
- SUSFS version and commit references
- Feature list (BBR, Wireguard, manual hooks)
- Installation instructions and warnings

## Important Security Notes

### Pre-Installation Requirements (from README.md)
1. Install official or 5ec1cff KernelSU first
2. Open KernelSU Manager → Settings
3. **Disable global umount** to prevent SUSFS conflicts
4. This prevents system issues like black screen, WiFi problems, baseband failures

### Installation Methods
- **Recommended**: HorizonKernelFlasher (not Kernel Flasher)
- **Alternative**: SukiSU Ultra manager installation
- **Safety**: Always `fastboot boot` before permanent flashing

## Common Development Tasks

### Enabling/Disabling Kernel Versions
Edit the matrix section in respective `kernel-a{version}-{kernel}.yml` files by commenting/uncommenting entries.

### Updating Security Patches
Modify the `os_patch_level` values in matrix configurations to match latest Android security bulletins.

### Adding Custom Kernel Configurations
Add new `CONFIG_*` entries in the "Add Configuration Settings" step of `gki-kernel.yml`.

### Debugging Build Issues
- Check GitHub Actions logs for specific failure points
- Verify SUSFS branch availability for new kernel versions
- Ensure patch compatibility between kernel versions
- Check toolchain cache hits and compilation errors

## Build Artifacts

### Output Files
- `android{version}-{kernel}.{sublevel}-{patch_level}-AnyKernel3.zip`
- Raw kernel images (Image, Image.lz4, Image.gz) - for debugging
- Boot images (disabled by default)

### Artifact Storage
- GitHub Actions artifacts (temporary)
- GitHub Releases (permanent, public)
- Telegram notifications with download links