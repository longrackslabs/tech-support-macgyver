# Peloton workshop tablet

Session notes: October 4, 2026.

A Peloton tablet with a broken touchscreen now works as a workshop and 3D print farm hub: Home Assistant in a browser, MakerWorld browsing, and Spotify controlling a garage Echo Dot.

## Hardware and operating system

- Peloton PLTN-TTR01 / TTR01, Android 10 (API 29).
- CPU supports arm64-v8a and armeabi-v7a.
- Physical display: 1920 × 1080, landscape; portrait apps render at 1080 × 1920.
- Physical density: 240 dpi. Current override: 200 dpi.
- Power supplied through the middle USB-C port using a Mac USB-C charger.
- Other USB-C port currently holds a Logitech mouse receiver through a USB-A adapter.
- Borrowed Apple Magic Keyboard paired over Bluetooth.
- Control laptop: older MacBook Pro running Linux Mint, with ADB installed and Warp terminal.
- Bootloader remains locked. Android was not replaced; no Google Play components were installed.

## How we gained control

1. Used the USB mouse to enable developer options and USB debugging.
2. Paired the Bluetooth Magic Keyboard to interact with the USB debugging authorization dialog while the laptop occupied the USB port.
3. Approved the laptop's RSA fingerprint, including the always-allow option.
4. Connected the tablet to Wi-Fi.
5. Enabled classic ADB over TCP using the authorized USB connection, then connected from the laptop over Wi-Fi.
6. Disconnected the laptop cable and restored the USB mouse connection.

Android 10 can use this classic ADB-over-TCP workflow. It does not require the newer Android wireless-debugging pairing interface.

## Installed apps and verified results

| App | Version | Result |
| --- | --- | --- |
| Kvaesitso | 1.41.0 | Installed and made the default launcher. |
| Firefox | 157.0 ARM64 | Browsing works; Home Assistant login completed. |
| Chrome | 154.0.8037.126 ARM64, API 29+ | Home Assistant login completed; dashboard rendered better than Firefox. MakerWorld Google SSO worked. |
| Bambu Handy | 4.1.2 | Launches and requests portrait. Login parked: screen offered email/password and Facebook, but no Google SSO. |
| Home Assistant Companion (minimal) | 2026.8.4 | Installed from the official GitHub release; welcome screen verified. Server setup and login pending. |
| Spotify | 9.1.88.2204 | Facebook SSO worked. User successfully controlled playback on the garage Echo Dot from the tablet. |

Chrome account/profile sync is not available in the current configuration, which lacks Google Play Services. Web-based Google SSO did work in MakerWorld. The cause of Handy's missing Google login option was not established.

## Other changes

- Force-stopped `com.peloton.activity` to dismiss the bike-error screen. The app remains installed and enabled and may return.
- Enabled automatic network time, disabled automatic timezone, and set `America/Los_Angeles`.
- Changed display density to 200 dpi; the effect on Home Assistant sizing was limited.
- No global rotation override was made for Handy; the user physically turned the tablet on its side.

## ADB operations

Replace `TABLET_IP` with the tablet's current LAN address. After connecting, `adb devices` should show it as `device`.

```sh
# Initial setup: authorized USB connection required.
adb -d tcpip 5555
adb connect TABLET_IP:5555
adb devices

# Launch installed apps.
adb -s TABLET_IP:5555 shell am start -n com.spotify.music/.MainActivity
adb -s TABLET_IP:5555 shell am start -n bbl.intl.bambulab.com/com.bambulab.bambulab.MainActivity
adb -s TABLET_IP:5555 shell am start -a android.intent.action.VIEW -d 'http://homeassistant.local:8123' -p com.android.chrome

# Dismiss the Peloton bike-error app if it returns.
adb -s TABLET_IP:5555 shell am force-stop com.peloton.activity

# Restore original display density or launcher.
adb -s TABLET_IP:5555 shell wm density reset
adb -s TABLET_IP:5555 shell cmd package set-home-activity com.peloton.launcher/.LauncherActivity

# Restore the workshop launcher.
adb -s TABLET_IP:5555 shell cmd package set-home-activity de.mm20.launcher2.release/de.mm20.launcher2.ui.launcher.LauncherActivity
```

