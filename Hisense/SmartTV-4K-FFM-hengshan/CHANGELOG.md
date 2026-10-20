# Hisense SmartTV 4K FFM / hengshan changelog

Changes and discoveries for Android 12 / firmware `00.00.00.132` in Basic TV mode. Use the [cleanup reference](notes.md) for package names, commands and rollback.

## 2026-10-20 — Remote shortcuts with custom Home

- The user reported that the remote's gear/settings button no longer opens the floating Wi-Fi/Bluetooth/general quick-settings panel. The separate Menu-button picture/sound settings, input switcher, number input, Netflix button and favorite shortcut still work.
- YouTube, Prime Video, Disney+ and Tubi quick-access buttons appear to no longer work, according to the user. This reports shortcut behavior, not failure of the apps themselves; these buttons were not independently reproduced through ADB.
- Recorded these tradeoffs in the launcher reference. The responsible package or component has not been isolated by rollback testing; the observations apply to the current configuration, with both Google TV Home and Setup Wizard disabled alongside the earlier cleanup. No TV settings or package states changed for this documentation update.

## 2026-10-20 — Launcher-independent instructions

- Rewrote the launcher reference as install/select, disable both competing Home packages, verify, and rollback steps using the reader's chosen Home component. AT4K remains the tested example, not a required launcher; its recorded accessibility settings are not presented as universal prerequisites.
- Moved earlier investigation details into this changelog and corrected the reference's stale package total and unchanged-package list: 40 packages are disabled, including Google TV Home and Setup Wizard. This documentation update made no TV changes; the next debloat candidates remain proposals only.

## 2026-10-20 — Native Home routing without enhanced key filtering

- The user reported that enhanced detection broke volume-button holds: individual presses worked, but holding a volume button did not repeat. Turned **Enhanced home detection** off; Android now reports its input filter disabled. Left accessibility approval, **Override Current Launcher**, **Override back button exit**, and **Start on boot** unchanged.
- A reversible Google TV-only disable still resolved Home to Setup Wizard Recovery rather than AT4K. Restored Google TV to default before the next test; that trial left the original 38 disabled packages and both packages' component overrides unchanged.
- After discussing possible Wi-Fi/ADB loss, the user explicitly requested disabling both competing packages. Disabled `com.google.android.apps.tv.launcherx` and `com.google.android.tungsten.setupwraith` for user 0, each from default (`0`) to disabled-user (`3`). The disabled total is now 40: the previous 38 plus these two packages. Android Settings and its fallback Home remain enabled.
- The native Home resolver now selects `com.overdevs.at4k/.MainActivity`. An ADB Home press from Settings reached AT4K and stayed there through all eight foreground samples, with enhanced filtering off. Wi-Fi and ADB remained connected during the checks; this does not establish reconnect or reboot persistence.
- The user confirmed that the physical Home and volume-down hold tests worked with this configuration. A subsequent read-only check still resolved native Home to AT4K and reported the input filter disabled; no further TV settings were changed. Wake/reboot behavior and the effects of losing other bundled Google TV/Setup Wizard features remain unverified. No reboot was performed. Package and Home rollback are in the reference.

## 2026-10-20 — AT4K physical Home routing confirmed

- The user confirmed that pressing physical remote Home reached AT4K with enhanced detection enabled. A subsequent read-only ADB check showed `com.overdevs.at4k/.MainActivity` in the foreground and the input filter still enabled. Stock Home and Setup Wizard were enabled at this checkpoint. Reboot persistence and wake behavior were untested; the later volume-hold report and configuration change are recorded above.
- After manual accessibility approval, verified that AT4K's service was enabled and bound. Selected AT4K as the HOME-role holder; the firmware resolver still selects Vanilla Home, with the same candidate priorities recorded below.
- Enabled **Override Current Launcher** and **Enhanced home detection**, both previously off. Left **Start on boot** off and the existing **Override back button exit** option on. Android reports its input filter enabled. AT4K warns that enhanced detection may interfere with volume-button long presses.
- Inspection of the installed APK's accessibility handler found that normal window detection does not list Basic TV's `VanillaModeHomeActivity`; enhanced mode instead filters Home key events. ADB-injected Home still returned to Vanilla, but Android 12's injected-input path bypasses the accessibility input filter. The injected test therefore did not predict the successful physical remote result.
- Android rejected an individual Setup Wizard Recovery component disable with `SecurityException`; it did not change that component. No system launcher package was disabled, no reboot was performed, and the 38-package disabled list is unchanged. Left Settings foreground for a physical remote test rather than risking the sole Wi-Fi/ADB connection by disabling Setup Wizard wholesale.

## 2026-10-20 — AT4K reinstalled

- Reinstalled AT4K Launcher 1.2 (version code `19`, `com.overdevs.at4k`) for user 0 from the APK linked by its official website. Android accepted the install with the same signing identity recorded before removal; no data-clear operation was used.
- At the reinstall checkpoint, stock Vanilla Home remained the default. Home candidates retained priorities Vanilla `2`, Setup Wizard Recovery `1`, AT4K `0`, and Settings fallback `-1000`; reinstalling did not change that ordering.
- Initially left accessibility disabled for manual approval on the TV. The TV-listings permission prompt was not answered by the agent. Reinstallation itself changed no Home preference, stock-launcher/Setup Wizard package state, or existing debloat disable; the subsequent Home configuration is recorded above.

