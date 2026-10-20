# TCL BeyondTV / RT2851 platform

Cleanup reference for two tested units, A and B: 26 common package disables, screensaver off, a Netflix background restriction and replacement Home. **BeyondTV-RT2851 is a platform directory, not a confirmed retail model.** Matching firmware does not establish identical retail hardware or guarantee the same results. Disable only features you do not use.

The [three additional optional packages](#additional-optional-packages-on-one-reviewed-unit) apply to one reviewed unit, not both. See [settings and rollback](#additional-settings), [verification limits](#verification-and-limits), and the [changelog](CHANGELOG.md) for dated changes and unresolved findings.

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

Use authorized ADB over the local network on TCP port 5555, targeting Android user 0. Enable the TV's developer/ADB access as appropriate for its firmware and approve the computer on the TV. Use a trusted network; do not expose ADB to the Internet. The examples below are documentation, not a universal debloat script.

Replace `<TV_IP>` with the intended TV address. Confirm the physical target as well as its properties: the two tested units have the same fingerprint.

```sh
adb connect '<TV_IP>:5555'
adb -s '<TV_IP>:5555' shell getprop ro.product.model
adb -s '<TV_IP>:5555' shell getprop ro.product.board
adb -s '<TV_IP>:5555' shell getprop ro.build.fingerprint
adb -s '<TV_IP>:5555' shell getprop ro.software.version_id
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief --user 0 -a android.intent.action.MAIN -c android.intent.category.HOME
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

The following 26 packages are disabled on **both** units, each from original default state (`0`) to disabled-user (`3`). Restore default state when rolling back one of these packages. Purpose labels describe the affected feature areas, not proof that every component of a package has only that function. Retain accessibility, voice, casting, and remote services when needed. Turn off the [screensaver setting](#additional-settings) before disabling its providers.

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

Unit B's recorded 47 disabled packages include these 26 changes and 21 preexisting disables outside this cleanup. Leave unrelated preexisting states alone when rolling back; the changelog records the original unit-specific change counts and exceptions.

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

## Additional optional packages on one reviewed unit

The following three packages are disabled for user 0 on one reviewed Android 11 / `V8-R851T02-LF1V715` (`AR05`) unit. These additions are not established on both historical units, and matching firmware alone does not identify a unit as A or B. Each changed from manifest/default state (`0`) to disabled-user (`3`); package data and existing component overrides were retained.

| Purpose / feature lost | Exact package | Recorded change | Exact package-state rollback |
| --- | --- | --- | --- |
| USB copying of retail-demo videos into eSticker resources, with demo-copy status / factory LED integration | `com.tcl.copydatatotv` | 0 → 3 | `pm default-state --user 0 com.tcl.copydatatotv` |
| Supported USB microphone / karaoke capture, playback mixing, gain and reverb | `com.tcl.micmanager` | 0 → 3 | `pm default-state --user 0 com.tcl.micmanager` |
| Play auto-install configuration stub; effects on future setup / Play behavior untested | `android.autoinstalls.config.tcl.device` | 0 → 3 | `pm default-state --user 0 android.autoinstalls.config.tcl.device` |

Disable only unused features. MicManager controls audio mixing; verify normal audio after changing it. The Play configuration stub is not an identified background worker, and no speedup was measured for these additions.

For one selected package from this table, record its own User 0 state before changing it:

```sh
adb -s '<TV_IP>:5555' shell dumpsys package '<PACKAGE_FROM_THIS_TABLE>'
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 '<PACKAGE_FROM_THIS_TABLE>'
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
# Rollback only that package to its recorded original default state (0):
adb -s '<TV_IP>:5555' shell pm default-state --user 0 '<PACKAGE_FROM_THIS_TABLE>'
```

Alternatively, prepend `adb -s '<TV_IP>:5555' shell` to the exact rollback command in the table. Do not substitute `pm enable` or restore all disabled packages. This unit has 50 disabled packages: these three additions plus 47 preexisting disables, all preserved. This is not a revised total for both historical units.

Home selection, keyboard, accessibility, input-filter state, screensaver setting and Netflix AppOp are unchanged by these additions. The [preserved services](#exceptions-and-preserved-services) remain outside this disable list. See [verification limits](#verification-and-limits) before applying these firmware-specific changes elsewhere.

## Additional settings

Both units have `secure screensaver_enabled=0`, originally `1`; the selected screensaver still references Backdrop. Turn the setting off before disabling its providers. Both also have Netflix's `RUN_ANY_IN_BACKGROUND` AppOp set to `deny`, originally no explicit override (effective allow).

For rollback, restore the selected Backdrop package to its recorded default state before re-enabling the screensaver setting. This restores one of the disabled packages as a prerequisite, not every screensaver or unrelated package.

```sh
# Inspect first and keep your own original values.
adb -s '<TV_IP>:5555' shell settings --user 0 get secure screensaver_enabled
adb -s '<TV_IP>:5555' shell cmd appops get --user 0 com.netflix.ninja RUN_ANY_IN_BACKGROUND
# Apply the recorded settings:
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 0
adb -s '<TV_IP>:5555' shell cmd appops set --user 0 com.netflix.ninja RUN_ANY_IN_BACKGROUND deny
# Rollback: restore the selected provider before enabling the screensaver.
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.backdrop
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 1
adb -s '<TV_IP>:5555' shell cmd appops set --user 0 com.netflix.ninja RUN_ANY_IN_BACKGROUND default
```

AppOps do not guarantee the absence of cached or other background processes. Do not substitute a global AppOp reset for this single-operation rollback.

## Exceptions and preserved services

- A disabled `com.tcl.smartalexa` package state does not establish that its persistent process is absent. No uninstall is part of these instructions.
- `com.tcl.usercenter` remains enabled and is not part of the rollback list; the changelog records its earlier disable reversal.
- Keep `com.tcl.guard` outside this disable list: it bundles maintenance, camera/microphone privacy indications, storage warnings, app-launch controls and reporting. No Guard disable was applied.
- Keep `com.tcl.assistant` and `com.tcl.xian.StartandroidService` outside this disable list. They supply callable TV-control integration and device/account activation with a credential provider, respectively, rather than isolated advertising functions.
- Core TV/input, settings, configuration, update, audio, DRM, Wi-Fi, Bluetooth, keyboard and physical remote services were preserved. Examples include `com.tcl.tvinput`, `com.tcl.settings`, `com.android.tv.settings`, `com.tcl.providers.config`, `com.tcl.versionUpdateApp`, `com.tcl.systemserver`, and `com.tcl.keycustomfunctionservice`.

No root access, package uninstall, app-data clear, or bootloader changes were used for this cleanup.

## Verification and limits

For the single-unit configuration with the three additional packages disabled, the user confirmed video and sound, HDMI, physical Home and volume buttons worked. This is user-reported functional validation, not an independent playback measurement or a check of every streaming service, HDMI port or remote function. Reboot/wake persistence after these additions remains unverified.

The [changelog](CHANGELOG.md) records earlier unit-specific persistence checks and ADB observations. Those checkpoints do not validate the later additions. Verify the TV functions you use after each selected change, and test reboot/wake behavior separately.

Useful read-only checks after a selected change:

```sh
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
adb -s '<TV_IP>:5555' shell dumpsys package com.google.android.feedback
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief --user 0 -a android.intent.action.MAIN -c android.intent.category.HOME
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
