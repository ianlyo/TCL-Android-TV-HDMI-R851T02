# TCL HDMI Launcher

<p align="left">
  <b>English</b> | <a href="README_zh.md">繁體中文</a>
</p>

[![Latest Release](https://img.shields.io/github/v/release/ian20040409/TCL-Android-TV-HDMI-R851T02?style=flat-square&color=blue)](https://github.com/ian20040409/TCL-Android-TV-HDMI-R851T02/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/ian20040409/TCL-Android-TV-HDMI-R851T02/total?style=flat-square)](https://github.com/ian20040409/TCL-Android-TV-HDMI-R851T02/releases)
[![License](https://img.shields.io/github/license/ian20040409/TCL-Android-TV-HDMI-R851T02?style=flat-square)](LICENSE)

I built this to replace the sluggish, ad-filled stock Android TV launcher on my TCL TV (R851T02 / RTD2851). If you mostly use external devices like an Apple TV, PS5, Switch, or TV box, this turns the TV into a simple display that boots straight to your HDMI input without getting in the way.

**[Download Latest APK](https://github.com/ian20040409/TCL-Android-TV-HDMI-R851T02/releases/latest)**

---

## What it does

- **Auto-boot into HDMI (or an app)**: On boot or when pressing Home, a short countdown switches automatically to your default HDMI port. Press any D-Pad arrow to cancel it. You can also set it to launch a specific app (like YouTube or Netflix) or open the App Drawer instead.
- **Built-in `TvView` HDMI player**: Uses Android's TV Input Framework (`TvView`) directly instead of TCL's buggy `com.tcl.tv`. Passes through 4K @ 60Hz, HDR10, and Dolby Vision cleanly without extra overlays, and includes an optional Signal Search HUD and no-signal auto-sleep timer.
- **Source-aware CEC handling**: Reads `HdmiCecController` logs so the TV only goes to sleep if the standby signal comes from the HDMI port you're actually watching—meaning a docked Switch going to sleep in the background won't shut off your TV while you're on Apple TV. It also blocks `com.tcl.voicestandby` power-off loops from TCL's firmware.
- **Quick switching**: Hit `1`, `2`, or `3` on the remote to jump straight to HDMI 1–3 from the launcher or viewer, or press `INPUT` / `SOURCE` to cycle through them. With Button Mapper enabled, pressing `INPUT` in other apps also brings you back to the launcher instead of opening TCL's stock input picker.
- **TV App Drawer**: Simple card grid with category tabs (All, Recent, TV, Mobile, System, Frozen). If Shizuku is running, you can freeze/unfreeze bloatware directly from the app menu.
- **One-click Shizuku setup**: Includes a built-in downloader for the [Shizuku TV build](https://github.com/thedjchi/Shizuku/releases). Once authorized, it grants `READ_LOGS` and enables the Wake Guard accessibility service automatically—no need to dig through TV system menus.
- **Pure black (`#000000`) UI & Leanback settings**: Designed for dark rooms, with a native TV settings menu, basic button remapping, a shortcut to TCL's picture/sound menu (`com.tcl.settings`), and a live CEC event viewer for debugging.

---

## Screenshots

| Input Selector | Countdown Dialog | Picture Settings Shortcut | App Drawer |
|:---:|:---:|:---:|:---:|
| ![Main](readme_pic/Screenshot_20260925_222244.png) | ![Countdown](readme_pic/Screenshot_20260925_222304.png) | ![Settings](readme_pic/Screenshot_20260925_222322.png) | ![Apps](readme_pic/Screenshot_20260925_222344.png) |

---

## Setup

### 1. Install the APK
Grab the APK from [Releases](https://github.com/ian20040409/TCL-Android-TV-HDMI-R851T02/releases/latest) and sideload it over ADB:
```bash
adb connect <TV-IP>:5555
adb install -r app-release.apk
```

### 2. Set as default launcher
```bash
adb shell cmd package set-home-activity com.lnu.tclhdmilauncher/.MainActivity
```

### 3. Disable the stock launcher (optional, recommended)
Stops the stock Google TV / Android TV launcher from waking up in the background:
```bash
adb shell pm disable-user --user 0 com.google.android.tvlauncher
adb shell am force-stop com.google.android.tvlauncher
```
*(You can re-enable it anytime with `adb shell pm enable com.google.android.tvlauncher`.)*

### 4. Enable Wake Guard & CEC auto-switching

Wake Guard (an Accessibility Service) works around standby black-screen issues and handles Home button behavior. `READ_LOGS` lets the app watch CEC source changes.

- **With Shizuku (easiest)**  
  If you have [Shizuku (thedjchi's TV build)](https://github.com/thedjchi/Shizuku/releases) running—or install it from inside the launcher's settings—just grant Shizuku access in Settings. It will automatically grant `READ_LOGS` and turn on Wake Guard.

- **Manual ADB**  
  1. Grant `READ_LOGS` and restart the app:
     ```bash
     adb shell pm grant com.lnu.tclhdmilauncher android.permission.READ_LOGS
     adb shell am force-stop com.lnu.tclhdmilauncher
     ```
  2. Enable the accessibility service (or tap **Wake Guard** in the app's status bar):
     ```bash
     adb shell settings put secure enabled_accessibility_services com.lnu.tclhdmilauncher/.WakeAccessibilityService
     ```

---

## Remote Controls

| Where | Button | Action |
|---|---|---|
| **Launcher / Viewer** | `1` / `2` / `3` | Jump to HDMI 1 / 2 / 3 |
| **Launcher / Viewer** | `INPUT` / `SOURCE` | Cycle HDMI 1 -> HDMI 2 -> HDMI 3 |
| **Other Apps** | `INPUT` / `SOURCE` | Jump back to HDMI Launcher (requires Button Mapper) |
| **Launcher** | D-Pad Arrows | Move focus; cancels auto-switch if countdown is running |
| **Launcher** | `OK` / `Enter` | Open selected HDMI port or app (long-press HDMI card to set as default) |
| **Launcher** | `SETTINGS` / `MENU` | Open launcher settings |
| **HDMI Viewer** | `SETTINGS` | Open TCL picture / sound settings (`com.tcl.settings`) |
| **HDMI Viewer** | `BACK` / `MENU` | Return to launcher |
| **App Drawer** | `OK` / `Enter` | Launch app |
| **App Drawer** | `MENU` / Long-press `OK` | App options (App info, uninstall, freeze/unfreeze via Shizuku) |

---

## Tested Hardware & Input IDs

Tested on a **TCL 65C715** (Chassis: RTD2851 / R851T02, Firmware: `V8-R851T02-LF1V662`, Android 9):

| Input | Port | TvInput ID |
|---|---|---|
| HDMI 1 | 1 | `com.tcl.tvinput/.passthroughinput.TvPassThroughService/HW1413744128` |
| HDMI 2 | 2 | `com.tcl.tvinput/.passthroughinput.TvPassThroughService/HW1413744384` |
| HDMI 3 | 3 | `com.tcl.tvinput/.passthroughinput.TvPassThroughService/HW1413744640` |

---

## Permissions & Privacy

Everything runs locally on the TV:
- **`READ_LOGS`**: Only filters `HdmiCecController` lines in memory to track active CEC sources. Nothing is saved or sent anywhere.
- **Accessibility Service (`WakeAccessibilityService`)**: Watches window state changes and key events to handle Home button redirection and prevent standby black screens. Does not log input or personal data.
- **Shizuku**: Used strictly for local shell commands (granting `READ_LOGS`, toggling accessibility, freezing apps).

---

## More Docs

If you run into TCL firmware quirks or want to see how the workarounds work under the hood:

- **[Troubleshooting Guide (docs/troubleshooting.md)](docs/troubleshooting.md)** ([繁體中文](docs/troubleshooting_zh.md)) — Covers Apple TV dual-standby loops (`com.tcl.tv`), stuck Dolby Vision banners, `TvView` / Realtek HWC black screens, and how the CEC log listener works.
- **[ADB Cheatsheet (docs/adb-cheatsheet.md)](docs/adb-cheatsheet.md)** — Common ADB commands for this TV.
- **[Firmware & Compatibility Notes (docs/firmware-guide.md)](docs/firmware-guide.md)** — Notes on Android 9 V662 vs. Android 11.
