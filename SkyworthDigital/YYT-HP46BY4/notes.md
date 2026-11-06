# SkyworthDigital YYT / HP46BY4 platform

Reversible cleanup reference for one user-identified Android projector: ten optional-service disables, screensaver off, and a separately documented AT4K Home cutover with two competing packages disabled. **The Home cutover lost network ADB before final verification; do not treat it as a validated recipe.** YYT / HP46BY4 identifies the reported platform, not a confirmed retail projector model. The firmware's model string is `4K Google TV Stick`; it does not establish the enclosure's manufacturer or retail model. Results apply only to the recorded firmware.

## Device identity

| Property | Recorded value |
| --- | --- |
| Manufacturer / brand | SkyworthDigital / Skyworth |
| ADB model / device | `4K Google TV Stick` / `YYT` |
| Board / platform / hardware | `HP46BY4` / `HP46B` / `amlogic` |
| Android / API | 11 / 30 |
| Build / incremental | `RTT0.210829.002` / `C2.5.3_20250414` |
| Security patch | 2025-04-05 |
| ABI | 32-bit ARM: `armeabi-v7a`, `armeabi` |
| Build fingerprint | `Skyworth/HP4609-Skyworth/YYT:11/RTT0.210829.002/C2.5.3_20250414:user/release-keys` |
| Android user | 0 |

## Connection and baseline

Use authorized local-network ADB on port 5555. Replace `<TV_IP>` with the projector's address and approve the computer on the projector. Confirm the physical target and fingerprint; never expose ADB to the Internet.

```sh
adb connect '<TV_IP>:5555'
adb -s '<TV_IP>:5555' shell getprop ro.product.model
adb -s '<TV_IP>:5555' shell getprop ro.build.fingerprint
adb -s '<TV_IP>:5555' shell am get-current-user
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
adb -s '<TV_IP>:5555' shell cmd package resolve-activity --brief --user 0 -a android.intent.action.MAIN -c android.intent.category.HOME
```

The only preexisting disabled package was `com.google.android.tv.axel`, at disabled-user state (`3`); it was left unchanged. All ten packages below changed from manifest/default (`0`) to disabled-user (`3`) for user 0. Explicit enabled state (`1`) is different from default state. Preserve different original states and component overrides on other devices. For updated system apps, read the active package's User 0 state, not the separate hidden factory-copy record in a full package dump.

## Disabled packages and tradeoffs

