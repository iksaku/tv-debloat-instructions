# TCL BeyondTV / RT2851 platform

Verified observations from two anonymized units, A and B, on 2026-10-18. **BeyondTV-RT2851 is a platform directory, not a confirmed retail model.** Matching firmware does not establish identical retail hardware or guarantee the same results.

## Device identity

| Property | Both tested units |
| --- | --- |
| Manufacturer / ADB model | TCL / BeyondTV |
| Android / API | 11 / 30 |
| Software | V8-R851T02-LF1V715, incremental AR05 |
| Security patch | 2021-12-05 |
| ABI | 32-bit ARM (`armeabi-v7a`, `armeabi`) |
| RAM | 2 GiB |
| Build fingerprint | `TCL/BeyondTV/BeyondTV:11/PPR1.180610.011/AR05:user/release-keys` |

Unit A reports board `RTD2851` and hardware/platform `rtd285o`. Its rear label was tentatively read as **50A423**, not confirmed; its factory menu showed project **50P8** and panel **V500DJ6**. These are distinct identifiers. Unit B's retail model is unconfirmed and is not inferred from its matching fingerprint.

## Connection and baseline

Both units were accessed with authorized ADB over the local network on TCP port 5555, targeting Android user 0. Enable the TV's developer/ADB access as appropriate for its firmware and approve the computer on the TV. Use a trusted network; do not expose ADB to the Internet. The examples below are documentation, not a universal debloat script.

Replace `<TV_IP>` with the intended TV address. Confirm the physical target as well as its properties: the two tested units have the same fingerprint.

```sh
adb connect '<TV_IP>:5555'
adb -s '<TV_IP>:5555' shell getprop ro.product.model
adb -s '<TV_IP>:5555' shell getprop ro.product.board
adb -s '<TV_IP>:5555' shell getprop ro.build.fingerprint
adb -s '<TV_IP>:5555' shell getprop ro.software.version_id
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME
```

Record each candidate's `dumpsys package` **User 0** state before changing it. `enabled=0` means manifest/default state; `enabled=1` means explicitly enabled; `enabled=3` means disabled-user. Keep existing component overrides and previously disabled packages intact. A package disabled before this work is not counted as a task change.

## Default launcher replacement

**Verdict: worked with caveats on the tested Android 11 / V8-R851T02-LF1V715 (AR05) units.** Disabling the stock launcher at package level, after verifying a replacement was installed and usable, was decisive. Normal Home selection and accessibility alone were insufficient in the observed setup. This result does not establish compatibility with every replacement launcher or other TCL firmware.

**What was tried:** the replacement was installed and opened, selected as Home, and its accessibility-based Home handling was investigated. The supported package-level command then changed `com.google.android.tvlauncher` from default (`0`) to disabled-user (`3`); no component-security bypass was used.

**What was found:** a selected replacement or enabled accessibility setting alone did not establish reliable Home behavior. Unit A also showed an enabled accessibility service that was not bound at a recorded checkpoint. With the stock launcher disabled, the Home resolver and foreground checks reached the replacement.

**Final observed result:** unit A's replacement Home and stock launcher disabled state survived the recorded AC power cycle; later package checks still returned to replacement Home, but no reboot after its final PartnerCustomizer/eSticker changes is confirmed. Unit B's Home navigation from Settings and Netflix succeeded, and its replacement Home plus bound accessibility service persisted after one normal reboot covering all 26 package disables.

The relevant command pattern is shown below for an already installed and verified replacement. Substitute its actual component for `<REPLACEMENT_HOME_COMPONENT>`; do not paste the placeholder literally. Verify that the replacement activity is actually visible, not merely a permission prompt behind a successful launch status.

```sh
adb -s '<TV_IP>:5555' shell am start -n '<REPLACEMENT_HOME_COMPONENT>'
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 '<REPLACEMENT_HOME_COMPONENT>'
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.google.android.tvlauncher
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME
adb -s '<TV_IP>:5555' shell input keyevent KEYCODE_HOME
```

Confirm the resulting screen and physical remote Home behavior. The stock Home rollback commands below restore its recorded default state before selecting and opening it.

## Completed package changes

