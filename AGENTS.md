# Writing TV cleanup references

## File roles

- Keep each device's starting-point reference in `<Manufacturer>/<Model-or-platform>/notes.md`. Use the existing manufacturer/platform layout; do not invent a retail model from an ADB identifier.
- Keep dated changes, discoveries and ongoing findings in a sibling `CHANGELOG.md`. Add a short entry for each meaningful update, newest first. Mark unverified findings as unverified; do not turn a hypothesis into a disable instruction.
- Once a change is confirmed, update the reference to describe the resulting configuration and summarize what changed in the changelog. Keep commands and rollback in the reference rather than duplicating them in the changelog.
- Use the root `README.md` as the device index, not an investigation journal.

## What belongs in notes.md

- Write a settled, usable reference: device/firmware identity, connection instructions, disabled services, exact package names, feature tradeoffs, and rollback.
- Include the Android user and explicit ADB target in commands. Use placeholders such as `<TV_IP>` instead of publishing a household address or device serial.
- Record original and resulting package states. For these Android commands, `0` is manifest/default, `1` is explicitly enabled, and `3` is disabled-user. Restore the recorded original state: `pm default-state` and `pm enable` are not interchangeable.
- Explain settings changes separately, with original values and any required order. Restore a screensaver package before re-enabling its screensaver setting, for example.
- Distinguish newly disabled packages from preexisting disables. Never recommend indiscriminately enabling every disabled package as rollback.
- State what a package actually provides, including bundled features. An installed package, a suggestive name, or a running process alone does not establish that it is bloat or safe to disable.
- Keep essential exclusions and safety constraints. Results belong to the identified firmware; do not claim universal compatibility, complete ad/tracking removal, or measured speedups without evidence.

## What does not belong in the reference

- Do not frame the current configuration as an ongoing phase, partial investigation, task checklist, or chronological session report.
- Put new discoveries, changed decisions and unresolved findings in `CHANGELOG.md`, not in the operational instructions.
- Omit comparisons with other debloat lists and narratives such as "apps opened successfully." App launch and process-presence checks are not playback, audio, remote or HDMI tests.
- Keep raw inventories, device dumps, screenshots, copied APKs and verbose test logs out of tracked notes. Store local review artifacts under `.amp/in/artifacts/` and scratch evidence elsewhere under `.amp/in/`; ensure `/.amp/in/` is in the repository-local Git exclude file first.

## Scope and validation

- Apply these conventions to the sections being edited. Preserve unrelated sections, especially launcher instructions, unless the user explicitly asks to change them; do not rewrite settled instructions merely to impose this format.
- Check package names, state transitions and rollback against recorded device evidence. Do not list an absent package as a completed disable or guess package aliases across firmware versions.
- Keep reviewed candidates distinct from applied changes. A request to inspect or document a TV does not authorize disabling services, installing launchers, rebooting, or changing settings.
- Before finishing, reread edited files, run `git diff --check`, validate local links/anchors, and syntax-check shell examples without executing them on a TV. Preserve the user's existing edits.