Disabling retains installed packages and their data. Turn off the [screensaver setting](#screensaver-setting) before disabling its providers. These are optional feature removals, not evidence of measured performance gains.

| Feature removed or affected | Exact package | Recorded state |
| --- | --- | --- |
| Google feedback reporting interface | `com.google.android.feedback` | 0 → 3 |
| Android printing | `com.android.printspooler` | 0 → 3 |
| Google Calendar synchronization | `com.google.android.syncadapters.calendar` | 0 → 3 |
| Play Games app / integrations | `com.google.android.play.games` | 0 → 3 |
| Basic alternative screensavers | `com.android.dreams.basic` | 0 → 3 |
| Google TV Ambient Mode / Backdrop and associated features | `com.google.android.apps.tv.dreamx` | 0 → 3 |
| Google Cast receiver | `com.google.android.apps.mediashell` | 0 → 3 |
| Google search / voice assistant | `com.google.android.katniss` | 0 → 3 |
| TalkBack and bundled accessibility features | `com.google.android.marvin.talkback` | 0 → 3 |
| Speech synthesis and potentially recognition / voice-input integrations | `com.google.android.tts` | 0 → 3 |

For one selected package from the table, inspect and record its original state before applying a disable. The rollback shown restores the recorded default state on this unit, not explicit enabled state:

```sh
adb -s '<TV_IP>:5555' shell dumpsys package '<PACKAGE_FROM_TABLE>'
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 '<PACKAGE_FROM_TABLE>'
adb -s '<TV_IP>:5555' shell pm list packages -d --user 0
# Rollback for a package in this unit's table: original state was 0.
adb -s '<TV_IP>:5555' shell pm default-state --user 0 '<PACKAGE_FROM_TABLE>'
```

Restore only selected cleanup changes. Do not enable every disabled package or alter the preexisting Axel disable as part of rollback.

Restoring speech or accessibility packages does not by itself select their engine/services. Review those settings separately when restoring their features.

## Screensaver setting

`secure screensaver_enabled` changed **1 → 0**, before either screensaver provider was disabled. The selected component remains `com.google.android.apps.tv.dreamx/.service.Backdrop`.

```sh
adb -s '<TV_IP>:5555' shell settings --user 0 get secure screensaver_enabled
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 0
```

Restore the selected provider before re-enabling the setting:

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.apps.tv.dreamx
adb -s '<TV_IP>:5555' shell settings --user 0 put secure screensaver_enabled 1
```

No other settings or AppOps were changed.

## Installed apps

Both apps were absent before installation for user 0. The reused installers matched the SHA-256 values recorded from official releases:

| App / official release | Installed version | Package / activity | Compatibility |
| --- | --- | --- | --- |
| [AT4K v1.3](https://github.com/avadhesh18/at4k/releases/tag/v1.3) | `1.3`, code 20 | `com.overdevs.at4k/.MainActivity` | minSdk 27; includes 32-bit ARM |
| [Wholphin v1.1.0](https://github.com/damontecres/Wholphin/releases/tag/v1.1.0) | `1.1.0-0-g2bb662fd`, code 61 | `com.github.damontecres.wholphin/.MainActivity` | minSdk 23; `armeabi-v7a` |

- AT4K SHA-256: `1bb6dad64f26e8d82d65f5a02f76327affb24b825f4d8d3f99c4d56d6fab77a3`
- Wholphin SHA-256: `1096e30eaacf4add580e8991fb060fd95b3dce63a9fb0f5fd0cbb42b0419639a`

With the verified APKs in the local current directory:

```sh
adb -s '<TV_IP>:5555' push AT4K-v1.3.apk /data/local/tmp/AT4K-v1.3.apk
adb -s '<TV_IP>:5555' shell pm install --user 0 /data/local/tmp/AT4K-v1.3.apk
adb -s '<TV_IP>:5555' push Wholphin-v1.1.0-armeabi-v7a.apk /data/local/tmp/Wholphin-v1.1.0-armeabi-v7a.apk
adb -s '<TV_IP>:5555' shell pm install --user 0 /data/local/tmp/Wholphin-v1.1.0-armeabi-v7a.apk
adb -s '<TV_IP>:5555' shell rm /data/local/tmp/AT4K-v1.3.apk /data/local/tmp/Wholphin-v1.1.0-armeabi-v7a.apk
adb -s '<TV_IP>:5555' shell am start --user 0 -W -n com.overdevs.at4k/.MainActivity
adb -s '<TV_IP>:5555' shell am start --user 0 -W -n com.github.damontecres.wholphin/.MainActivity
```

For existing installations, review signature/version compatibility and use `pm install -r --user 0` to preserve data. Do not uninstall or clear data as an update workaround.

AT4K displayed its app interface; its read-TV-channel-list permission was accepted. Wholphin displayed server selection and still needs a Jellyfin server/account; playback remains unverified.

Leave the apps installed when reverting package disables; removal is not required and uninstalling would discard their app data. Home selection and competing-package changes are separate from installation.

### Optional AT4K accessibility

AT4K's service `com.overdevs.at4k/com.overdevs.at4k.Hra` was present in the enabled accessibility-service setting before Home cutover, following manual user setup. No ADB accessibility setting write was performed. Disabling TalkBack does not disable Android's accessibility framework. Accessibility interception is separate from native default Home routing; the Home resolver still chose stock Home with this service enabled. To undo the manual accessibility approval, turn AT4K off in Settings → Accessibility. No app-data clear is needed.

### Home cutover and recovery

**Not functionally verified after both disables.** Stock Google TV Home and Setup Wizard each started at default state (`0`); their disable commands returned disabled-user (`3`). Selecting AT4K returned success. Disabling only stock Home exposed `com.google.android.tungsten.setupwraith/.RecoveryActivity` in both the resolver and foreground, even after repeating Home selection.

Setup Wizard handles setup/recovery and may own network/settings entry points, not just a competing Home activity. After its user-requested disable, ADB went offline and reconnection timed out. Causality and the final Home result are unverified. Ensure physical-remote access to network/developer settings and a local recovery path before attempting these changes; do not assume another platform's successful cutover applies here.

Recorded commands, after visually confirming AT4K was usable:

```sh
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 com.overdevs.at4k/.MainActivity
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.google.android.apps.tv.launcherx
# Stock-only disable routed Home to Setup Wizard Recovery on this firmware.
# The following was explicitly requested despite setup/network tradeoffs:
adb -s '<TV_IP>:5555' shell pm disable-user --user 0 com.google.android.tungsten.setupwraith
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 com.overdevs.at4k/.MainActivity
```

Once network ADB is available, verify the Home resolver, navigate from Settings using Home, inspect the actual screen, and test physical remote buttons and connectivity. Reboot/wake persistence needs separate validation.

Rollback restores the original default states before selecting stock Home:

```sh
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.tungsten.setupwraith
adb -s '<TV_IP>:5555' shell pm default-state --user 0 com.google.android.apps.tv.launcherx
adb -s '<TV_IP>:5555' shell cmd package set-home-activity --user 0 com.google.android.apps.tv.launcherx/.home.HomeActivity
adb -s '<TV_IP>:5555' shell am start --user 0 -W -n com.google.android.apps.tv.launcherx/.home.HomeActivity
```

If AT4K accessibility or its Home override redirects navigation during rollback, turn that interception off manually. If network ADB is unavailable, restore network/developer access locally first; these commands cannot repair an unreachable connection. No factory reset or app-data clear is part of rollback.

## Preserved features and safety constraints

- Stock Home and Setup Wizard are the two separately requested Home-cutover disables above; final AT4K Home behavior remains unverified.
- Phone-remote integration (`com.google.android.tv.remote.service`) remains unchanged. Cast, Google search/voice, TalkBack and speech services are disabled as listed above; the physical Bluetooth pairing packages remain outside the disable set.
- All existing streaming and user apps remain unchanged, including Netflix, YouTube, Prime Video, Disney+, Crunchyroll, Mercado Play, Tubi, Max, Apple TV, Paramount+, Jellyfin, and Send Files to TV.
- Magis was explicitly preserved. This unit has `com.xtv.x1x` version `4.99.2`, exposing `com.android.mgstv.*` provider authorities; the literal package `com.android.mgstv` is absent. Its user 0 default state (`0`), version, data inode, permissions and component overrides matched before/after installation. No disable, update, uninstall or data clear was performed on it.
- Keep projector/OEM packages outside this disable set: `com.sdt.projector.observer`, `com.sdt.producttest`, `com.oemservice`, `com.oem.qa`, `com.mft.frontpanelledsservice`, `com.mft.rcupair`, `com.droidlogic`, and `com.droidlogic.googletv.settings`. Observed components include a projector `com.sdt.hdmi.MyAccessibilityService`, pairing activity/boot receiver, LED boot receiver, and product-test key-check/debug screens plus a remote-data provider. Those integrations are not proof of expendability; their full responsibilities are not established.
- Preserve Settings, System UI, Google Play/Play Services/GSF, keyboard, networking, Bluetooth, package management, shared WebView, audio/DRM, and firmware/remote update pathways. Setup Wizard is the explicit exception described above; other provisioning packages remain unchanged. `com.google.android.tv.dfuservice` and `android.autoinstalls.config.google.gtvpai` were not changed.
- The recorded keyboard is `com.google.android.inputmethod.latin/com.android.inputmethod.latin.LatinIME`. AT4K accessibility was manually enabled; its setting was not altered by the Home-cutover commands.

No root, uninstall, app-data clear, factory reset, firmware flash or reboot was performed.

## Verification and limits

Before Home cutover, the active-package comparison confirmed exactly the five selected service changes, all `0 → 3`, with ten cleanup services plus the preexisting Axel disable. Magis metadata/state was unchanged, screensaver was off, and Settings and stock Home were inspected. The two later competing-Home disable commands returned success, but no final package inventory, Home resolver, actual Home screen or network-settings check could be obtained after ADB went offline. The expected total is thirteen disabled packages if no other states changed; that total is not a final device readback.

These checks do not establish playback, sound, physical remote operation, projector focus/keystone controls, HDMI/CEC, firmware updating, or persistence after reboot/wake/AC loss. Those functions remain unverified. Cast, Google search/voice, TalkBack, speech services and Ambient Mode are deliberately unavailable in this configuration. No measured speedup or complete ad/tracking removal is claimed. See [CHANGELOG.md](CHANGELOG.md) for dated findings.
