# KOReader for Nook Glowlight 4 Plus

Custom KOReader build for the **Nook Glowlight 4 Plus** (model `bnrv1300`, Android 8.1).

Download the APK from [Releases](../../releases).

**No root required.** All features work without root on a stock or Magisk-patched device.

---

## What's different from stock KOReader

| Feature | Stock KOReader | This build |
|---------|---------------|------------|
| Frontlight brightness | Not supported | ✅ Full control (0–100%) |
| Frontlight warmth | Not supported | ✅ Full control (0–100%), no root required |
| EPD waveform | Not supported | ✅ GC16 (full refresh) + GU16 (partial) |
| WiFi toggle | Opens system settings | ✅ Toggle in-app from Settings → Network |
| Network info | Not shown | ✅ SSID, IP address, and MAC address |

### How it works

**Warmth** is set by sending an intent to `com.nook.partner`'s `GlowLightService` — a Barnes & Noble system service that is exported with no permission requirement. This avoids needing root or `DEVICE_POWER`. Warmth is automatically restored on resume because the hardware node resets on every app re-entry.

**Brightness** is written to `Settings.System.SCREEN_BRIGHTNESS`. This requires granting KOReader the "Modify system settings" permission once after install (see below).

**EPD waveforms** are applied via the same reflection hook used by `com.nook.partner`'s `EpdDisplayControllerImpl`, targeting `view.invalidate(int)`.

**WiFi toggle** calls `WifiManager.setWifiEnabled()`, which works natively on Android 8.1. No root is needed on the GL4 Plus.

**Network info** reads SSID and IP from `WifiManager.getConnectionInfo()` and MAC from `NetworkInterface`.

### Android permissions

| Permission | Why it's needed |
|------------|----------------|
| `WRITE_SETTINGS` | Write `Settings.System.SCREEN_BRIGHTNESS` for brightness control |
| `ACCESS_WIFI_STATE` | Read WiFi connection state and network details |
| `CHANGE_WIFI_STATE` | Toggle WiFi on/off without leaving KOReader |

No `INTERNET` permissions beyond what stock KOReader already requests. No `READ_PHONE_STATE`, `LOCATION`, or other sensitive permissions are added by this build.

---

## Installation

### 1. Sideload the APK

Download `koreader-gl4.apk` from the latest [Release](../../releases) and install it:

```
adb install koreader-gl4.apk
```

Or copy it to the device and open it from the Files app.

### 2. Grant "Modify system settings"

Brightness control requires a one-time permission grant not included in the standard install dialog:

**Option A — Settings UI:**
> Settings → Apps → KOReader → Advanced → Modify system settings → Allow

**Option B — ADB:**
```
adb shell appops set org.koreader.launcher WRITE_SETTINGS allow
```

This permission survives updates but resets on a fresh uninstall + reinstall.

### 3. Enable com.nook.partner (if needed)

Warmth control depends on Barnes & Noble's `GlowLightService`. It is usually enabled. If warmth has no effect, try enabling just the frontlight service first:

```
adb shell pm enable com.nook.partner/com.nook.partner.service.GlowLightService
```

If that doesn't work, enable the whole package:

```
adb shell pm enable com.nook.partner
```

**Note:** enabling the full package also re-enables B&N's background services — OTA update checks, status bar overlay, and system receivers. If you previously disabled the package, you can re-disable individual components afterward. See [nook-gl4plus-research](https://github.com/backcountrymountains/nook-gl4plus-research) for the full component inventory.

---

## Known limitations

