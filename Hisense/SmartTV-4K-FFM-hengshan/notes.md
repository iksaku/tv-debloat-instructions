# Hisense SmartTV 4K FFM / hengshan platform

Completed investigation and partial cleanup results recorded on 2026-10-18. This is a platform identifier from ADB, **not a confirmed retail model**. These notes describe the completed four-package cleanup and launcher investigation checkpoint; they do not declare the entire TV setup complete.

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

The connection used authorized local-network ADB over TCP port 5555, Android user 0. Replace `<TV_IP>` with the intended TV address and verify the target before using any example. Record original package and component states first; these examples are not a blanket safe-disable list.

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

## Completed package changes

State `0` means manifest/default state, `1` explicitly enabled, and `3` disabled-user. Package-level `disable-user` retains the installed package and its data; it is not an uninstall. Preserve original component overrides and unrelated preexisting disabled packages.

| Purpose | Exact package | Recorded change | Exact package-state rollback |
| --- | --- | --- | --- |
| ACR service | `com.vt.tv.evservice` | 0 → 3 | `pm default-state --user 0 com.vt.tv.evservice` |
| Google feedback | `com.google.android.feedback` | 0 → 3 | `pm default-state --user 0 com.google.android.feedback` |
| Hisense Channels streaming / search / dedicated-button integration | `net.vidaatv.hisense.android.tv` | 0 → 3 | `pm default-state --user 0 net.vidaatv.hisense.android.tv` |
| Alexa TV control / setup | `com.vt.alexa` | 0 → 3 | `pm default-state --user 0 com.vt.alexa` |
| Restore YouTube access | `com.google.android.youtube.tv` | 3 → 1 | `pm disable-user --user 0 com.google.android.youtube.tv` |
| Restore Prime Video access | `com.amazon.amazonvideo.livingroom` | 3 → 1 | `pm disable-user --user 0 com.amazon.amazonvideo.livingroom` |
| Restore Disney+ access | `com.disney.disneyplus` | 3 → 1 | `pm disable-user --user 0 com.disney.disneyplus` |

Netflix (`com.netflix.ninja`) remained enabled. Restoring the original state of the three streaming apps means disabling them again, not resetting their data. The four confirmed optional-package disables are ACR, Google feedback, Hisense Channels and Alexa. Channels and Alexa were tested individually after the earlier two-package cleanup.

Disabling Channels removes its streaming service and related search/button integration; disabling Alexa removes its TV voice-control/setup feature. The [official exact-package Channels listing](https://play.google.com/store/apps/details?id=net.vidaatv.hisense.android.tv) describes advertising-supported live and on-demand streaming. The [Hisense Alexa guide](https://assets.hisense-usa.com/assets/ProductDownloads/46/3d89c6aa49/How-to-set-up-Amazon-Alexa-on-Hisense-TVs_0.pdf) corroborates the general TV-control role on older models, not this exact firmware. Local package metadata identified the installed packages; neither source establishes a universal safe-disable recommendation.

For one reviewed package at a time, prepend `adb -s '<TV_IP>:5555' shell` to a rollback command in the table. Examples of the completed changes and inspection:

```sh
adb -s '<TV_IP>:5555' shell dumpsys package com.vt.tv.evservice
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.vt.tv.evservice
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.google.android.feedback
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 net.vidaatv.hisense.android.tv
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.vt.alexa
adb -s '<TV_IP>:5555' shell pm enable --user 0 com.google.android.youtube.tv
adb -s '<TV_IP>:5555' shell pm enable --user 0 com.amazon.amazonvideo.livingroom
adb -s '<TV_IP>:5555' shell pm enable --user 0 com.disney.disneyplus
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
```

## Preserved functionality and verification limits

AirPlay package states were preserved exactly: `com.mediatek.AirplayAPK=1`, `com.mediatek.airplaydaemon=0`, `com.mediatek.partner.airplay_sys=0`, and `com.vt.tv.airplaykeycheck=0`. AirPlay system/daemon services were observed after the changes and reboot. DLNA, SyncPlay, AIOT, native audio/input/network services and DRM support were retained. Stock launcher, Setup Wizard and Settings remained at default state; Home still resolved to Vanilla. No additional optional-package changes beyond the four listed disables are claimed.

YouTube, Prime Video and Disney+ returned successful launch status before and after the two-package disable batch; Netflix and Settings also launched. One normal reboot verified persistence of the earlier ACR and feedback disables, streaming-app enabled states, preserved AirPlay states and bound accessibility. Core audio, input, TV, Settings and remote services were observed; these process checks are not full functional tests.

After each later Channels/Alexa disable, Netflix, YouTube, Prime Video and Disney+ launched successfully. An explicit Settings activity launch reported a stock-launcher account-verification activity; the normal `android.settings.SETTINGS` action then reached MainSettings, confirmed in the foreground. No login or provisioning action was performed, and the cause of that transient activity is unestablished. AirPlay services remained running, ADB stayed responsive, and HDMI/eARC, audio, input, remote, Wi-Fi and DRM services/endpoints were present. **The latest Channels and Alexa disables have no reboot persistence or standby/wake retest.**

Physical Home failure was user-confirmed, but authenticated streaming playback, audio, HDMI switching, and AirPlay discovery/streaming were not comprehensively verified in this checkpoint. No measured speedup, removal of all tracking/ads, or elimination of all background processes is claimed. The debloat changes used disable-user without uninstall or data clear. The replacement launcher alone was subsequently uninstalled for user 0 with the keep-data option, as documented above.
