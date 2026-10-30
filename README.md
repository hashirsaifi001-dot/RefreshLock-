# Refresh Lock

A minimalist, liquid-glass root app that locks your display refresh rate to **120 Hz** and automatically steps down to **90 Hz** and then **60 Hz** when the phone gets hot.

Built for a rooted Realme 12x on Android 16, but it should work on any rooted Android 11+ device whose ROM honours the standard refresh-rate settings.

> **Status:** source project, not yet compiled or tested on a device. If the build fails, open an issue or paste the error message.

## Features

- One big glass circle: tap to turn the lock **on** or **off**
- Locks to **120 Hz** when on
- Drops to **90 Hz** when warm and **60 Hz** when hot, with hysteresis so it doesn't flicker
- Live **SoC** and **battery** temperature cards
- Circle colour shows the current state: cyan = 120 Hz, amber = 90 Hz, red = 60 Hz, grey = off
- Runs as a foreground service, so it keeps working after you close the app
- Turning it off removes the lock and restores the system default

## Requirements

- Rooted device (Magisk, KernelSU or APatch)
- Android 11 or newer (`minSdk 30`), targets Android 16 (`targetSdk 36`)
- A display that supports the rates you want (the Realme 12x panel supports 120 Hz)

## Build the APK

### Option A: Android Studio

1. Unzip the project and open the folder in Android Studio.
2. Wait for Gradle sync to finish.
3. Choose **Build → Build APK(s)**.
4. Install `app/build/outputs/apk/debug/app-debug.apk`.

### Option B: GitHub (no Android Studio needed)

1. Create a free GitHub repository and upload all files from this folder. Keep the `.github` folder.
2. Open the **Actions** tab, choose **Build APK**, then **Run workflow**.
3. Download the `RefreshLock-apk` artifact. It contains `app-debug.apk`.

> A zip of the source cannot be renamed to `.apk`. It has to be built first.

## Usage

1. Open the app and grant **root** when Magisk or KernelSU asks.
2. Allow notifications (Android requires a notification for the background service).
3. Tap the circle to lock 120 Hz.
4. Tap it again to turn the lock off.

## Temperature thresholds

| Rate   | SoC      | Battery  |
|--------|----------|----------|
| 120 Hz | below 48°C | below 40°C |
| 90 Hz  | 48°C and up | 40°C and up |
| 60 Hz  | 55°C and up | 44°C and up |

The app uses whichever sensor is hotter. It steps back up only after the temperature falls **3°C** below the threshold.

To change the limits, edit `Thermal.kt`:

```kotlin
private const val SOC_90 = 48
private const val SOC_60 = 55
private const val BAT_90 = 40
private const val BAT_60 = 44
private const val HYST = 3
```

## How it works

The app runs these root commands:

```sh
settings put system min_refresh_rate 120.0
settings put system peak_refresh_rate 120.0
```

Setting the minimum and peak to the same value locks the display rate. Turning the lock off runs `settings delete` on both keys.

Temperatures come from `/sys/class/thermal/thermal_zone*` (zones whose type contains `soc` or `cpu`) and `/sys/class/power_supply/battery/temp`.

## Limitations

- This locks the **display refresh rate**. It cannot guarantee a game renders at 120 FPS or avoids frame drops. That depends on the game and on thermal throttling.
- Realme UI / ColorOS may override the standard settings in power-saving mode or per app.
- The lock does not auto-start after a reboot. Open the app and tap the circle again.

## Troubleshooting

**The lock doesn't hold.** Turn off battery saver and any adaptive refresh rate option. If it still resets, your ROM may use a different settings key. Dump all settings, change the refresh rate in Settings → Display, dump again, and compare:

```sh
su -c "settings list system; settings list secure; settings list global" > a.txt
# change refresh rate in Settings, then:
su -c "settings list system; settings list secure; settings list global" > b.txt
diff a.txt b.txt
```

**SoC temperature shows "—".** Your thermal zone names differ. List them and adjust the match in `Root.kt`:

```sh
su -c "cat /sys/class/thermal/thermal_zone*/type"
```

**Check the active rate:**

```sh
su -c "dumpsys display | grep -iE 'refreshrate|fps'"
```

## Project structure

```
app/src/main/java/com/example/refreshlock/
  MainActivity.kt       UI, toggle, polling
  CircleView.kt         glass circle (custom drawn)
  BackgroundView.kt     gradient background
  ThermalService.kt     foreground service, applies the lock
  Thermal.kt            thresholds and step logic
  Root.kt               su commands and temperature reading
.github/workflows/build.yml   CI build for the APK
```

## Disclaimer

**Use this app entirely at your own risk.**

This app requires root access and forces system display settings. Doing so can cause serious problems, including but not limited to:

- Overheating, thermal throttling or unexpected shutdowns
- Faster battery drain and accelerated battery wear
- System instability, crashes, boot loops or a device that no longer works properly
- Permanent hardware damage
- Data loss
- Voided warranty

Rooting your device and modifying system settings may already void your warranty. The temperature limits in this app are rough defaults, not guarantees of safety, and the app has not been tested on every device.

This software is provided "as is", without warranty of any kind. The author and contributors are not responsible for any damage, data loss or other harm to your device, accounts or data. **By installing or using this app, you accept all risk.** Back up your data first, and do not use it if you are not comfortable with these risks.

## License

This project is released under the [MIT License](LICENSE).

```
MIT License

Copyright (c) 2026 [Your Name]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