Wi-Fi ADB has not been tested across a reboot. It may require the authorized USB setup again, and the tablet's IP address may change. Keep ADB on a trusted local network; do not expose port 5555 to the internet.

## Input and daily use

The current mouse behaves inconsistently: left-click sometimes works, while double-click, long press, or right-click sometimes produced the desired action. Keyboard arrows and Return helped. The cause remains unresolved; a new device is not a guaranteed fix.

The user reported keyboard app switching was useful after trying the suggested shortcuts. Exact modifier-key behavior should be checked with the dedicated keyboard.

Chosen dedicated input: **ProtoArc XK01 TP**, a foldable Bluetooth keyboard with a built-in touchpad. The manufacturer lists Android support. Checkout showed **$56.99 with free shipping**. Delivery, pairing, and touchpad behavior have not been tested.

## Next checks

- Pair the dedicated keyboard and test typing, clicks, scrolling, and app switching.
- Complete server setup and login in Home Assistant Companion.
- Arrange launcher access to Chrome, Home Assistant, and Spotify.
- Plan and test recovery after a reboot with USB access available.
- Decide whether Handy is worth further investigation; printer access in Handy remains untested.
- Netflix has been discussed but not installed or tested.

## Downloads and provenance

- [Kvaesitso official GitHub release](https://github.com/MM2-0/Kvaesitso/releases/tag/v1.41.0)
- [Firefox official Mozilla release directory](https://ftp.mozilla.org/pub/fenix/releases/157.0/android/)
- Chrome and Bambu Handy: APKMirror downloads made by the user on the laptop.
- [Home Assistant Companion official release](https://github.com/home-assistant/android/releases/tag/2026.8.4): minimal APK SHA-256 `8f58a7df71c61447d3370f5d5956a66e3523625dc4b51c8434bd166d3e731bea`, matched the published release digest.
- [Spotify download page used](https://spotify.en.uptodown.com/android/download)
- [ProtoArc XK01 TP manufacturer specifications](https://www.protoarc.com/products/xk01-tp-foldable-keyboard-with-touchpad)

Spotify APK SHA-256: `1e2e5169f0c2f670a5a3d8e5a54e2a11404e28b46b96025d30ad92f167ed097b`. It matched the download page and passed archive integrity checks before installation. Launch was verified; login and playback were confirmed by the user.

The local `baseline/` directory contains initial package, property, setting, and screenshot captures. **It is not a full system backup.** APKs, screenshots, UI dumps, and local network details remain local and are excluded from Git tracking.

## Peloton updates disabled

On October 4, 2026, background downloads and active device-management/OTA services were found. Five update packages were disabled for Android user 0 using `pm disable-user --user 0`. Their state was verified as `disabled-user`, and no active services belonging to them remained. This setting persists across ordinary reboots, but reboot persistence has not been tested; factory reset or another privileged component could change it.

Disabled packages:

- `com.peloton.updater`
- `com.onepeloton.dm.android`
- `com.onepeloton.OTAService`
- `com.onepeloton.bgupdater`
- `com.onepeloton.fwupdateservice`

No packages were uninstalled and downloaded update APKs were retained. About 284 MB remained free on the data partition at inspection. The download directory contained about 656 MB of Peloton app/component APKs, including WebView and Netflix. This did not establish that a full OS update was downloaded.

To restore a package to its original default enabled state:

```sh
adb -s TABLET_IP:5555 shell pm default-state --user 0 PACKAGE_NAME
```

Google browser logins worked for YouTube, MakerWorld, Drive, and Amazon. Chrome profile login failed with a brief message; logs reported missing Google Play services and Play Store. Google account/services installation remains untested.

## Downloaded installer cleanup

The 14 downloaded Peloton update APKs were copied to the control laptop and verified against tablet SHA-256 checksums. Each tablet file was checked again before deleting only those downloaded copies. Installed packages and app login data were retained. Free data-partition space afterward: 889 MB (79% used). The installer backup and checksum manifest remain local, outside this repository.

## Main Peloton app disabled

During an intermittent YouTube click failure, Android input state showed a full-screen touchable `com.peloton.activity` overlay above Chrome. Chrome held keyboard focus, and no sustained input queue backlog was observed. Force-stopping the app removed its overlay; this is a suspect rather than proof of the click failure's cause. Raw mouse capture included paired left/right button press and release events, but lacked a controlled click sequence to assess duplicates.

The main app was then disabled with `pm disable-user --user 0 com.peloton.activity`. Chrome remained foreground, Kvaesitso remained the default launcher, and the Peloton overlay was absent. Mouse behavior still needs user confirmation. Other Peloton hardware packages remain enabled pending individual assessment.

Restore with `adb -s TABLET_IP:5555 shell pm default-state --user 0 com.peloton.activity`.

## Clock screensaver

Installed Clock Screensaver & Widget 2.2 (`systems.sieber.fsclock`) from F-Droid. Selected `systems.sieber.fsclock/.FullscreenDream` as the system screensaver, enabled it, enabled activation while charging, and disabled dock-only activation. Android Settings confirmed “Fullscreen Clock” and “While charging”. Existing screen-off timeout remains 1,200,000 ms (20 minutes). The dream service started during preview; pointer input dismissed an initial preview. Full-screen clock rendering was then verified visually. Automatic activation after 20 minutes of inactivity remains to be observed; video playback may keep the screen awake.

Original screensaver settings are saved locally in `screensaver-before.txt`. To disable: `adb -s TABLET_IP:5555 shell settings put secure screensaver_enabled 0`. To open controls: `adb -s TABLET_IP:5555 shell am start -a android.settings.DREAM_SETTINGS`.

## Home Assistant ADB control

Added the built-in Android Debug Bridge integration to the Home Assistant server using the tablet's LAN IP, port 5555, Android TV device class, and the built-in Python ADB implementation. Approved HA's RSA key with always-allow. Early attempts failed; a retry with the laptop's ADB session disconnected created a loaded integration. This does not establish which setting or connection change resolved the failures.

HA exposes media-player and remote entities for the tablet. The media player reported idle and media volume 40%. A wake command sent through HA's `androidtv.adb_command` action completed successfully (HTTP 200). Automatic screen-off/wake behavior remains to be tested. Keep the laptop ADB session disconnected when testing HA control; concurrent connections require further investigation. Wi-Fi ADB reboot persistence is still untested.

The existing shop-light automations were inspected: light-on uses workbench/print-farm PIR events; light-off combines garage motion and garage presence checks. A proposed tablet automation would wake on shop activity and turn the display off only after the relevant sensors show no occupancy continuously for ten minutes. No new occupancy automation has been installed yet.

## Occupancy screen automation enabled

Created and enabled “Workshop Tablet - Occupancy Screen Control” without changing the shop-light automations. It uses Workbench PIR, Garage Motion, and Garage Presence. The missing print-farm PIR referenced by an older light automation is excluded.

Any sensor on wakes the display. All three sensors must be explicitly off continuously for ten minutes to sleep the display; unknown/unavailable is not treated as empty. On HA startup, current occupancy wakes the tablet. The automation preserves the current app and uses `input keyevent 224` (wake) and `input keyevent 223` (sleep) through HA's ADB action. The clock screensaver remains configured independently.

Verified: automation saved, loaded, and enabled; ADB echo returned the expected text; manually triggering with presence on succeeded; a brief sleep/wake test reported Android `mWakefulness=Dozing` then `mWakefulness=Awake`. The automatic ten-minute empty-room cycle remains to be observed. Template-trigger waiting periods restart when HA or automations reload.

See [occupancy-automation.yaml](occupancy-automation.yaml) for a reusable copy; replace its media-player entity placeholder with your tablet's actual entity.
