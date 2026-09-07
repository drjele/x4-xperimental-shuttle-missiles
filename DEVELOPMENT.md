# Development

## Checks and formatting

Use Python 3.10 or newer and Bash. Install the pinned tools in a virtual environment:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements-dev.txt
export PATH="$PWD/.venv/bin:$PATH"
python3 scripts/check.py
```

Run `python3 scripts/check.py --fix` to format Python and shell and normalize text whitespace. The same checks run on pushes and pull requests. XML is checked for well-formedness; game schemas, XPath matches and gameplay require separate X4 validation. Blender scripts are parsed and linted without importing Blender.

Use UTF-8, LF, a final newline, spaces and no trailing whitespace. Indent code with four spaces and workflow YAML with two. Use descriptive names, uppercase shell variables, constant-first equality comparisons and explicit boolean checks. Ruff's E712 rule is disabled to retain explicit boolean comparisons. Keep shell free of prose comments. Keep only short, non-obvious constraints in code; put explanations here. XML continuation attributes may align with their opening attribute. Preserve XPath selectors, savegame identifiers and embedded game expressions when applying formatting.

## Installation and publishing helpers

`install.sh` and `publish.sh` both source `lib/find_x4.sh`. The library searches usual Steam roots and additional library folders. `X4_PATH`, `X_TOOLS_PATH`
and `PROTON_PATH` override discovery. Proton Experimental is preferred when found; otherwise the helper uses the last matching Proton directory it encounters.

Installation replaces the extension directory with a copy of `extension/`. Refresh it after edits; X4 enumerates real extension directories, so a symlink does not substitute for installation. Restart the game after installing or removing.

Publishing stages a separate copy inside the game's extensions directory. The repository keeps its readable extension id; `steam/workshop-id` holds the numeric Workshop id. The helper changes only the staged manifest, runs the interactive WorkshopTool and restores the manual installation after success. On Linux it runs WorkshopTool through Proton and maps paths through drive Z. A failed upload can leave the staged copy behind; rerun `./install.sh` to restore it.

## Release metadata

`content.xml` uses an integer version multiplied by 100 and an ISO release date. The date matches the corresponding released entry in `CHANGELOG.md`. Development changes belong under `Unreleased`; they do not advance the manifest's release version or date. An unreleased scaffold may retain its initial creation date until its first release. Keep existing extension ids stable.

## Implementation constraints

### extension/extensions/ego_dlc_timelines/assets/units/size_s/ship_ter_s_xperimental_01.xml

Both patches live under extensions/ego_dlc_timelines/ inside this extension, mirroring the DLC's own path. That is not decoration: index/components.xml and index/macros.xml in Timelines resolve the component and the macro to extensions\ego_dlc_timelines\assets\units\size_s\..., so a patch at the plain assets/ path targets a base-game file that does not exist for a DLC-only ship, and is applied to nothing. The failure is silent - X4 logs a selector that matches nothing, but says nothing at all about a patch aimed at a missing file. Shipped that way once and it did nothing.

Adds the second missile mount. A mount is not geometry: the vanilla con_missilelauncher_01 is a bare &lt;connection&gt; with an &lt;offset&gt; and no &lt;parts&gt;, unlike the thruster, elevator and seat connections in the same file, which all name a mesh. The visible launcher comes from the equipped launcher macro, instantiated at that point - which is why a second mount needs no change to the ship model and no Blender at all.

The mirrored coordinate is read off the gun mounts: con_weapon_01..03 are tagged symmetry_left at negative x, con_weapon_04..06 symmetry_right at positive x. The vanilla launcher sits at x=-3.66873, so its opposite is x=+3.66873 with y and z unchanged.

No symmetry tags on the new mount, matching the vanilla launcher, which carries none either. Adding them would pair the two in the UI; leaving them off keeps the slots independent, which is what allows a different launcher type on each side.

The story macro ship_ter_s_xperimental_01_a_story_macro references the same component, so it gets the mount too.

### extension/extensions/ego_dlc_timelines/assets/units/size_s/macros/ship_ter_s_xperimental_01_a_macro.xml

The ship's own missile magazine, vanilla 0.

Total capacity is the ship's magazine plus the capacity of every launcher and turret fitted - menu_ship_configuration.lua walks the weapon and turret slots and adds each slot macro's capacity to the base. The stock ten missiles therefore come entirely from weapon_gen_s_guided_02_mk1_macro, which declares &lt;storage capacity="10"/&gt;. With two mounts the launchers contribute 20, so the 80 here totals 100.

This value cannot be exposed as an in-game option, and that is a property of the game rather than a shortcut. A magazine is a macro property, read from file at savegame load; there is no MD action that writes one. set_ammo, add_ammo, remove_ammo, deplete_ammo and evaluate_missile_storage all move ammo inside the existing capacity, and add_ammo's documented "storage limits are ignored" applies to unit macros only. The one runtime lever the game has is an equipment mod - equipmentmods.xml defines &lt;missilecapacity ware="mod_ship_missilecapacity_01_mk1" min="1" max="4"/&gt; and add_equipment_mods installs it - but the value is rolled randomly inside the ware's own min/max and it occupies one of the ship's three mod slots, so it is not a usable substitute for a setting.

Macros are re-read on every savegame load, so editing the number and reloading is enough - no new game, and it reaches ships that already exist.

### Verifying the patches offline

Neither patch can be checked by scripts/check.py, which only tests XML well-formedness. What matters for a diff is whether its selector matches, so apply both against the shipped files before shipping: extract assets/units/size_s/ship_ter_s_xperimental_01.xml and assets/units/size_s/macros/ship_ter_s_xperimental_01_a_macro.xml from ego_dlc_timelines/ext_01.cat, run each &lt;add&gt; and &lt;replace&gt; selector with lxml, and assert exactly one match. Note that this only proves the selectors are right - it says nothing about whether the patch is filed at the path the game resolves, which is the other half and the one that failed first time. Both matched one node each against 9.00 build 611726. In game the same failure shows up as an XPath-matched-nothing line in the debug log at startup.
