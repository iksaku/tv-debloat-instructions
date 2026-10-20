# Hisense SmartTV 4K FFM / hengshan platform

Starting-point reference for the tested firmware: 32 optional packages disabled, screensaver off, and a custom launcher selected for native Home routing with two competing Home packages disabled. This is a platform identifier from ADB, **not a confirmed retail model**. Disable only features you do not use; these instructions are not a universal safe-disable list.

Go to [launcher replacement](#default-launcher-replacement), [disabled services](#disabled-services), [screensaver settings](#screensaver-setting), or [streaming apps](#streaming-apps) for commands and rollback. The [changelog](CHANGELOG.md) records changes and discoveries separately.

## Device identity and connection

| Property | Tested unit |
| --- | --- |
| Manufacturer / ADB model | Hisense / `SmartTV 4K FFM` |
| Product / device | `hengshan` |
| Board / platform / hardware | `his9618` / `merak` / `mt5896` |
| Android / API | 12 / 31 |
| Build / incremental | `STT2.230310.002` / `00.00.00.132` |
| Security patch | 2025-05-01 |
| ABI | 32-bit ARM (`armeabi-v7a`, `armeabi`) |
| Mode | Basic TV |
| Fingerprint | `Hisense/hengshan/hengshan:12/STT2.230310.002/00.00.00.132:user/release-keys` |

Use authorized local-network ADB over TCP port 5555, targeting Android user 0. Replace `<TV_IP>` with the intended TV address and verify the target before using any example. Record original package and component states first; rollback values below belong to this unit and may differ on another TV.

```sh
adb connect '<TV_IP>:5555'
adb -s '<TV_IP>:5555' shell getprop ro.product.model
adb -s '<TV_IP>:5555' shell getprop ro.build.fingerprint
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME
```

## Default launcher replacement

Use a compatible custom TV launcher of your choice. On this tested Android 12 firmware in Basic TV mode, native Home replacement required installing and selecting the custom launcher, then disabling **both Google TV Home and Setup Wizard**. Selecting a preferred Home alone was insufficient; disabling only Google TV Home exposed Setup Wizard's Recovery activity instead of the replacement.

AT4K was the tested launcher, not a required choice. These package steps describe this firmware's Home routing; they do not certify every launcher or other firmware. See the [changelog](CHANGELOG.md) for test results and limits.

Keep Android Settings (`com.android.tv.settings`) and its fallback Home enabled. Google TV Home also supplies dashboard, profile, media-control and other integrations. Setup Wizard handles setup/recovery, not just Home: disabling it may remove app-owned Wi-Fi configuration and cut off network ADB. Plan local network recovery before proceeding; this is not a universal safe-disable recipe.

### Install and select a custom launcher

Install a verified APK from the chosen launcher's trusted distribution source, compatible with Android 12 and this TV's 32-bit ARM ABI. Skip installation if your chosen launcher is already installed. List its available Home activity:

```sh
adb -s '<TV_IP>:5555' install -r --user 0 '<VERIFIED_REPLACEMENT_APK>'
adb -s '<TV_IP>:5555' shell cmd package query-activities --brief --user 0 -a android.intent.action.MAIN -c android.intent.category.HOME
```

Substitute the chosen launcher's actual `package/activity` from that list for `<REPLACEMENT_HOME_COMPONENT>` below. Open it and confirm its Home screen is usable, not merely a permission prompt behind a successful launch status, before selecting it:

```sh
adb -s '<TV_IP>:5555' shell am start --user 0 -n '<REPLACEMENT_HOME_COMPONENT>'
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 '<REPLACEMENT_HOME_COMPONENT>'
```

Turn off optional enhanced Home-key interception if previously enabled. The tested native-routing configuration leaves AT4K's **Enhanced home detection** off; enabling it broke volume-hold repetition. Accessibility permissions and other launcher-specific settings are separate from these package steps; approve any needed accessibility service deliberately on the TV rather than through ADB.

### Disable the two competing Home packages

Record each package's User 0 state before changing it. On the tested unit, both changed from default (`0`) to disabled-user (`3`), with their individual component overrides unchanged. These are whole-package disables, separate from the 32 optional-service disables below; no uninstall or data clear is required.

| Package | Recorded change | Exact package-state rollback |
| --- | --- | --- |
| `com.google.android.apps.tv.launcherx` | 0 → 3 | `pm default-state --user 0 com.google.android.apps.tv.launcherx` |
| `com.google.android.tungsten.setupwraith` | 0 → 3 | `pm default-state --user 0 com.google.android.tungsten.setupwraith` |

```sh
adb -s '<TV_IP>:5555' shell dumpsys package com.google.android.apps.tv.launcherx
adb -s '<TV_IP>:5555' shell dumpsys package com.google.android.tungsten.setupwraith
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.google.android.apps.tv.launcherx
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.google.android.tungsten.setupwraith
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief --user 0 -a android.intent.action.MAIN -c android.intent.category.HOME
```

The resolver should now select your custom Home component. From Settings, test the physical remote's Home button and volume holds; also check networking and the TV functions you use. Verify persistence after reboot/wake separately rather than assuming an immediate Home test covers it. If routing or required features fail, use the rollback below.

### Roll back to stock Home

Turn off any custom-launcher Home override you enabled so it cannot redirect Home during rollback. Restore both packages to their recorded default state before selecting stock Home; do not substitute `pm enable` for `pm default-state`.

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.tungsten.setupwraith
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.apps.tv.launcherx
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 com.google.android.apps.tv.launcherx/.home.VanillaModeHomeActivity
adb -s '<TV_IP>:5555' shell am start --user 0 -n com.google.android.apps.tv.launcherx/.home.VanillaModeHomeActivity
```

Verify physical Home returns to stock before removing a replacement launcher. Uninstalling it or clearing its data is not required for rollback. Restoring packages does not guarantee restoration of any network configuration removed during disabling.

### Tested launcher example, not a requirement

The verified example used AT4K Launcher 1.2 (version code `19`, package `com.overdevs.at4k`, Home component `com.overdevs.at4k/.MainActivity`). Physical Home and volume-down holds worked with both competing packages disabled. These app-specific settings record that example; they are not prerequisites for every custom launcher.

| AT4K setting | Original value | Recorded current value |
| --- | --- | --- |
| Accessibility service (`com.overdevs.at4k/com.overdevs.at4k.Hra`) | Off | On, manually approved on the TV |
| Settings → Controls → Override Current Launcher | Off | On |
| Settings → Controls → Enhanced home detection | Off | Off |
| Settings → Controls → Start on boot | Off | Off |
| Settings → Controls → Override back button exit | On | On |

To undo these app-specific changes, turn **Override Current Launcher** off and switch AT4K's accessibility service off manually in TV Settings. Leave the other values as shown, and use the package/Home rollback above to restore stock navigation. Accessibility remaining enabled in the successful test does not establish that other launchers require it; the native Home resolver itself selected the custom activity.

## Disabled services

State `0` means manifest/default state, `1` explicitly enabled, and `3` disabled-user. Package-level `disable-user` retains the installed package and its data; it is not an uninstall. Preserve original component overrides and unrelated preexisting disabled packages.

All 32 packages below are disabled in the recorded configuration. Each was originally at default state (`0`), which is the rollback target. Before disabling DreamX, turn off the [screensaver setting](#screensaver-setting).

| Purpose | Exact package | Recorded change | Exact package-state rollback |
| --- | --- | --- | --- |
| ACR service | `com.vt.tv.evservice` | 0 → 3 | `pm default-state --user 0 com.vt.tv.evservice` |
| Google feedback | `com.google.android.feedback` | 0 → 3 | `pm default-state --user 0 com.google.android.feedback` |
| Hisense Channels streaming / search / dedicated-button integration | `net.vidaatv.hisense.android.tv` | 0 → 3 | `pm default-state --user 0 net.vidaatv.hisense.android.tv` |
| Alexa TV control / setup | `com.vt.alexa` | 0 → 3 | `pm default-state --user 0 com.vt.alexa` |
| Google Calendar synchronization | `com.google.android.syncadapters.calendar` | 0 → 3 | `pm default-state --user 0 com.google.android.syncadapters.calendar` |
| Android printing | `com.android.printspooler` | 0 → 3 | `pm default-state --user 0 com.android.printspooler` |
| TalkBack and Switch Access | `com.google.android.marvin.talkback` | 0 → 3 | `pm default-state --user 0 com.google.android.marvin.talkback` |
| Family Moment photo / camera / screensaver features | `com.vt.hishow` | 0 → 3 | `pm default-state --user 0 com.vt.hishow` |
| BT Music mode | `com.vt.tv.singlelisten` | 0 → 3 | `pm default-state --user 0 com.vt.tv.singlelisten` |
| Basic alternative screensavers | `com.android.dreams.basic` | 0 → 3 | `pm default-state --user 0 com.android.dreams.basic` |
| Google speech synthesis, recognition and voice input | `com.google.android.tts` | 0 → 3 | `pm default-state --user 0 com.google.android.tts` |
| Google TV remote / keyboard / search integration | `com.google.android.tv.remote.service` | 0 → 3 | `pm default-state --user 0 com.google.android.tv.remote.service` |
| Ambient Mode / Backdrop and associated features | `com.google.android.apps.tv.dreamx` | 0 → 3 | `pm default-state --user 0 com.google.android.apps.tv.dreamx` |
| Hisense OEM-tab content | `com.vidaa.oemtabcontentprovider` | 0 → 3 | `pm default-state --user 0 com.vidaa.oemtabcontentprovider` |
| Store / demo mode and its SyncPlay service | `com.vt.storemode` | 0 → 3 | `pm default-state --user 0 com.vt.storemode` |
| B2B LAN / serial control | `com.vt.tv.b2b` | 0 → 3 | `pm default-state --user 0 com.vt.tv.b2b` |
| Play auto-install configuration | `android.autoinstalls.config.google.gtvpai` | 0 → 3 | `pm default-state --user 0 android.autoinstalls.config.google.gtvpai` |
| DLNA content sharing | `com.vt.tv.dlna` | 0 → 3 | `pm default-state --user 0 com.vt.tv.dlna` |
| Media Center / USB playback / DLNA sharing controls | `com.vt.tv.mediacenter` | 0 → 3 | `pm default-state --user 0 com.vt.tv.mediacenter` |
| Screen Casting / Miracast and related connection screens | `com.vt.tv.mobiledeviceconnectiontips` | 0 → 3 | `pm default-state --user 0 com.vt.tv.mobiledeviceconnectiontips` |
| AIOT smart-home integration | `com.vt.tv.aiot` | 0 → 3 | `pm default-state --user 0 com.vt.tv.aiot` |
| AIOT IP control and device allow/deny lists | `com.vt.tv.aiotprotocolapp` | 0 → 3 | `pm default-state --user 0 com.vt.tv.aiotprotocolapp` |
| AIOT power / volume / source control bridge | `com.vt.tv.aiotsystemservice` | 0 → 3 | `pm default-state --user 0 com.vt.tv.aiotsystemservice` |
| Camera gestures and camera-assisted picture / audio features | `com.vt.tv.ai_sense` | 0 → 3 | `pm default-state --user 0 com.vt.tv.ai_sense` |
| SyncPlay discovery server | `com.vt.udpserver` | 0 → 3 | `pm default-state --user 0 com.vt.udpserver` |
| Hisense Home partner configuration / favorites / recommendations | `com.vt.tv.customerlauncher` | 0 → 3 | `pm default-state --user 0 com.vt.tv.customerlauncher` |
| Calman external display-calibration connector | `com.vt.tv.calmanconnector` | 0 → 3 | `pm default-state --user 0 com.vt.tv.calmanconnector` |
| Ginga broadcast-interactive applications | `tv.mopa.ginga` | 0 → 3 | `pm default-state --user 0 tv.mopa.ginga` |
| ATSC3 broadcast interaction | `com.vt.tv.interactive` | 0 → 3 | `pm default-state --user 0 com.vt.tv.interactive` |
| Device Assistant cleanup / app management / diagnostics | `com.vt.tv.protector` | 0 → 3 | `pm default-state --user 0 com.vt.tv.protector` |
| Storage-full / storage-protection / USB-storage prompts | `com.vt.tv.smartsense.ui` | 0 → 3 | `pm default-state --user 0 com.vt.tv.smartsense.ui` |
| Vendor web-page viewer, separate from the shared WebView engine | `com.mediatek.support.webview` | 0 → 3 | `pm default-state --user 0 com.mediatek.support.webview` |

### Disable one selected package and roll it back

Substitute one exact package from the table for `<PACKAGE_FROM_TABLE>`. Record its own User 0 state first; the rollback below applies to the recorded default state on this unit.

```sh
adb -s '<TV_IP>:5555' shell dumpsys package '<PACKAGE_FROM_TABLE>'
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 '<PACKAGE_FROM_TABLE>'
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
# Rollback to the recorded default state (0):
adb -s '<TV_IP>:5555' shell pm default-state --user 0 '<PACKAGE_FROM_TABLE>'
```

Alternatively, prepend `adb -s '<TV_IP>:5555' shell` to a rollback command in the table. Do not use `pm enable` as a substitute for `pm default-state`: explicit enabled state (`1`) is different from the recorded default (`0`). Do not enable all disabled packages indiscriminately.

Re-enabling speech or accessibility packages does not by itself select their engine or services; review those settings deliberately when restoring the features.

## Screensaver setting

The recorded `secure screensaver_enabled` value changed from `1` to `0`. Turn it off before disabling `com.google.android.apps.tv.dreamx`:

```sh
adb -s '<TV_IP>:5555' shell settings --user 0 get secure screensaver_enabled
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 0
```

The component preference was left at `com.google.android.apps.tv.dreamx/.service.Backdrop`. To roll back, restore the package before restoring the screensaver setting:

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.apps.tv.dreamx
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 1
```

## Streaming apps

YouTube, Prime Video and Disney+ are explicitly enabled in this configuration. They were previously disabled; the rollback column below restores that earlier state only if desired. It is not part of undoing the disabled-service list.

| Purpose | Exact package | Recorded change | Exact package-state rollback |
| --- | --- | --- | --- |
| Restore YouTube access | `com.google.android.youtube.tv` | 3 → 1 | `pm disable-user --user 0 com.google.android.youtube.tv` |
| Restore Prime Video access | `com.amazon.amazonvideo.livingroom` | 3 → 1 | `pm disable-user --user 0 com.amazon.amazonvideo.livingroom` |
| Restore Disney+ access | `com.disney.disneyplus` | 3 → 1 | `pm disable-user --user 0 com.disney.disneyplus` |

To reproduce a restore, substitute its package below. Prepend the same ADB target and `shell` prefix to its rollback command in the table.

```sh
adb -s '<TV_IP>:5555' shell pm enable --user 0 '<STREAMING_PACKAGE_FROM_TABLE>'
```

Netflix (`com.netflix.ninja`) remains enabled.

## Packages left unchanged

Keep Settings, System UI, Play Services/GSF, keyboard, Bluetooth/Wi-Fi, shared Google WebView, TV/input/audio/DRM services and TV/remote firmware updates outside this disable list. Stock Home and Setup Wizard are handled separately in the [launcher replacement steps](#default-launcher-replacement). The mixed-function `com.vt.tv.smartsense` and `com.vt.tv.smartservice` packages remain enabled; they are distinct from the disabled storage-prompt UI.

AirPlay and its key-provisioning package states were left unchanged: `com.mediatek.AirplayAPK=1`, `com.mediatek.airplaydaemon=0`, `com.mediatek.partner.airplay_sys=0`, `com.vt.tv.airplaykeycheck=0`, and `com.vt.edv.drmp=0`. Disabling `com.vt.tv.mobiledeviceconnectiontips` also removes an AirPlay-related entry screen; retaining the core packages does not establish that every AirPlay workflow remains available.

The recorded configuration has 40 disabled packages: the 32 listed services, the two competing Home packages, and six preexisting disables outside this cleanup: `com.android.providers.calendar`, `com.android.vending`, `com.hisense.odinbrowser`, `com.mediatek.dtv.tvinput.dvbtuner`, `com.mediatek.dtv.tvinput.isdbtuner`, and `com.mediatek.tis`. Leave these preexisting states alone when rolling back the changes above.