## 2026-10-20 — Optional vendor cleanup

- Disabled 12 optional vendor packages: the three AIOT packages, camera smart functions, SyncPlay discovery, Hisense Home configuration, Calman connector, Ginga, ATSC3 interaction, Device Assistant, storage-prompt UI and the vendor web-page viewer. The cleanup now covers 32 packages; including six preexisting disables, the TV has 38 disabled and 117 enabled packages.
- All 12 changed from default (`0`) to disabled-user (`3`); rollback is `pm default-state --user 0`. No uninstall, data clear or component-override edits were used.
- Preserved all five AirPlay-related packages, including `com.vt.edv.drmp`, and all other package/component states. Stock Home's package state, Home selection and documented replacement instructions remain unchanged.
- No reboot followed this batch. Reboot persistence, physical remote, picture/audio, HDMI and end-to-end AirPlay operation were not tested for these additions.

## 2026-10-20 — Remaining enabled features

- Completed a read-only inventory of all 129 enabled packages; the 26 disabled packages and stock Home remained unchanged. Review candidates are not additional completed disables.
- Identified separate optional IP-control/AIOT, camera-smart-function, SyncPlay discovery and Hisense launcher-content packages. Ginga and ATSC3 interaction are broadcast extras; their dependency on the TV stack still needs testing before disabling.
- Confirmed an AirPlay-key provisioning path in `com.vt.edv.drmp`: its worker checks AirPlay keys and requests key downloads through a native service. Do not assume every package containing “drm” implements general streaming DRM.
- Found reporting and storage maintenance bundled in `com.vt.tv.smartservice`, including a real trim-service implementation. Separately, `com.vt.tv.smartsense` bundles logging with audio-port, TV-input and physical-remote functions; neither is a reporting-only package.
- Distinguished configuration overlays from background apps. A process named `com.mediatek.tis` was present despite its preexisting package disable; the startup mechanism was not established, so package state alone does not prove absence of a same-named process.

## 2026-10-19 — Expanded cleanup

- Disabled 16 more optional packages, bringing the cleanup to 20 package disables. All 16 additions and the screensaver-off setting persisted through a normal reboot; preexisting disabled packages remained untouched.
- Identified `com.vt.hishow` as Family Moment and `com.vt.tv.singlelisten` as BT Music. StoreMode also contains a SyncPlay service, and Google TTS includes speech recognition and voice input—not just spoken output.
- Confirmed that Media Center includes DLNA sharing controls, while `com.vt.tv.mobiledeviceconnectiontips` contains actual Screen Casting/Miracast services and an AirPlay entry screen. Disabled these unused features while retaining the core AirPlay packages.
- Identified `com.vidaa.oemtabcontentprovider` as a source of Hisense content for stock Home. Disabled it without replacing stock Home or disabling Setup Wizard.
- Turned off `secure screensaver_enabled` before disabling DreamX. Rollback restores the package before restoring the setting; the selected screensaver component remains unchanged.
- Found `com.vt.tv.interactive` bound to the core TV service and handling ATSC3 interaction. Left it outside the disable list.

## 2026-10-18 — Initial cleanup and launcher result

- Disabled ACR, Google feedback, Hisense Channels and Alexa; restored YouTube, Prime Video and Disney+ from their earlier disabled states.
- Default Home replacement was not achieved in this earlier attempt. Disabling stock Home exposed Setup Wizard's Recovery activity rather than the replacement. Restored stock Home to default (`0`), preserving component overrides, and removed the replacement for user 0; Setup Wizard remained unchanged. This bounded result was superseded by the successful native-routing configuration above.
- Preferred-Home selection, the then-tested accessibility configuration and one normal reboot did not redirect physical Home. Android rejected individual Vanilla component disabling with `SecurityException`. An explicit `MAIN` + `HOME` launch created a Home-type task but left a permission activity in front; a successful start status was not proof of a visible launcher screen, and the next Home press still returned to Vanilla.
- Vanilla Home and Setup Wizard Recovery were explicitly enabled components, while normal Google TV Home and Setup Wizard's higher-priority Main activity were disabled at baseline. The three recorded provisioning flags were already `1`. No root, reset, provisioning changes or component-security bypass was used; completing full Google TV setup was not tested as a fix.
- After selecting stock Home, a keep-data removal made the replacement unavailable for user 0 and its activity stopped resolving. Retained package metadata remained, but retained data bytes were not independently inspected. No reboot followed that removal; the later reinstall is recorded above.
- External reports informed caution, not a claim that replacement was impossible: the [OpenCore TV migration audit](https://github.com/eponce00/OpenCore-TV/blob/main/docs/DEVICE_PROFILE_MIGRATION_AUDIT.md) reported Recovery takeover and network/ADB loss on a matching platform with unconfirmed firmware; [launcher issue #605](https://github.com/spocky/miproja1/issues/605) described an explicit-Home workaround on a different device/launcher that did not resolve our earlier test.
- [Android 12 Wi-Fi service source](https://android.googlesource.com/platform/packages/modules/Wifi/+/refs/tags/android-mainline-12.0.0_r122/service/java/com/android/server/wifi/WifiServiceImpl.java) contains removal of app-owned network state when a creator app is disabled or removed. Local metadata did not establish Setup Wizard ownership of the saved network. That remained a possible failure mechanism, not a reproduced local loss; the later two-package disable retained Wi-Fi/ADB connectivity during testing. No third-party debloat code was run.
