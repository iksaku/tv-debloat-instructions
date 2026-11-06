# TCL UnionTV / RT2841 platform

Reversible cleanup reference for one Android 9 TV: 29 package disables, screensaver off, AT4K Home, and Wholphin installed. **UnionTV / RT2841 identifies the reported platform, not a confirmed retail model.** Results apply to the recorded firmware; they do not establish compatibility with other TCL TVs.

## Device identity

| Property | Recorded value |
| --- | --- |
| Manufacturer / model / device | TCL / UnionTV / UnionTV |
| Board / hardware version | RTD2841 / RT2841 |
| Hardware | `rtd284x` |
| Android / API | 9 / 28 |
| Software | `V8-R41KT01-LF1V391` |
| Security patch | 2020-12-01 |
| ABI | 32-bit ARM: `armeabi-v7a`, `armeabi`; no 64-bit ABI advertised |
| Build fingerprint | `TCL/UnionTV/UnionTV:9/PPR2.180905.006.A1/builder.20210125.083312:user/release-keys` |
| Android user | 0 |

## Connection and original state

Use authorized local-network ADB on port 5555. Approve the computer on the TV, use a trusted network, and never expose ADB to the Internet. Replace `<TV_IP>` with the intended TV's address and confirm the physical target.

```sh
adb connect '<TV_IP>:5555'
adb -s '<TV_IP>:5555' shell getprop ro.product.model
adb -s '<TV_IP>:5555' shell getprop ro.software.version_id
adb -s '<TV_IP>:5555' shell getprop ro.build.fingerprint
adb -s '<TV_IP>:5555' shell am get-current-user
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief --user 0 -a android.intent.action.MAIN -c android.intent.category.HOME
```

No packages were disabled at the baseline. All 29 packages below changed from manifest/default state (`enabled=0`) to disabled-user (`enabled=3`) for user 0. State `1` means explicitly enabled; it is not interchangeable with default state. Preserve any different original states or component overrides on another device.

## Installed apps and Home

Both apps were absent before installation and were installed for user 0 from their official GitHub releases. Versions below were the latest stable releases checked on 2026-11-05, not a promise of future currency.

