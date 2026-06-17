# KOReader for Nook Glowlight 4 Plus

Custom KOReader build for the **Nook Glowlight 4 Plus** (model `bnrv1300`, Android 8.1).

Download the APK from [Releases](../../releases).

---

## What's different from stock KOReader

| Feature | Stock KOReader | This build |
|---------|---------------|------------|
| Frontlight brightness | Not supported | ✅ Full control (0–100%) |
| Frontlight warmth | Not supported | ✅ Full control (0–100%), no root required |
| EPD waveform | Not supported | ✅ GC16 (full refresh) + GU16 (partial) |

### How it works

**Warmth** is set by sending an intent to `com.nook.partner`'s `GlowLightService` — a Barnes & Noble system service that is exported with no permission requirement. This avoids needing root or `DEVICE_POWER`.

**Brightness** is written to `Settings.System.SCREEN_BRIGHTNESS`. This requires granting KOReader the "Modify system settings" special permission once after install (see below).

**EPD waveforms** are applied via the same reflection hook used by `com.nook.partner`'s `EpdDisplayControllerImpl`, targeting `view.invalidate(int)`.

Warmth is automatically restored after resume from background because the hardware node resets on every app re-entry.

---

## Installation

### 1. Sideload the APK

Download `koreader-android-arm64.apk` from the latest [Release](../../releases) and install it with:

```
adb install koreader-android-arm64.apk
```

Or copy it to the device and open it from the Files app.

### 2. Grant "Modify system settings"

Brightness control requires a one-time permission grant that Android does not include in the standard install dialog:

**Option A — Settings UI:**
> Settings → Apps → KOReader → Advanced → Modify system settings → Allow

**Option B — ADB:**
```
adb shell appops set org.koreader.launcher WRITE_SETTINGS allow
```

This permission survives updates but resets on a fresh uninstall + reinstall.

### 3. Enable com.nook.partner (if needed)

Warmth control depends on Barnes & Noble's `GlowLightService`. It is usually enabled, but if warmth has no effect, run:

```
adb shell pm enable com.nook.partner
```

---

## Known limitations

- **WRITE_SETTINGS resets on fresh reinstall.** Re-run the permission grant after uninstalling and reinstalling.
- **AutoWarmth plugin conflict.** KOReader's AutoWarmth plugin overrides manual warmth on resume. Disable it in Plugin Manager if your warmth setting keeps resetting.
- **WiFi programmatic toggle not yet available.** The Settings → Network → WiFi menu opens Android's system Wi-Fi settings instead of toggling in-app. This is planned for the next release.
- **nopowen users: button fix required.** If you use the `nopowen` deep-sleep patch for battery life, page-turn buttons go silent when `mWakefulness=Asleep`. The fix is to add `android.timeout.set(-1)` inside `InterceptReaderWidget` in the patch file so `FLAG_KEEP_SCREEN_ON` prevents Android sleep while nopowen handles AllWinner deep sleep independently. See [`nopowen.md`](https://github.com/backcountrymountains/koreader/blob/nook-gl4plus-clean/nopowen.md) in the working branch.

---

## Upstream PRs

These changes are being contributed back to mainline KOReader:

| Repo | PR | Status |
|------|----|--------|
| [koreader/android-luajit-launcher](https://github.com/koreader/android-luajit-launcher) | [#592 — Add lights and EPD support for Nook Glowlight 4 Plus](https://github.com/koreader/android-luajit-launcher/pull/592) | Open, ready for review |
| [koreader/koreader](https://github.com/koreader/koreader) | [#15561 — android: add Nook Glowlight 4 Plus (bnrv1300) lights support](https://github.com/koreader/koreader/pull/15561) | Draft (waiting on #592) |

Once both PRs merge, this build will be equivalent to a stock KOReader release for the GL4 Plus.

---

## Building from source

Source is on the working branch of the forks:

- **KOReader:** [`backcountrymountains/koreader`](https://github.com/backcountrymountains/koreader) — branch `nook-gl4plus-clean`
- **luajit-launcher:** [`backcountrymountains/android-luajit-launcher`](https://github.com/backcountrymountains/android-luajit-launcher) — branch `nook-gl4plus-personal`

The build uses the standard KOReader Android build system. See the upstream [build documentation](https://github.com/koreader/koreader/blob/master/doc/Building.md) for prerequisites.

---

## Device notes

| Field | Value |
|-------|-------|
| Model | Nook Glowlight 4 Plus |
| Internal ID | `bnrv1300` |
| Android version | 8.1 (Oreo) |
| Architecture | arm64-v8a |
| Frontlight hardware | lm3630a (warm + cool LEDs) |
| EPD controller | AllWinner (Emperor) |
