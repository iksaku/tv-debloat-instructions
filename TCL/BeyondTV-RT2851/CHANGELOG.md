# TCL BeyondTV / RT2851 changelog

Changes and discoveries for the identified Android 11 / `V8-R851T02-LF1V715` (`AR05`) platform. Use the [cleanup reference](notes.md) for package names, commands and rollback. Single-unit observations below are not results for both historical units.

## 2026-10-20 — Reference cleanup and screensaver rollback order

- Moved the older two-unit investigation and launch-check history into this changelog, keeping current package states, feature tradeoffs, safety boundaries and rollback in the reference. Preserved the existing launcher and factory-menu instructions. Recorded the three additions as single-unit results rather than extending them to both historical units.
- Corrected the documented screensaver rollback order: restore the selected Backdrop package before enabling its setting. Re-read the repository's note-writing guidance after pulling `main`, which was already up to date. This was a documentation-only update; no TV commands were executed.

## 2026-10-20 — User-confirmed playback and remote behavior

- The user confirmed video and sound, HDMI, physical Home and volume buttons worked with the three additional packages disabled. This is user-reported functional validation, separate from the earlier ADB navigation and service checks; it does not establish every streaming service, HDMI port or remote function.
- Updated the reference's validation limits. Reboot/wake persistence remains unverified; Guard remains unchanged. This documentation update made no TV changes.

## 2026-10-20 — Three additional optional packages disabled

- After the user authorized trying the recommendations, disabled the retail-demo copier, USB microphone/karaoke integration and Play auto-install configuration stub one at a time. All three changed from default (`0`) to disabled-user (`3`) for user 0. The reviewed unit now has 50 disabled and 87 enabled packages; all 47 preexisting disables and unrelated package/component states were preserved against the fresh pre-change baseline.
- MicManager's process was present before its disable and absent in the subsequent samples. No candidate process appeared in the final samples; this is not a measured performance improvement or proof of permanent process suppression.
- ADB remained reachable during the checks. Home still resolved to AT4K; Settings and injected Home navigation reached the expected resumed activities before and after the changes. Keyboard, bound AT4K accessibility, disabled input filter, screensaver setting and Netflix AppOp were unchanged. HDMI ports 1–4 and existing input-registration states remained present, and the sampled crash buffer stayed empty. These observations do not establish playback, audio, physical remote or HDMI functionality.
- No reboot, uninstall, data clear, component override, launcher selection or settings change was performed. At this checkpoint, physical remote, playback/audio, HDMI playback and reboot/wake persistence were unverified; subsequent user confirmation is recorded above. Guard and the lower-priority candidates were not disabled; a Guard trial remains separate from this batch.
- Reconnection completed the earlier inventory checks: enabled/disabled membership was unchanged, but Play Store's package state had changed from explicitly enabled (`1`) in the earlier review to default (`0`) before this batch. The cause was not established; no Play Store state command was issued. All unrelated state-preservation comparisons use the fresh baseline, not a claim that nothing changed while ADB was offline.

## 2026-10-20 — Read-only follow-up to the Hisense cleanup

- Confirmed the connected TCL's platform/firmware and inspected its installed APKs rather than copying Hisense package names. The initial inventory contained 90 enabled and 47 disabled packages, including all 26 common documented disables and Google TV/Movies. Native Home already selected AT4K with input filtering off, so no Setup Wizard or enhanced Home-detection change was proposed.
- Identified `com.tcl.copydatatotv` as a USB retail-demo resource copier and `com.tcl.micmanager` as USB microphone/karaoke mixing with gain/reverb controls, not simply generic far-field voice support. Identified the Play auto-install configuration stub separately from a running service. Calendar storage and the local HTML document viewer remain lower-priority, unapplied candidates; they are not aliases for the Hisense packages.
- Found that Guard bundles maintenance, microphone/camera privacy indicators/history, storage warnings, app-launch policy, app-management helpers and reporting. It remains a possible separately authorized experiment, not an isolated telemetry disable. Assistant implements callable TV/input/channel/picture/audio/power controls; device authentication supplies activation and a credential/token provider. Neither was disabled. Ordinary remote/HDMI dependence on Assistant and external authentication-provider consumers remain unverified.
- User Center remained default-enabled; the earlier disable reversal is still unexplained. Guard's component-enable and launch-AppOps code did not establish package-level reenabling as its cause. No uninstall/root workaround was attempted.
- The agreement UI writes a reporting-related property whose checked/unchecked mapping does not establish the meaning suggested by its name. Downstream effects remain unverified; no property or consent setting was changed. Partial decompiler errors and uninspected native code limit the static findings.
- ADB went offline before the first review's closing snapshots; those empty captures were failed reads, not evidence of removed packages. No device changes were made during that read-only review. The authorized changes after reconnection are recorded above.

## 2026-10-18 — Original two-unit cleanup and persistence checks

These observations were moved from the original reference; they are not new tests.

- Disabled 26 common packages from default (`0`) to disabled-user (`3`) on each unit. Unit A also disabled Google TV/Movies from explicit enabled state (`1`), for 27 task changes; unit B retained that app and had 47 disabled packages: 26 task changes plus 21 preexisting disables. Unit A's total disabled-package count was not used as a task-change count.
- Both units changed the screensaver setting from `1` to `0`, leaving Backdrop selected, and restricted Netflix's background AppOp from no explicit override (effective allow) to `deny`. Netflix still launched afterward; that launch did not establish playback or absence of background processes.
- Unit A's first 25 final disables and both settings survived an AC power cycle. Later PartnerCustomizer/eSticker disables passed package-state, Settings/YouTube/Netflix launch, Home foreground and protected-service checks; no reboot after those two additions was confirmed. Keyboard/accessibility settings were unchanged, but an enabled accessibility service was not necessarily bound. Playback and physical HDMI/audio were not established by those checks.
- Unit B's 26 task disables and 21 preexisting disables persisted through a normal reboot with a changed boot ID and completed boot. Replacement Home, bound accessibility, keyboard, screensaver and Netflix AppOp checks also persisted. Settings, Netflix, YouTube, Disney+, HBO Max, ViX and Google TV/Movies launched; Prime Video timed out initially and passed a bounded recheck. Audio/DRM, network, remote and TV-input processes were present, HDMI ports 1–4 were registered, and the sampled crash buffer had no fatal exceptions. The user confirmed audio/video generally, not every port, DRM path or service's playback.
- `com.tcl.smartalexa` retained a persistent process despite its package disable, including after reboot. Unit A's attempted `com.tcl.usercenter` disable returned to default after reboot; it was left there, and unit B retained it. Neither package was uninstalled. The mechanism restoring User Center was not established.
