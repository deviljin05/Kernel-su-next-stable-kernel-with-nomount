<div align="center">

# 🔥 BLAZE Kernel Builder

### Custom GKI **android12-5.10** kernel, built entirely in GitHub Actions

**KernelSU-Next · SUSFS · NoMount · Baseband Guard · BBRv3 · Kali NetHunter · Droidspaces**

![Kernel](https://img.shields.io/badge/kernel-android12--5.10%20(GKI%20LTS)-3DDC84?style=for-the-badge&logo=android&logoColor=white)
![Root](https://img.shields.io/badge/root-KernelSU--Next-8A2BE2?style=for-the-badge)
![SUSFS](https://img.shields.io/badge/SUSFS-enabled-orange?style=for-the-badge)
![CI](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Output](https://img.shields.io/badge/output-AnyKernel3-informational?style=for-the-badge)

*One click. One workflow. A flashable kernel, plus Magisk modules for wireless pentesting.*

</div>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Quick Start](#-quick-start)
- [Workflow Inputs](#-workflow-inputs)
- [Build Pipeline](#-build-pipeline)
- [Build Outputs](#-build-outputs)
- [NetHunter Support](#-nethunter-support)
- [Flashing Guide](#-flashing-guide)
- [Customization](#-customization)
- [Troubleshooting](#-troubleshooting)
- [Limitations](#-limitations)
- [Credits](#-credits)
- [Disclaimer](#-disclaimer)

---

## 🧭 Overview

This repository contains a single GitHub Actions workflow (`build.yml`) that compiles a **Generic Kernel Image (GKI)** for **Android 12 / Linux 5.10**. It pulls the official AOSP common kernel, layers on root, hiding, networking and wireless-security features, and packages everything as a flashable **AnyKernel3** zip.

No local toolchain is needed. Everything runs on GitHub's runners and is cached with **ccache** for faster rebuilds.

---

## ✨ Features

| Category | Feature | Notes |
|---|---|---|
| 🔐 **Root** | **KernelSU-Next** | Configurable branch (default `dev-susfs`) |
| 🕶️ **Hiding** | **SUSFS** | Kernel patch from `simonpunk/susfs4ksu`, pinnable to a commit |
| 📂 **Mount** | **NoMount** *(built-in)* | From `maxsteeel/nomount`, toggleable, with post-build verification |
| 🛡️ **Security** | **Baseband Guard (BBG)** | LSM-based protection, toggleable |
| 🚀 **Network** | **BBRv3** TCP backport | Auto-rollback to BBR v1 if a hunk cannot be fixed |
| 🧠 **Memory** | **MGLRU** | Multi-generational LRU |
| ⚡ **Performance** | Safe daily-use optimizations, **LTO=thin**, ccache | |
| 🐧 **Containers** | **Droidspaces** kABI-safe patches + configs | |
| 🔧 **Compat** | **NTSync** patches | |
| 📡 **Pentesting** | **Full Kali NetHunter** kernel support | See [NetHunter Support](#-nethunter-support) |
| 📦 **Packaging** | **AnyKernel3** zip + 2 Magisk modules | Uploaded as workflow artifacts |

---

## 🚀 Quick Start

1. **Fork** this repository (or copy `build.yml` into `.github/workflows/`).
2. Open the **Actions** tab and select **Build Custom GKI Kernel**.
3. Click **Run workflow**, fill in the [inputs](#-workflow-inputs), and start it.
4. When the run finishes, download the artifacts from the run summary page.
5. Flash the zip, following the [Flashing Guide](#-flashing-guide).

> 💡 The first run is the slowest. Later runs reuse the ccache and are noticeably faster.

---

## ⚙️ Workflow Inputs

| Input | Description | Default |
|---|---|---|
| `os_patch_level` | Kernel branch tag. `lts` tracks the rolling LTS branch | `lts` |
| `susfs_commit` | SUSFS commit or branch to use. Pin it for reproducible builds | *(see workflow)* |
| `ksun_branch` | KernelSU-Next branch | `dev-susfs` |
| `device_name` | Prefix used in the output zip name | *(set per device)* |
| `enable_bbg` | Include Baseband Guard | `true` |
| `enable_nomount` | Build NoMount into the kernel | `true` |
| `nomount_branch` | NoMount branch | `dev` |

---

## 🏗️ Build Pipeline

```text
 ┌──────────────────────────────────────────────────────────────┐
 │ 1. Prepare      checkout · free disk · deps · repo · ccache  │
 │ 2. Sync         GKI android12-5.10 source · detect SUBLEVEL  │
 │ 3. Root         SUSFS patches · KernelSU-Next · fixes        │
 │ 4. Features     NoMount · BBG · BBRv3 · MGLRU                │
 │ 5. NetHunter    configs · WiFi injection patches             │
 │ 6. Extras       NTSync · Droidspaces · CFG80211 check        │
 │ 7. Build        branding · LTO=thin · kernel compile         │
 │ 8. Drivers      NoMount verify · RTL8188EU/EUS external .ko  │
 │ 9. Package      AnyKernel3 · Magisk modules · upload         │
 └──────────────────────────────────────────────────────────────┘
```

Each patch step is written to fail loudly or roll back cleanly where a safe fallback exists (for example BBRv3 → BBR v1).

---

## 📦 Build Outputs

| Artifact | Purpose |
|---|---|
| `<device>-KSUNext-android12-5.10-susfs<commit>.zip` | Flashable **AnyKernel3** kernel package |
| `Wireless_LKM.zip` | Magisk module that loads the NetHunter wireless kernel modules at boot |
| `RTL8188EU-EUS-Driver.zip` | Magisk module that loads the external RTL8188EU/EUS driver |

---

## 📡 NetHunter Support

The kernel side of Kali NetHunter is built in, so supported adapters can do **monitor mode** and **packet injection**.

**Included**

- **mac80211 / cfg80211 injection patches** (`tx.c`, `chan.c`, RTL8187, ZD1211RW)
- A broad set of **USB Wi-Fi drivers** as modules (Atheros, MediaTek, Ralink, Realtek and others)
- **USB gadget** support: HID, ACM, RNDIS
- **Bluetooth**, **CAN**, **USB serial**, **SDR** (e.g. HackRF), **NFS**, **WireGuard**, **Netfilter**
- **RTL8188EU/EUS** external driver (aircrack-ng `v5.3.9`) built against the freshly compiled kernel

**Notes**

- Some options in the NetHunter config list do not exist on 5.10 and are silently ignored. This is harmless.
- The **NetHunter app and chroot are not part of this build**. Install them separately.
- Wi-Fi drivers ship as **modules**. The `Wireless_LKM` module loads them at boot.

---

## 📲 Flashing Guide

**Requirements:** unlocked bootloader, a custom recovery or a kernel flasher that supports AnyKernel3, and a **backup of your current boot image**.

1. Download the kernel zip from the workflow artifacts.
2. Flash it through your recovery or kernel flasher.
3. Reboot, then install the **KernelSU-Next manager** app.
4. *(Optional)* Install `Wireless_LKM.zip` and `RTL8188EU-EUS-Driver.zip` as Magisk modules. Reboot again.

> ⚠️ Flash only a build made for your **device's GKI generation and kernel version**. A mismatch can cause a bootloop.

---

## 🛠️ Customization

- **Change the kernel branding:** edit the *Apply custom kernel branding* step (current suffix: `-BLAZE-KSUN-STABLE`).
- **Add or remove kernel options:** edit the config steps. They append `CONFIG_*` lines to the GKI defconfig.
- **Disable a feature:** use the `enable_bbg` / `enable_nomount` inputs, or remove the related step.
- **Reproducible builds:** pin `susfs_commit` and use a fixed branch instead of `lts`.

> Appended config options are dropped silently if the symbol does not exist or a dependency is missing. Check the final `.config` when adding options.

---

## 🩺 Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| Steps pane looks black on GitHub mobile | UI glitch while the run starts. Refresh, use the desktop site, or open the raw logs |
| A patch fails to apply | Upstream changed. Pin `susfs_commit` or the KernelSU-Next branch to a known-good version |
| Config option has no effect | Missing dependency or vendor gate. Inspect the final `.config` |
| Module will not load on device | Version magic or ABI mismatch. Rebuild with the same kernel you flashed |
| Bootloop after flashing | Wrong GKI generation for the device. Restore your boot backup |

---

## ⚠️ Limitations

- Targets **android12-5.10 GKI** only.
- The external **RTL8188EU/EUS driver build** and on-device module loading depend on your adapter and device. Check the first run's logs.
- Upstream projects change often. Builds that worked yesterday can break after an upstream update, which is why pinning is recommended.

---

## 🙏 Credits

- [Android Open Source Project](https://android.googlesource.com/kernel/common): GKI common kernel
- [KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next): root framework
- [simonpunk / susfs4ksu](https://gitlab.com/simonpunk/susfs4ksu): SUSFS
- [maxsteeel / nomount](https://github.com/maxsteeel/nomount): NoMount
- [Kali NetHunter](https://www.kali.org/docs/nethunter/): wireless pentesting platform
- [aircrack-ng / rtl8188eus](https://github.com/aircrack-ng/rtl8188eus): RTL8188EU/EUS driver
- [AnyKernel3](https://github.com/osm0sis/AnyKernel3): flashable packaging
- Everyone behind BBRv3, MGLRU, NTSync, Droidspaces and Baseband Guard

---

## 📜 Disclaimer

This project is provided **as is**, with no warranty. Flashing custom kernels can brick your device, void your warranty, or cause data loss. You are responsible for what you flash.

The wireless and NetHunter features are for **learning and authorized security testing only**. Use them only on networks and devices you own or have explicit permission to test.

---

<div align="center">

**Built with 🔥 by the BLAZE project**

*If this helped you, give the repo a ⭐*

</div>
