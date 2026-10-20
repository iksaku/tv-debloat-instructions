# Hisense SmartTV 4K FFM / hengshan platform

Starting-point reference for the tested firmware: 32 optional packages disabled, screensaver off, and stock Vanilla Home retained. This is a platform identifier from ADB, **not a confirmed retail model**. Disable only features you do not use; these instructions are not a universal safe-disable list.

Go to [disabled services](#disabled-services), [screensaver settings](#screensaver-setting), or [streaming apps](#streaming-apps) for commands and rollback. The [changelog](CHANGELOG.md) records changes and discoveries separately.

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

**Verdict: not achieved by the tested methods on this Android 12 / 00.00.00.132 firmware in Basic TV mode.** A replacement could be launched directly, but Home continued to select stock Vanilla Home. The replacement was subsequently removed for user 0, and stock Home was explicitly selected. This is a bounded test result, not a claim that replacement is universally impossible.

### What was tried

| Method | Observed result |
| --- | --- |
| Normal Home preference / HOME role selection | The replacement was stored as the preferred Home and HOME-role holder; the normal default-app screen showed it selected. Home still chose Vanilla. |
| User-enabled accessibility | The replacement's service was enabled, bound, running, and reported active controls. Injected Home from Settings and YouTube still returned to Vanilla. The user also confirmed the same outcome with physical remote Home. |
| One normal reboot | A new boot and completed startup were verified. Home role and bound accessibility persisted, but Home still selected Vanilla. |
| Individual Vanilla component disable | Android rejected the shell operation with `SecurityException`; no component state changed. |
| Whole `com.google.android.apps.tv.launcherx` package disable | The supported package operation succeeded, but Home resolution promoted Setup Wizard's `RecoveryActivity`, not the replacement. The package was immediately restored to its exact original default state. No Home key or Setup Wizard launch was requested while stock was disabled. |
| Explicit `MAIN` + `HOME` launch of replacement | The replacement task became `type=home`, but foreground was a permission activity, and the next Home key still resumed Vanilla. No permission was granted or denied in that diagnostic. |

The whole-package trial's preflight also returned a successful start status while a permission activity was foreground. Earlier direct rendering of the replacement had been verified, but that preflight was not fresh confirmation of its visible Home screen.

### What was found

During the investigation, before removal of the replacement, the observed Home candidates were:

| Component | Priority / state |
| --- | --- |
| `com.google.android.apps.tv.launcherx/.home.VanillaModeHomeActivity` | 2; component explicitly enabled, package default |
| `com.google.android.tungsten.setupwraith/.RecoveryActivity` | 1; component explicitly enabled, package default |
| `<REPLACEMENT_HOME_COMPONENT>` | 0; enabled |
| `com.android.tv.settings/.system.FallbackHome` | -1000; enabled |

The normal Google TV Home component and Setup Wizard's higher-priority `MainActivity` were disabled at baseline. The three recorded provisioning flags were already `1`; an unset general setup-completion flag did not explain this result. The enabled Recovery activity remained above the replacement when Vanilla was removed from resolution.

The stock package also contains dashboard, profiles, media controls, input/boot hooks and sleep-timer functions. Setup Wizard contains deferred setup and other system integration, not just a Home candidate. Neither package is an isolated recommendation feature. Completing full Google TV setup was not tested as a launcher fix, and its effect on these priorities is not established.

### Final observed result and reproducible diagnostics

Stock `com.google.android.apps.tv.launcherx` was restored to **default (`enabled=0`)**, preserving its existing component overrides. Vanilla Home opened successfully after rollback; ADB and Settings remained usable. Setup Wizard was not disabled. No root, reset, provisioning edits, or component-security bypass was used. Reliable default Home takeover was not achieved. At the user's request, the replacement was then removed for user 0 using a keep-data uninstall; stock Vanilla Home is now the HOME-role holder and launches successfully.

The normal-selection and explicit-Home diagnostic patterns were:

```sh
# Substitute the installed replacement's actual component.
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 '<REPLACEMENT_HOME_COMPONENT>'
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME
adb -s '<TV_IP>:5555' shell am start -a android.intent.action.MAIN -c android.intent.category.HOME -n '<REPLACEMENT_HOME_COMPONENT>'
adb -s '<TV_IP>:5555' shell input keyevent KEYCODE_HOME
```

These failed to establish replacement Home here. The historical whole-package trial used `pm disable-user --user 0 com.google.android.apps.tv.launcherx`; it is not a recommended fix. Its exact rollback was:

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.apps.tv.launcherx
adb -s '<TV_IP>:5555' shell am start -n com.google.android.apps.tv.launcherx/.home.VanillaModeHomeActivity
```

After restoring the stock package, its Home preference was explicitly selected before removing the replacement:

```sh
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 com.google.android.apps.tv.launcherx/.home.VanillaModeHomeActivity
```

The completed removal used the following pattern, with the installed replacement's actual package substituted:

```sh
adb -s '<TV_IP>:5555' shell pm uninstall -k --user 0 '<REPLACEMENT_PACKAGE>'
```

The command returned success. The installed-package listing excluded the replacement and its launcher activity no longer resolved for user 0. A retained package record remained visible with the include-uninstalled query; OEM metadata also retained an inconsistent installed flag. Installed-package availability and activity resolution established the removal outcome. The `-k` option requested data retention, but data bytes were not independently inspected or backed up.

Stock Home launched after removal, and the normal Settings action reached MainSettings. The four debloat disables and AirPlay package/service states were unchanged. **No reboot followed the removal.** Do not disable every Home handler or Setup Wizard to force the resolver to choose a replacement.

Removal rollback, not executed: reinstall a verified APK with the same signing identity using `adb -s '<TV_IP>:5555' install -r '<VERIFIED_REPLACEMENT_APK>'`. Retained data may be reused; permissions, accessibility and Home selection require deliberate review rather than automatic re-enablement.

### External evidence, separate from our tests

- The [OpenCore TV migration audit](https://github.com/eponce00/OpenCore-TV/blob/main/docs/DEVICE_PROFILE_MIGRATION_AUDIT.md) reports a `SmartTV_4K_FFM` / `hengshan` device where disabling stock Home exposed Recovery, and disabling Setup Wizard was followed by network/ADB loss. Its firmware is unconfirmed; matching platform strings do not prove the same outcome or cause on this unit.
- [Launcher issue #605](https://github.com/spocky/miproja1/issues/605) reports a temporary explicit `MAIN` + `HOME` workaround with a different launcher and device. Our corresponding diagnostic created a Home-type task but did not fix Home routing.
- [Android 12 Wi-Fi service source](https://android.googlesource.com/platform/packages/modules/Wifi/+/refs/tags/android-mainline-12.0.0_r122/service/java/com/android/server/wifi/WifiServiceImpl.java) contains removal of app-owned network state when a creator app is disabled or removed. This is a possible mechanism, not a verified cause on this Hisense firmware. Limited local metadata did not establish Setup Wizard ownership of the saved network. The external network-loss report remains a risk signal, not a reproduced local failure.

No third-party debloat code was run.

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

Keep stock Home, Setup Wizard, Settings, System UI, Play Services/GSF, keyboard, Bluetooth/Wi-Fi, shared Google WebView, TV/input/audio/DRM services and TV/remote firmware updates outside this disable list. The mixed-function `com.vt.tv.smartsense` and `com.vt.tv.smartservice` packages remain enabled; they are distinct from the disabled storage-prompt UI.

AirPlay and its key-provisioning package states were left unchanged: `com.mediatek.AirplayAPK=1`, `com.mediatek.airplaydaemon=0`, `com.mediatek.partner.airplay_sys=0`, `com.vt.tv.airplaykeycheck=0`, and `com.vt.edv.drmp=0`. Disabling `com.vt.tv.mobiledeviceconnectiontips` also removes an AirPlay-related entry screen; retaining the core packages does not establish that every AirPlay workflow remains available.

The recorded configuration has 38 disabled packages: the 32 listed services plus six preexisting disables outside this cleanup: `com.android.providers.calendar`, `com.android.vending`, `com.hisense.odinbrowser`, `com.mediatek.dtv.tvinput.dvbtuner`, `com.mediatek.dtv.tvinput.isdbtuner`, and `com.mediatek.tis`. Leave these preexisting states alone when rolling back the changes above.
