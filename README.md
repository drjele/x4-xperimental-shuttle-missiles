# Experimental Shuttle Missiles for X4: Foundations

<p align="center">
  <img src="extension/preview.jpg" alt="Experimental Shuttle Missiles" width="512">
</p>

The Experimental Shuttle carries one missile launcher, on one side, and ten missiles. The ten are not the ship's — its own magazine is zero. Every missile it carries comes from the launcher.

This mod mirrors the launcher onto the empty side and gives the ship a magazine of its own. Two mounts and a magazine of 80 make 100 missiles, against the 10 the stock ship gets.

**The ship model is not touched.** It does not need to be, and that is the whole trick — see below.

**Requires X4: Foundations 9.00 and the Timelines DLC**, which is where the ship lives. Works on an existing savegame.

## Install

```bash
./install.sh
```

The helper copies `extension/` into the game's `extensions/<extension-id>`
directory, using the id in `extension/content.xml`. It searches the usual Steam layouts and additional library folders. To choose an installation:

```bash
X4_PATH="/path/to/X4 Foundations" ./install.sh
```

Restart X4 after installing or updating. To remove the manual installation:

```bash
./install.sh --uninstall
```

## What it changes

|                    | Stock  | With this mod |
|--------------------|--------|---------------|
| Missile mounts     | 1      | 2             |
| Ship magazine      | 0      | 80            |
| **Total missiles** | **10** | **100**       |

Both mounts take any small missile launcher, independently — you can put a guided launcher on one and a dumbfire on the other.

## Why the model does not need touching

A missile mount in X4 is not geometry. This is the whole of the vanilla one, in the Timelines copy of `assets/units/size_s/ship_ter_s_xperimental_01.xml`:

```xml

<connection name="con_missilelauncher_01" tags="advanced missile small weapon ">
    <offset>
        <position x="-3.66873" y="1.000061" z="6.772886"/>
    </offset>
</connection>
```

No `<parts>`, no mesh reference — a bare point in space. Every other connection in that file that *does* carry geometry (thrusters, the elevator, the pilot seat) has a `<parts>` block naming a mesh. The launcher has none, because the visible hardware comes from the launcher's own macro, instantiated at that point.

So the second mount is one more `<connection>` at the mirrored coordinate. Positive x is the opposite side: the ship's six gun mounts are tagged `symmetry_left` at negative x and `symmetry_right` at positive x, so `x = +3.66873` with the same y and z puts the new launcher exactly opposite the old one.

## Where the missile capacity comes from

Not from the ship. `ship_ter_s_xperimental_01_a_macro` declares `<storage missile="0" />`. The launcher macro declares the capacity instead:

```xml

<macro name="weapon_gen_s_guided_02_mk1_macro" class="missilelauncher">
    <storage capacity="10"/>
```

and the game adds the two together. `menu_ship_configuration.lua` walks every weapon and turret slot and adds that slot's macro capacity to the ship's base:

```lua
capacity = capacity + C.GetMacroMissileCapacity(data.macro)
```

So `total = ship magazine + the capacity of every launcher fitted`. The second mount is worth 10 on its own; the 80 in the magazine is what takes it to 100.

For scale: of the 94 S-class ship macros in a full install, only 26 have any magazine at all. The median is 4, the highest is 20, and the closest Terran comparison — the Katana — has 6.

## Changing the number

There is no in-game menu, and there cannot be one. A missile magazine is a macro property, read from file when the savegame loads, and the Mission Director has no action that writes it — `set_ammo`, `add_ammo` and `evaluate_missile_storage` all move ammo around inside the existing capacity. The only runtime lever the game offers is an equipment mod (`mod_ship_missilecapacity_01_mk1`), which rolls a random 1 to 4 and occupies one of the ship's three mod slots.

So the number lives in the patch. One line, in `extension/extensions/ego_dlc_timelines/assets/units/size_s/macros/ship_ter_s_xperimental_01_a_macro.xml`:

```xml

<replace sel="//macro[@name='ship_ter_s_xperimental_01_a_macro']/properties/storage/@missile">80</replace>
```

Change it, re-run `./install.sh`, reload your save. Macros are re-read on every load, so **no new game is needed** and the change reaches ships that already exist. `0` restores the vanilla magazine and leaves only the 20 the two launchers provide.

To keep the second mount but nothing else, delete the macro patch file. To keep the magazine but not the second mount, delete the component patch file.

## Why both patches live under `extensions/ego_dlc_timelines/`

Because that is where the game looks. Timelines' `index/components.xml` maps the component to a path that carries the DLC prefix:

```xml

<entry name="ship_ter_s_xperimental_01"
       value="extensions\ego_dlc_timelines\assets\units\size_s\ship_ter_s_xperimental_01"/>
```

and `index/macros.xml` does the same for the macro. A patch placed at the plain `assets/units/...` path inside a mod therefore targets a **base game** file — which for a DLC-only ship does not exist. The patch is then applied to nothing.

The failure mode is nastier than it sounds: X4 reports a selector that matches nothing (`No matching node for path ... in patch file`), but a patch aimed at a file that is not there produces **no message at all**. The mod loads, the log is clean, and nothing happens. If you patch DLC content, mirror the DLC's path inside your extension.

## Debugging

Add this to the game's launch options — Steam, right click X4, **Properties → General → Launch Options**:

```
-debug scripts -debug error -logfile debuglog.txt
```

The log lands next to your savegames: `$HOME/.config/EgoSoft/X4/<userid>/debuglog.txt` on Linux, `Documents\Egosoft\X4\<userid>\debuglog.txt` on Windows. If Steam is installed as a snap it runs the game with a redirected home, which puts both under `~/snap/steam/common/`.

This mod runs no scripts and so says nothing. What it can report is a patch that failed to apply: X4 prints an XPath that matched nothing at startup, by file and selector, so `grep -i drjele debuglog.txt` after a start is the check that both patches landed. Silence means they did.

## Status

Both selectors verified against the shipped 9.00 files by applying the patches offline: each matches exactly one node, and the result is two mirrored mounts plus a magazine of 80.

**Not yet verified in a running game.** The first attempt shipped both patches at the plain `assets/...` path and did nothing at all, silently — see the section above; they now sit under `extensions/ego_dlc_timelines/`, which is the path the index actually resolves to.

The open questions are whether a ship that already exists in a savegame picks up a connection that was not there when it was saved, and whether the launcher's own model looks right mirrored onto the other side.

## Publishing to the Steam Workshop

Install **X Tools** (Steam app 282160) and keep Steam running and logged in with an account that owns X4. On Linux, install Proton as well; on Windows, run the helper from Git Bash, MSYS or Cygwin.

```bash
./publish.sh publish
./publish.sh update "what changed"
```

Use `publish` once, then `update` with a change note. `X4_PATH`,
`X_TOOLS_PATH` and `PROTON_PATH` override automatic discovery. The staging location must contain an `extensions` directory.

The first upload records the numeric id in `steam/workshop-id`; retain that file for future updates. The readable id in the repository's `content.xml`
stays unchanged. After publishing, open the printed Workshop URL, complete any required Steam agreement and choose the item's visibility. Avoid keeping both the manual installation and a subscription to the same mod enabled.

Update the manifest version and release date together with `CHANGELOG.md`
when releasing. See [Development](DEVELOPMENT.md) for staging, platform and release conventions.

## Development

See [DEVELOPMENT.md](DEVELOPMENT.md) for setup, code style, validation and release conventions.

## Legal

MIT, see [`LICENSE`](LICENSE). Non-commercial fan project; X4: Foundations belongs to Egosoft GmbH and this project is not affiliated with or endorsed by Egosoft.
