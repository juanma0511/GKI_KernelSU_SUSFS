# GKI Kernels — KernelSU + SUSFS

Android GKI kernels built with **official KernelSU** and **SUSFS** root-hiding patches, for every supported GKI branch.

> [!WARNING]
> Flashing a custom kernel **voids your warranty** and may brick your device.  
> Always back up your device before flashing. Flash at your own risk.

---

## Features

| Feature | Description | Source |
|---------|-------------|--------|
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
| `kernels` | Which kernels to build (`all` or a specific version) | `all` |
| `release` | Automatically publish a GitHub Release on success | `false` |

> Security-patch (OS patch level) pinning is **disabled** — each job syncs the
> base GKI manifest branch directly (e.g. `common-android13-5.10`) instead of a
> `-YYYY-MM` patch-level branch.

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
│     ├── Apply 50_add_susfs_in_gki-*.patch → kernel/common         │
│     └── Apply KernelSU/*.patch            → kernel/common/KernelSU │
│        (rejected hunks are skipped, never fatal — see note below)   │
│  5. Append SUSFS/KSU config options to gki_defconfig                │
│     └── strip check_defconfig (both build.sh and Kleaf)            │
│  6. Build                                                           │
│     ├── Android 12–13/5.10–5.15 → LTO=thin build/build.sh          │
│     └── Android 14–15/5.15–6.6  → tools/bazel run …kernel_aarch64  │
│  7. Assemble AnyKernel3 + upload the flashable zip as the artifact  │
└─────────────────────────────────────────────────────────────────────┘
```

> Builds run from a clean checkout each time — there is **no ccache / Bazel
> disk-cache** layer. The artifact-based cache was removed because restoring and
> re-uploading the multi-GB tarball every run was slower than a cold build.

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

| Patch | Applied to |
|-------|-----------|
| `kernel_patches/50_add_susfs_in_gki-<version>.patch` | Kernel source (`kernel/common`) |
| `kernel_patches/KernelSU/*.patch` | KernelSU source (`kernel/common/KernelSU`) |

If a version-specific SUSFS branch is unavailable, the workflow falls back to `gki-android14-5.15`.

> [!NOTE]
> Patches are applied with `--fuzz=3 --forward`, and **rejected hunks do not fail
> the build**. SUSFS tracks a moving GKI base, so when a hunk no longer matches
> the current source it is simply skipped (that one feature is left unpatched)
> and any `*.rej` files are listed in the build log.

---

## Artifacts

Each build job uploads a single **flashable AnyKernel3 zip** as its artifact.
Because GitHub always zips an artifact on download, the kernel `Image` is placed
inside the AnyKernel3 tree and uploaded directly — so the downloaded artifact
*is* the flashable zip (no zip-inside-a-zip):

```
AnyKernel3-<version>-KernelSU-<tag>-SUSFS-<YYYYMMDD>.zip
├── anykernel.sh
├── tools/
├── META-INF/
└── Image          ← the built kernel
```

Flash it directly in a custom recovery (TWRP, etc.). Artifacts are retained for
**14 days**. Enable `release: true` in the workflow dispatch to publish them as a
versioned GitHub Release tagged `rN`.

---

## Credits

| Project | Author |
|---------|--------|
| [KernelSU](https://github.com/tiann/KernelSU) | tiann |
| [SUSFS4KSU](https://gitlab.com/simonpunk/susfs4ksu) | simonpunk |
| [GKI_KernelSU_SUSFS](https://github.com/WildKernels/GKI_KernelSU_SUSFS) | WildKernels (build base reference) |
| [AnyKernel3](https://github.com/osm0sis/AnyKernel3) | osm0sis |
