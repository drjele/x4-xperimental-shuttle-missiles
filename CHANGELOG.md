# Changelog

All notable changes to this project will be documented in this file.

## [v1.1.0] - 2026-09-28 - Verified in game, savegame warning

### Changed

- `save="1"` in the manifest. The mod is recorded in the savegame, so the game warns before loading one without it. The savegame still loads without the mod; without the warning a load with the mod disabled, uninstalled or mid-switch to the Workshop copy silently cost the ship its second launcher.
- Detailed Steam Workshop description, covering every setting, where to change it, and the required and optional dependencies, and a note on the launcher lost to a load without the mod and how to fit it again.
- Extension id shortened to `drjele_xperimental_missiles`. The Steam Workshop allows at most 32 characters for a folder name and the old id was 35.

### Added

- `publish.sh update` options `--minor`, `--namedesc` and `--readback`, for an update that leaves the version number alone, one that also pushes the name and description to Steam, and one that writes Steam's own text back into `content.xml.steam`.

### Fixed

- `publish.sh` passes `-batchmode`, so an upload no longer waits for a keypress a non-interactive run cannot give it.
- `publish.sh` runs the workshop tool inside the Steam snap's mount namespace. The snap has a private `/tmp`, so the Steam client IPC the tool needs is unreachable from outside it and the upload failed on a Steamworks assertion.
- `publish.sh` shows the tool's output, which Proton otherwise discards, and reads success or failure out of it rather than out of an exit code Proton does not pass on.
- `publish.sh` restores the local installation even when the upload fails, instead of leaving it holding the staged copy with the Workshop id in it.
- `publish.sh update` sends the preview image too, so a refreshed `extension/preview.jpg` reaches the Workshop item instead of leaving the one from the first upload in place.

### Notes

- Verified in a running game, 9.00 build 611726: a ship that already existed in the savegame picks up the second mount, a launcher fitted into it is saved on `con_missilelauncher_02`, and both launchers fire in the same volley.
- Nothing is visible on either side, in vanilla too: the mk1 small launchers share the component `weapon_gen_s_missile_01`, which has no mesh.
- A load without the mod silently drops the launcher on the second mount and keeps the missiles. Recovery is a wharf visit: the second slot is empty again, fit a launcher into it.

## [v1.0.0] - 2026-09-07 - Initial release

### Added

- A second missile mount, `con_missilelauncher_02`, mirrored onto the side of the Experimental Shuttle that has none, at `x = +3.66873` against the vanilla mount's `-3.66873`. The ship model is untouched: the vanilla mount carries no `<parts>`, so a mount is a bare point in space and the visible hardware comes from the launcher's own macro.
- A ship missile magazine of 80, where vanilla declares 0. With the two mounts the total is 100 missiles, against the 10 of the stock ship.

### Notes

- Missile capacity is the sum of the ship's magazine and the capacity of every launcher fitted, so the second mount is worth 10 on its own.
- The magazine has no in-game menu and cannot have one: it is a macro property with no Mission Director action that writes it. The number is one line in the macro patch, re-read on every savegame load, so changing it needs no new game.
- Requires the Timelines DLC, which is where the ship exists.
- Both patches are filed under `extensions/ego_dlc_timelines/` inside the extension, mirroring the path `index/components.xml` and `index/macros.xml` resolve to. A patch at the plain `assets/...` path targets a base-game file that does not exist for a DLC-only ship, and is silently applied to nothing.

[v1.1.0]: https://github.com/drjele/x4-xperimental-shuttle-missiles/releases/tag/v1.1.0
[v1.0.0]: https://github.com/drjele/x4-xperimental-shuttle-missiles/releases/tag/v1.0.0
