# TECNO SPARK 40C (KM4k / KM4n) — Magisk Root (Without Data Loss)

**🌐 Language / Dil:** **English 🇬🇧** | [Türkçe](README.md)

> Root guide for a **bootloader-unlocked TECNO SPARK 40C** (model `KM4k` / `KM4n`, MediaTek **Helio G81 / MT6768/MT6769**) using Magisk, **without wiping user data**.

---

## Overview

| | |
|---|---|
| **Device** | TECNO SPARK 40C |
| **Model code** | `KM4n` / `KM4k` (board: `km4k_xk67j`) |
| **SoC** | MediaTek MT6768 / MT6769 (Helio G81 / G85 family) |
| **Android** | 15 (HiOS 15.1) |
| **Method** | Magisk patched `init_boot` (A/B device) |
| **Data loss** | **NONE** |
| **Root** | Magisk **v30.7** (versionCode 30700) |
| **Build** | `KM4n-15.1.2.155(TR001PF001AZ)` |

---

## How it works (short version)

This device uses **A/B (seamless) partitions**. On Android 13+ GKI devices, root is achieved by patching the **`init_boot`** partition (NOT `boot`):

1. Obtain the **stock `init_boot.img`** (extracted from firmware).
2. Magisk app **patches** that image.
3. Flash the patched image with `fastboot flash init_boot ...`.
4. Since the bootloader is already unlocked, **user data is not wiped**.

This is the standard Magisk method; because `init_boot` is patched, `/data` is never touched.

---

## Requirements

- **Bootloader unlocked** (`fastboot getvar unlocked` → `yes`).
  - If not: enable Developer Options → OEM Unlocking → `fastboot flashing unlock` (this wipes data).
- PC with **ADB + Fastboot (platform-tools)**
- **7-Zip** (to extract the firmware archive)
- **Magisk v30.7 APK** → [`tools/Magisk-v30.7.apk`](tools/Magisk-v30.7.apk) (included)
- Stock firmware (to obtain `init_boot.img`) — see below.

---

## Firmware and `init_boot.img`

Board-compatible stock firmware:
`Tecno Spark 40C (KM4k-XK67JABCDEFGH-V-OP-250711V1696)`

> **Note:** The full firmware is ~4.9 GB and is NOT stored in this repo. After downloading, extract **only `init_boot.img`**. The [`stock_init_boot.img`](stock_init_boot.img) in this repo was already extracted from that firmware and can be used directly.

Extract `init_boot.img` from the archive:

```powershell
7z e "firmware.7z" -o out -ir!"*init_boot.img"
```

Verify (SHA-256):

```
stock_init_boot.img = 891197858105D3D57CFB6F2CFC7B658A21B0E842504ACA899D9005AF30EAD0FE
```

> ⚠️ **The patched image (`magisk_patched_init_boot.img`) is NOT stored in this repo.** Reason:
> a Magisk-patched image is **specific to the exact device and build**, and flashing it on a
> different device/build can cause a **bootloop / brick**. Therefore everyone must **patch their
> own image for their own device**. The steps are in [docs/ROOT-GUIDE.md](docs/ROOT-GUIDE.md);
> patching takes about 2 minutes.

---

## Step by step

Full guide: **[docs/ROOT-GUIDE.md](docs/ROOT-GUIDE.md)**

Quick summary:

```powershell
# 0) Device connected and ADB authorized
adb devices

# 1) Push the stock init_boot
adb push stock_init_boot.img /sdcard/Download/stock_init_boot.img

# 2) On phone: Magisk > Install > "Select and Patch a File" > stock_init_boot.img
#    Output: /sdcard/Download/magisk_patched-*.img

# 3) Pull the patched image back
adb pull /sdcard/Download/magisk_patched-XXXXX.img magisk_patched_init_boot.img

# 4) Enter fastboot and flash the patched init_boot
adb reboot bootloader
fastboot devices
fastboot flash init_boot magisk_patched_init_boot.img
fastboot reboot

# 5) Verify
adb shell su -c id     # uid=0(root) ... context=u:r:magisk:s0
```

---

## Hiding root (banking / POS / enterprise apps)

Some apps detect root/unlocked bootloader and refuse to run. The stack used:

| Component | Purpose |
|---|---|
| **Magisk Zygisk** | Zygisk engine (NOT ReZygisk — required for Shamiko) |
| **Shamiko** | Hides root/Zygisk traces (blacklist/denylist mode) |
| **PlayIntegrityFork** | Fixes Play Integrity DEVICE/STRONG |
| **Tricky Store** | Spoofed "locked bootloader" identity + keybox |

Details and pitfalls: **[docs/HIDE-ROOT.md](docs/HIDE-ROOT.md)**

> ⚠️ **Critical:** Shamiko does **NOT** work with **ReZygisk**. Magisk's built-in Zygisk must be ON and ReZygisk must be disabled, otherwise Shamiko reports `[❌ Unsupported environment]`.

---

## Repository contents

```
.
├── README.md
├── README.en.md                     # English version
├── LICENSE
├── .gitignore
├── stock_init_boot.img              # Original image extracted from firmware
├── tools/
│   └── Magisk-v30.7.apk
├── docs/
│   ├── ROOT-GUIDE.md                # Detailed root steps (incl. patching)
│   ├── HIDE-ROOT.md                 # Hiding root (Shamiko/PIF/TrickyStore)
│   ├── TROUBLESHOOTING.md           # Troubleshooting
│   └── UNROOT.md                    # Unroot / return to stock
```

---

## Important warnings

- **OTA updates** will remove root. Before updating, use Magisk → Install → **"Install to inactive slot"**; re-patch after the update.
- While the bootloader stays unlocked, `verifiedbootstate=orange`; strict apps may see this.
- Flashing the wrong partition can brick the device. **Always keep the `stock_init_boot.img` backup.**
- This guide is for educational purposes. You are responsible for what you do to your device.

> ⚠️ **DISCLAIMER:** By applying this guide, you accept all risks including **bricking,
> bootloop, data loss and voided warranty**. The authors/contributors cannot be held liable
> for any damage. Full text: **[docs/DISCLAIMER.md](docs/DISCLAIMER.md)**

---

## Credits / Resources

- [Magisk](https://github.com/topjohnwu/Magisk) — topjohnwu
- [Shamiko](https://github.com/LSPosed/LSPosed.github.io) — LSPosed
- [PlayIntegrityFork](https://github.com/osm0sis/PlayIntegrityFork) — osm0sis
- [Tricky Store](https://github.com/5ec1cff/TrickyStore) — 5ec1cff
- [Hovatek](https://www.hovatek.com) — firmware archive

---

## License

[MIT](LICENSE) — Guide and files are provided "as is", without warranty.
