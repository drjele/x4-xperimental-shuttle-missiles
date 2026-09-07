# Changelog

All notable changes to this project will be documented in this file.

## [v1.0.0] - 2026-09-07 - Initial release

### Added

- A second missile mount, `con_missilelauncher_02`, mirrored onto the side of the Experimental Shuttle that has none, at `x = +3.66873` against the vanilla mount's `-3.66873`. The ship model is untouched: the vanilla mount carries no `<parts>`, so a mount is a bare point in space and the visible hardware comes from the launcher's own macro.
- A ship missile magazine of 80, where vanilla declares 0. With the two mounts the total is 100 missiles, against the 10 of the stock ship.

### Notes

- Missile capacity is the sum of the ship's magazine and the capacity of every launcher fitted, so the second mount is worth 10 on its own.
- The magazine has no in-game menu and cannot have one: it is a macro property with no Mission Director action that writes it. The number is one line in the macro patch, re-read on every savegame load, so changing it needs no new game.
- Requires the Timelines DLC, which is where the ship exists.
- Both patches are filed under `extensions/ego_dlc_timelines/` inside the extension, mirroring the path `index/components.xml` and `index/macros.xml` resolve to. A patch at the plain `assets/...` path targets a base-game file that does not exist for a DLC-only ship, and is silently applied to nothing.

[v1.0.0]: https://github.com/drjele/x4-xperimental-shuttle-missiles/releases/tag/v1.0.0