The following 26 packages were changed from default state (`0`) to disabled-user (`3`) on **both** units. Purpose labels describe the affected feature areas, not proof that every component of a package has only that function. Disable only features you do not use; in particular, retain accessibility, voice, casting, and remote services when needed.

| Purpose / tradeoff | Exact packages |
| --- | --- |
| Stock Home and recommendations | `com.google.android.tvlauncher`, `com.google.android.tvrecommendations` |
| TCL content, promotions and messages | `com.tcl.waterfall.overseas`, `com.tcl.overseasappshow`, `com.tcl.messagebox` |
| TCL Home integration | `com.tcl.tv.tclhome_master`, `com.tcl.tv.tclhome_passive` |
| TCL analytics | `com.tcl.bi` |
| Partner customization and retail sticker | `com.tcl.partnercustomizer`, `com.tcl.esticker` |
| Alexa integration | `com.tcl.smartalexa` |
| Miracast and multiscreen integration | `com.tcl.miracast`, `com.tcl.MultiScreenInteraction_TV` |
| Google Cast receiver | `com.google.android.apps.mediashell` |
| Google search / voice assistant | `com.google.android.katniss`, `com.google.android.tv.assistant` |
| Games, feedback and printing | `com.google.android.play.games`, `com.google.android.feedback`, `com.android.printspooler` |
| Calendar synchronization | `com.google.android.syncadapters.calendar` |
| TalkBack and text-to-speech | `com.google.android.marvin.talkback`, `com.google.android.tts` |
| Screensaver providers | `com.google.android.backdrop`, `com.android.dreams.basic` |
| TCL media browser | `com.tcl.ui_mediaCenter` |
| Google TV remote service | `com.google.android.tv.remote.service` |

**Stock launcher prerequisite:** a replacement launcher must already be installed, selected, and verified through Home navigation before disabling `com.google.android.tvlauncher`.

| Difference | Unit A | Unit B |
| --- | --- | --- |
| `com.google.android.videos` (Google TV/Movies) | Explicitly enabled (`1`) → disabled-user (`3`) | Retained as a streaming app |
| Final task package disables | 27 | 26 |
| Final disabled-package total | Not used as a task-change count | 47: 26 task changes + 21 preexisting |
| `com.tcl.usercenter` | Attempted disable did not persist; returned to default after reboot and was left there | Retained |

### Reproduce one chosen change and restore its original state

For example, the recorded original state of the feedback package was default on both units:

```sh
adb -s '<TV_IP>:5555' shell dumpsys package com.google.android.feedback
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.google.android.feedback
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
# Rollback to the recorded default state (0):
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.feedback
```

Use the same command pattern, one reviewed package at a time, for the exact packages in the table. `disable-user` leaves the package installed and retains its data; it is not an uninstall or a data clear. Restoring default state is different from forcing enabled state. For unit A's Google TV/Movies exception, restore explicit enabled state instead:

```sh
adb -s '<TV_IP>:5555' shell pm enable --user 0 com.google.android.videos
```

