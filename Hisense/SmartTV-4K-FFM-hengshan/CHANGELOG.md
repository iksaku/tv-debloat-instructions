# Hisense SmartTV 4K FFM / hengshan changelog

Changes and discoveries for Android 12 / firmware `00.00.00.132` in Basic TV mode. Use the [cleanup reference](notes.md) for package names, commands and rollback.

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
- Default Home replacement was not achieved on this firmware. Disabling stock Home exposed Setup Wizard's Recovery activity rather than the replacement. Restored stock Home and removed the replacement for user 0; Setup Wizard remained unchanged.
