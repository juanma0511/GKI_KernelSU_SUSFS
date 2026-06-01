# GKI Kernels — KernelSU + SUSFS

Android GKI kernels built with **official KernelSU** and **SUSFS** root-hiding patches, for every supported GKI branch.

> [!WARNING]
> Flashing a custom kernel **voids your warranty** and may brick your device.  
> Always back up your device before flashing. Flash at your own risk.

---

## Features

| Feature | Source |
|---------|--------|
| **KernelSU** (official) | Root solution operating entirely in kernel space | [`tiann/KernelSU`](https://github.com/tiann/KernelSU) |
| **SUSFS** | Kernel-level root-hiding patches + KernelSU hooks | [`simonpunk/susfs4ksu`](https://gitlab.com/simonpunk/susfs4ksu) |

> This project uses [WildKernels/GKI_KernelSU_SUSFS](https://github.com/WildKernels/GKI_KernelSU_SUSFS) as the build base and reference.  
> **This repo uses the official KernelSU**

---

## Supported Kernel Versions

| Version | Android | Build system | SUSFS branch |
|---------|---------|--------------|--------------|
| `5.10` | Android 12 | `build/build.sh` (legacy) | `gki-android12-5.10` |
| `5.10` | Android 13 | `build/build.sh` (legacy) | `gki-android13-5.10` |
| `5.15` | Android 13 | `build/build.sh` (legacy) | `gki-android13-5.15` |
| `5.15` | Android 14 | Bazel (Kleaf) | `gki-android14-5.15` |
| `6.1`  | Android 14 | Bazel (Kleaf) | `gki-android14-6.1` |
| `6.6`  | Android 15 | Bazel (Kleaf) | `gki-android15-6.6` |

---

## How to Build

### Trigger via GitHub Actions (recommended)

1. Fork or push this repository to your GitHub account.
2. Go to **Actions → Build GKI Kernels — KernelSU + SUSFS**.
3. Click **Run workflow** and fill in the inputs:

| Input | Description | Default |
|-------|-------------|---------|
| `kernelsu_tag` | KernelSU branch or tag to use (e.g. `main`, `v1.0.5`) | `main` |
| `os_patch_level` | Android security patch level (`YYYY-MM`) | `2025-05` |
| `kernels` | Which kernels to build (`all` or a specific version) | `all` |
| `release` | Automatically publish a GitHub Release on success | `false` |

---

## Build Pipeline Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  For each kernel version (matrix job)                               │
│                                                                     │
│  1. Free disk space  (~30 GB freed from runner)                     │
│  2. Sync kernel sources  (repo tool → android.googlesource.com)     │
│  3. Set up KernelSU  (official setup.sh → tiann/KernelSU)          │
│  4. Clone susfs4ksu  (version-matched branch)                       │
│     ├── Apply kernel_patches/*.patch  → kernel/common              │
│     └── Apply ksu_patches/*.patch     → kernel/common/KernelSU     │
│  5. Write SUSFS config options to defconfig / Kleaf fragment        │
│  6. Build                                                           │
│     ├── Android 12–13/5.10–5.15 → LTO=thin build/build.sh          │
│     └── Android 14–15/5.15–6.6  → tools/bazel run …kernel_aarch64  │
│  7. Upload Image / Image.lz4 / Image.gz / *.ko as artifacts        │
└─────────────────────────────────────────────────────────────────────┘
```

### SUSFS config options enabled

```kconfig
CONFIG_KSU=y
CONFIG_KSU_SUSFS=y
CONFIG_KSU_SUSFS_SUS_PATH=y
CONFIG_KSU_SUSFS_SUS_MOUNT=y
CONFIG_KSU_SUSFS_SUS_KSTAT=y
CONFIG_KSU_SUSFS_SUS_OVERLAYFS=y
CONFIG_KSU_SUSFS_TRY_UMOUNT=y
CONFIG_KSU_SUSFS_SPOOF_UNAME=y
CONFIG_KSU_SUSFS_ENABLE_LOG=y
CONFIG_KSU_SUSFS_OPEN_REDIRECT=y
CONFIG_KSU_SUSFS_SUS_SU=y
```

---

## SUSFS Patch Sources

Patches are pulled from the matching branch of [`simonpunk/susfs4ksu`](https://gitlab.com/simonpunk/susfs4ksu):

| Branch | Applied to |
|--------|-----------|
| `kernel_patches/*.patch` | Kernel source (`kernel/common`) |
| `ksu_patches/*.patch` | KernelSU source (`kernel/common/KernelSU`) |

If a version-specific branch is unavailable, the workflow falls back to `gki-android14-5.15`.

---

## Artifacts

Each build job uploads a named artifact:

```
kernel-<version>-KernelSU-<tag>-SUSFS-<patch-level>
├── Image
├── Image.lz4   (if produced)
├── Image.gz    (if produced)
├── System.map  (if produced)
└── *.ko        (if produced)
```

Artifacts are retained for **14 days**. Enable `release: true` in the workflow dispatch to publish them as a versioned GitHub Release tagged `rN`.

---

## Credits

| Project | Author |
|---------|--------|
| [KernelSU](https://github.com/tiann/KernelSU) | tiann |
| [SUSFS4KSU](https://gitlab.com/simonpunk/susfs4ksu) | simonpunk |
| [GKI_KernelSU_SUSFS](https://github.com/WildKernels/GKI_KernelSU_SUSFS) | WildKernels (build base reference) |
| [AnyKernel3](https://github.com/osm0sis/AnyKernel3) | osm0sis |