- **WRITE_SETTINGS resets on fresh reinstall.** Re-run the permission grant after a clean uninstall + reinstall.
- **AutoWarmth plugin conflict.** KOReader's AutoWarmth plugin overrides manual warmth on resume. Disable it in Plugin Manager if your warmth setting keeps resetting.
- **nopowen users: button fix required.** If you use the `nopowen` deep-sleep patch, page-turn buttons go silent when `mWakefulness=Asleep`. Add `android.timeout.set(-1)` inside `InterceptReaderWidget` in the patch file so `FLAG_KEEP_SCREEN_ON` prevents Android sleep. See [`nopowen.md`](https://github.com/backcountrymountains/koreader/blob/nook-gl4plus-pr1-clean/nopowen.md).

---

## Build transparency

The APK in each release is built by [GitHub Actions](.github/workflows/build.yml) directly from the source code in this repository's linked fork — not from a developer's local machine. Every release links to the exact workflow run and source commit so you can verify what was compiled.

The source and build branches are:

| Repo | Branch | What it is |
|------|--------|------------|
| [`backcountrymountains/koreader`](https://github.com/backcountrymountains/koreader) | [`master`](https://github.com/backcountrymountains/koreader/tree/master) | Combined build — all features |
| [`backcountrymountains/koreader`](https://github.com/backcountrymountains/koreader) | [`nook-gl4plus-pr1-clean`](https://github.com/backcountrymountains/koreader/tree/nook-gl4plus-pr1-clean) | PR1: lights only (upstream-ready) |
| [`backcountrymountains/koreader`](https://github.com/backcountrymountains/koreader) | [`nook-gl4plus-pr2-clean`](https://github.com/backcountrymountains/koreader/tree/nook-gl4plus-pr2-clean) | PR2: WiFi toggle + network info (upstream-ready) |
| [`backcountrymountains/android-luajit-launcher`](https://github.com/backcountrymountains/android-luajit-launcher) | [`nook-gl4plus-pr1-clean`](https://github.com/backcountrymountains/android-luajit-launcher/tree/nook-gl4plus-pr1-clean) | Lights + EPD controller |
| [`backcountrymountains/android-luajit-launcher`](https://github.com/backcountrymountains/android-luajit-launcher) | [`nook-gl4plus-pr2-clean`](https://github.com/backcountrymountains/android-luajit-launcher/tree/nook-gl4plus-pr2-clean) | WiFi toggle + network details |

To build the APK yourself, see [Building from source](#building-from-source) below.

---

## Upstream PRs

These changes are being contributed back to mainline KOReader:

| Repo | PR | Status |
|------|----|--------|
| [koreader/android-luajit-launcher](https://github.com/koreader/android-luajit-launcher) | [#592 — Add lights and EPD support for Nook Glowlight 4 Plus](https://github.com/koreader/android-luajit-launcher/pull/592) | Open, ready for review |
| [koreader/koreader](https://github.com/koreader/koreader) | [#15561 — android: add Nook Glowlight 4 Plus (bnrv1300) lights support](https://github.com/koreader/koreader/pull/15561) | Draft (waiting on #592) |
| [koreader/android-luajit-launcher](https://github.com/koreader/android-luajit-launcher) | PR2 — WiFi toggle + network details | Not yet submitted |
| [koreader/koreader](https://github.com/koreader/koreader) | PR2 — android: WiFi toggle + network info | Not yet submitted |

Once all PRs merge, this build will be equivalent to a stock KOReader release for the GL4 Plus.

---

## Building from source

The APK is built using the standard KOReader Android toolchain in the [koreader/koandroid](https://hub.docker.com/r/koreader/koandroid) Docker image. To build locally:

```bash
# 1. Clone the fork with submodules
git clone --recurse-submodules https://github.com/backcountrymountains/koreader.git
cd koreader
git checkout master

# 2. Pull the build environment (requires Docker)
docker pull koreader/koandroid:0.9.1-22.04

# 3. Build
docker run --rm -v "$PWD":/src -w /src \
  -e BASH_ENV=/home/ko/.bashrc \
  koreader/koandroid:0.9.1-22.04 \
  bash -c 'make TARGET=android ANDROID_ARCH=arm64 OUTPUT_DIR=build INSTALL_DIR=install update'

# 4. The APK will be at koreader-android-arm64-*.apk
```

The APK is signed with the Android debug keystore. Because debug keystores are machine-specific, a locally-built APK will have a different signature than the release APK and will require an uninstall before installing over the existing app.

For full build prerequisites, see the upstream [Building.md](https://github.com/koreader/koreader/blob/master/doc/Building.md).

---

## Device notes

| Field | Value |
|-------|-------|
| Model | Nook Glowlight 4 Plus |
| Internal ID | `bnrv1300` |
| Android version | 8.1 (Oreo) |
| SoC | Allwinner B300 (Emperor), ARM Cortex-A53 |
| Architecture | arm64-v8a |
| Frontlight hardware | lm3630a (warm + cool LEDs) |
| EPD controller | AllWinner (Emperor) |