To return Home to stock on either tested unit, restore its original state first, select it, and verify it opens:

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.tvlauncher
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 com.google.android.tvlauncher/.MainActivity
adb -s '<TV_IP>:5555' shell am start -n com.google.android.tvlauncher/.MainActivity
```

If a replacement's accessibility service redirects Home, turn that service off manually when reverting Home. No uninstall or reset is required. Restore other package states individually according to the tables; do not enable all disabled packages indiscriminately.

## Additional settings

Both units changed `secure screensaver_enabled` from `1` to `0`. The screensaver component settings still referenced Backdrop; disabling the providers alone had not changed that configuration. Both also restricted Netflix's `RUN_ANY_IN_BACKGROUND` app-op from no explicit override (effective allow) to `deny`.

```sh
# Inspect first and keep your own original values.
adb -s '<TV_IP>:5555' shell settings --user 0 get secure screensaver_enabled
adb -s '<TV_IP>:5555' shell cmd appops get --user 0 com.netflix.ninja RUN_ANY_IN_BACKGROUND
# Changes made on these units:
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 0
adb -s '<TV_IP>:5555' shell cmd appops set --user 0 com.netflix.ninja RUN_ANY_IN_BACKGROUND deny
# Rollback to these units' recorded originals:
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 1
adb -s '<TV_IP>:5555' shell cmd appops set --user 0 com.netflix.ninja RUN_ANY_IN_BACKGROUND default
```

Netflix still launched after the restriction. App-ops do not guarantee the absence of cached or other background processes. Do not substitute a global app-op reset for this single-operation rollback.

## Exceptions and preserved services

- `com.tcl.smartalexa` still had a persistent process despite its disabled package state, including after reboot. It was not uninstalled.
- `com.tcl.usercenter` reenabled itself on unit A and remained present; it was retained on unit B. It was not uninstalled.
- `com.tcl.guard`, mixed-function `com.tcl.assistant`, and `com.tcl.xian.StartandroidService` were preserved rather than treating them as isolated advertising packages.
- Core TV/input, settings, configuration, update, audio, DRM, Wi-Fi, Bluetooth, keyboard and physical remote services were preserved. Examples include `com.tcl.tvinput`, `com.tcl.settings`, `com.android.tv.settings`, `com.tcl.providers.config`, `com.tcl.versionUpdateApp`, `com.tcl.systemserver`, and `com.tcl.keycustomfunctionservice`.

No root access, package uninstall, app-data clear, or bootloader changes were used for this cleanup.

## Verification and limits

**Unit A:** the first 25 final package disables, screensaver setting and Netflix restriction survived an AC power cycle. The later PartnerCustomizer and eSticker disables passed package-state, Settings/YouTube/Netflix launch, Home foreground, and protected-service checks. No reboot after those last two changes is confirmed. Their tests did not establish authenticated playback or physical HDMI/audio functionality. The recorded keyboard and accessibility settings remained unchanged during those checks; an enabled accessibility service was not necessarily bound.

**Unit B:** all 26 task disables persisted after one normal reboot, with a changed boot ID and `sys.boot_completed=1`; all 21 preexisting disabled packages remained disabled. Home resolution, the replacement Home's bound accessibility service, keyboard state, `screensaver_enabled=0`, and Netflix's app-op restriction persisted. Settings, Netflix, YouTube, Disney+, HBO Max, ViX and Google TV/Movies launched successfully. Prime Video initially timed out, then passed a bounded launch recheck with its activity in the foreground. Audio, Widevine/PlayReady, Wi-Fi, Bluetooth, physical remote and TV input processes were present, with HDMI ports 1–4 registered. No fatal exception entries appeared in the retained crash-buffer sample. The user confirmed audio/video worked generally; this does not independently verify every HDMI port, remote function, DRM path or streaming service's playback.

Useful read-only checks after a selected change:

```sh
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
adb -s '<TV_IP>:5555' shell dumpsys package com.google.android.feedback
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME
adb -s '<TV_IP>:5555' shell settings --user 0 get secure default_input_method
adb -s '<TV_IP>:5555' shell dumpsys tv_input
adb -s '<TV_IP>:5555' shell getprop sys.boot_completed
```

A successful app launch or a present DRM/input process is not a playback test. These records do not establish that all ads or background tasks disappeared, or that performance improved measurably. Memory readings were workload snapshots, not controlled benchmarks. Firmware updates or different regional hardware may change package behavior.

## Unit A only: power on after AC restoration

This factory-menu change was performed on **unit A only**. It is model/firmware specific and is not established for unit B or for all TVs sharing this platform. Factory menus contain reset and calibration controls: leave those and all unrelated settings untouched.

With an HDMI input selected, the user opened Picture → Advanced and highlighted the plain **Contrast** item. The code **1950** was entered once via ADB numeric key events:

```sh
adb -s '<TV_IP>:5555' shell input keyevent 8 16 12 7
```

These key codes correspond to digits 1, 9, 5, 0. The main Factory menu opened; the user manually changed **Power on mode: STANDBY → ON**, exited, and disconnected/reconnected AC. The user verified the TV started without a remote action or HDMI prompt. Post-boot checks independently confirmed a new boot.

Rollback is to return to the same menu on this verified unit and manually restore **Power on mode = STANDBY**. No other factory setting was changed. Do not blindly enter the sequence on another firmware or use factory reset as rollback.