| App | Installed version | Package / activity | Compatibility |
| --- | --- | --- | --- |
| [AT4K Launcher v1.3](https://github.com/avadhesh18/at4k/releases/tag/v1.3) | `1.3`, code 20 | `com.overdevs.at4k/.MainActivity` | APK minSdk 27; includes 32-bit ARM |
| [Wholphin v1.1.0](https://github.com/damontecres/Wholphin/releases/tag/v1.1.0) | `1.1.0-0-g2bb662fd`, code 61 | `com.github.damontecres.wholphin/.MainActivity` | APK minSdk 23; `armeabi-v7a` build |

Wholphin's release documents Jellyfin Server 10.10.x and 10.11.x compatibility. A Jellyfin server must be added in Wholphin; no server/account configuration or playback validation is part of this record.

Downloaded APK SHA-256 values matched the official GitHub release asset digests:

- AT4K: `1bb6dad64f26e8d82d65f5a02f76327affb24b825f4d8d3f99c4d56d6fab77a3`
- Wholphin: `1096e30eaacf4add580e8991fb060fd95b3dce63a9fb0f5fd0cbb42b0419639a`

The working installation method transferred each APK before calling the package manager. These examples assume the verified files are downloaded into the current local directory:

```sh
adb -s '<TV_IP>:5555' push AT4K-v1.3.apk /data/local/tmp/AT4K-v1.3.apk
adb -s '<TV_IP>:5555' shell pm install --user 0 /data/local/tmp/AT4K-v1.3.apk
adb -s '<TV_IP>:5555' push Wholphin-v1.1.0-armeabi-v7a.apk /data/local/tmp/Wholphin-v1.1.0.apk
adb -s '<TV_IP>:5555' shell pm install --user 0 /data/local/tmp/Wholphin-v1.1.0.apk
adb -s '<TV_IP>:5555' shell rm /data/local/tmp/AT4K-v1.3.apk /data/local/tmp/Wholphin-v1.1.0.apk
```

For an existing installation, use `pm install -r --user 0` to retain app data, after reviewing signature/version compatibility. Do not uninstall or clear data to work around an update failure.

Open AT4K, respond to its permissions as appropriate, and confirm the actual app grid is usable before disabling stock Home. Home selection alone initially returned to stock Home on this firmware; package-level stock Home disable was needed for the recorded cutover.

```sh
adb -s '<TV_IP>:5555' shell am start --user 0 -W -n com.overdevs.at4k/.MainActivity
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 com.overdevs.at4k/.MainActivity
# Only after verifying AT4K is usable:
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.google.android.tvlauncher
adb -s '<TV_IP>:5555' shell input keyevent KEYCODE_HOME
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief --user 0 -a android.intent.action.MAIN -c android.intent.category.HOME
```

Resulting Home is `com.overdevs.at4k/.MainActivity`. No accessibility service was enabled by this cleanup; the recorded value remained `null`. Launcher hide/reorder preferences belong to the user and are not part of the package-disable procedure.

## Disabled packages and tradeoffs

Every package in this table was present and changed **0 → 3** for user 0. Disabling retains installed packages and their data. These are feature removals, not proof that every component is bloat or that all associated processes stop.

| Feature removed or affected | Exact packages |
| --- | --- |
| Stock Home and Google TV recommendations | `com.google.android.tvlauncher`, `com.google.android.tvrecommendations` |
| TCL Channel/content recommendations | `com.tcl.waterfall.overseas` |
| TCL Home passive integration | `com.tcl.tv.tclhome_passive` |
| TCL analytics | `com.tcl.bi` |
| Partner customization and retail sticker | `com.tcl.partnercustomizer`, `com.tcl.esticker` |
| Google Cast receiver | `com.google.android.apps.mediashell` |
| Google search / voice assistant | `com.google.android.katniss` |
| Play Games, feedback, printing | `com.google.android.play.games`, `com.google.android.feedback`, `com.android.printspooler` |
| Calendar synchronization | `com.google.android.syncadapters.calendar` |
| TalkBack and text-to-speech | `com.google.android.marvin.talkback`, `com.google.android.tts` |
| Screensaver providers | `com.google.android.backdrop`, `com.android.dreams.basic` |
| TCL media browser | `com.tcl.ui_mediaCenter` |
| Google TV phone-remote integration | `com.google.android.tv.remote.service` |
| Retail-demo USB video-copy support | `com.tcl.copydatatotv` |
| Play auto-install configuration stub; future provisioning effects untested | `android.autoinstalls.config.tcl.device` |
| TCL Home dashboard, including IoT/device pages, gifts and messages | `com.tcl.dashboard` |
| TCL app delivery/install/uninstall backend | `com.tcl.appmarket2` |
| TCL account/sign-in integration; service/provider consumers unverified | `com.tcl.usercenter` |
| Standalone video, audio and picture players; vendor file/URL playback, slideshow and music-alone features | `com.tcl.videoplayer`, `com.tcl.audioplayer`, `com.tcl.imageplayer` |
| Additional launcher data, network guide and boot notifications | `com.tcl.partnercustomizer.resource` |
| Cleaner/maintenance UI, storage-low handling and bundled install/uninstall interfaces; other consumers unverified | `com.tcl.tvweishi` |

The TCL app backend exposes privileged install/uninstall services and the `com.tcl.appstore` provider. Disabling it removes TCL-managed app delivery, not just promotions. The retained `com.tcl.versionUpdateApp` has its own app-store/install services and references the same installer permission and install broadcast. This is evidence of shared integration, not proof that disabling the backend cannot affect future updates. Firmware updating and TCL-specific delivery after the change remain unverified; restore the backend before diagnosing a failure in those paths.

The Android package installer, ADB package manager and Google Play remain enabled. ADB reinstallation succeeded after the TCL backend disable; that does not establish that every installation or OTA path works.

## Settings and rollback

`secure screensaver_enabled` changed **1 → 0**, before disabling its providers. The selected provider remains `com.google.android.backdrop/.Backdrop`. No Netflix AppOp restriction was applied.

```sh
adb -s '<TV_IP>:5555' shell settings --user 0 get secure screensaver_enabled
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 0
```

To reproduce one reviewed package change, record its own User 0 state first. Substitute an exact package from the table; do not copy this procedure to unknown packages:

```sh
adb -s '<TV_IP>:5555' shell dumpsys package '<REVIEWED_PACKAGE>'
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 '<REVIEWED_PACKAGE>'
# Rollback for any package in this unit's table: original state was 0.
adb -s '<TV_IP>:5555' shell pm default-state --user 0 '<REVIEWED_PACKAGE>'
```

Restore only selected task changes, never indiscriminately enable every disabled package. For example, restore TCL app delivery with:

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.tcl.appmarket2
```

Restore stock Home before changing the Home selection back:

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.tvlauncher
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 com.google.android.tvlauncher/.MainActivity
adb -s '<TV_IP>:5555' shell am start --user 0 -W -n com.google.android.tvlauncher/.MainActivity
```

Restore Backdrop before re-enabling the screensaver:

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.backdrop
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 1
```

The newly installed apps can remain installed when reverting Home; uninstalling them is not required and would discard their app data. Do not clear AT4K's data to revert Home or undo package disables.

## Preserved services and apps

- **MagisTV (`com.android.mgstv`) was explicitly preserved**, installed in default state (`0`) before and after; no uninstall, disable or app-data clear was performed. Other existing streaming apps were preserved too.
- TV inputs/live TV, settings/configuration, firmware updating, audio, DRM, networking, Bluetooth, pairing, keyboard and physical-remote routing remain outside the disable set. Examples: `com.tcl.tvinput`, `com.tcl.tv`, `com.tcl.settings`, `com.android.tv.settings`, `com.tcl.providers.config`, `com.tcl.versionUpdateApp`, `com.android.bluetooth`, `com.tcl.tcl_bt_rcu_service`, `com.tcl.autopair`, `com.tcl.rc.ota`, `com.tcl.keycustomfunctionservice`.
- `com.tcl.tvweishi` is disabled by explicit choice despite its mixed maintenance responsibilities. It is not assumed to be an alias of another firmware's Guard package. Restore its default state using the rollback pattern above if maintenance/storage or package-management behavior fails.
- Device activation/authentication, setup, terminal management and framework customization remain unchanged. Unresolved responsibilities are not treated as proof of safety. User Center is disabled by explicit choice, including its service/providers; effects on their consumers remain unverified.
- The selected keyboard remains `com.tcl.inputmethod.international/.T_IME`.

No root, package uninstall, app-data clear, factory reset, firmware flash or reboot was performed.

## Verification limits

All 29 target packages were confirmed disabled-user, with no additional disables outside the approved set. After the six additional disables, Wholphin still displayed server selection and the Home resolver still selected AT4K. This does not establish playback compatibility or persistence after reboot, AC loss or wake.

Playback, sound, physical remote Home/volume, HDMI operation and firmware updates remain unverified. Wholphin still needs a Jellyfin server/account. Package presence, input registration and visible app screens are not substitutes for these functional checks. No measured speedup or complete ad/tracking removal is claimed. See [CHANGELOG.md](CHANGELOG.md) for the dated observations and unresolved findings.
