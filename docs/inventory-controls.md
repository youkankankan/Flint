# Inventory administration and crash navigation

## Install the updated Bridge

Live inventory editing requires FlintBridge 0.3.0 or newer. Editing logged-out
players while Minecraft is running requires FlintBridge 0.5.0.
Non-destructive player effects (lightning, hearts, celebration particles) require
FlintBridge 0.6.0. They are online-only, console-only actions addressed by UUID;
Core validates the action and the Bridge checks the player again on the main thread.
Lightning uses Bukkit's effect-only API, not a damaging lightning strike.
The source of the updated plugin now lives in `bridge/` so it can be maintained alongside Core.

Build with a JDK 17 or newer and Maven:

```powershell
mvn -f bridge/pom.xml package
```

Stop Minecraft normally before replacing `plugins/FlintBridge.jar` with
`bridge/target/FlintBridge.jar`. Keep a copy of the previous jar, then start
the server normally. Do not use Bukkit `/reload` or replace a loaded plugin.
No running server or deployed plugin is modified by the build.

## Boundaries and safety

- GUI calls ServerSession commands. Core validates and routes `inventory-edit`.
- HTTP is read-only monitoring. It returns items, a revision, and edit availability.
- Edits use one console-only RCON command, `flint-inventory`. No RCON polling.
- The plugin checks revision and executes the complete change on the server thread.
- Changed inventories, dead players, open containers, crafting contents,
  and cursor items prevent editing. Inventory history is always read-only.
- Move is a whole-stack move or swap, not a split or merge. ItemStack clones preserve
  the server-side item metadata; the GUI never reconstructs an existing item from text.
- Equipment slots can be inspected or deleted but are not move/give destinations.
- Give requires an empty slot, an exact registered material key, and a legal stack
  size. Unsupported mod item IDs are rejected rather than approximated.
- Before each edit, full inventory/ender contents are serialized to
  `plugins/FlintBridge/inventory-edits/<uuid>/*.bin`. Up to 100 pre-edit records
  are retained per player. These are recovery records, not a GUI restore feature.
- Failure to write the pre-edit record aborts the edit. Two-slot write failures
  roll back both slots. Commands are never automatically retried on timeout.
  After a timeout, refresh and inspect the actual inventory before another edit.
- Replaying an old revision fails. Drag/drop commits a whole-stack move or swap
  immediately. Context-menu deletion commits immediately. No edit-enable switch
  or second confirmation is required. Adding items uses the ID/count inputs.
- Item visuals use local game/mod assets; see `item-visuals.md` for sources and renderer limits.
- Inventory monitoring uses a one-second GUI polling delay on the Players page,
  retaining non-overlapping Core requests and idle-window backoff. After mutations,
  a fresh request follows any pre-command poll; stale responses are not used as
  the post-command refresh.
- Drag previews live in a document-body portal so page transforms cannot offset
  viewport pointer coordinates. Flat and 3D images are alpha-centered inside a
  stable square. Native icon-only controls use the same centering rules.

The HTTP snapshot now reads Bukkit objects on the main server thread, with a
bounded HTTP executor and a two-second snapshot wait.

## Logged-out players

- Last observed details are saved locally in `flint-data/players/last-known.json`.
  The GUI labels them as saved data with their observation time. Data never
  observed by Flint is not invented. Player data files supply saved position,
  health, XP, game mode and inventory where available.
- Current saved inventory is distinct from the ten-minute history. History is
  always read-only. Inventory file changes are detected by stat and SHA-256.
- Core uses the existing `prismarine-nbt` parser/writer on complete typed NBT,
  preserving unrelated fields, custom tags, enchantments and Forge capabilities.
- While Minecraft runs, Core only creates a staged file. The console-only Bridge
  command commits it on the main server thread after checking that the player is
  still offline and the original file hash matches. A login or save invalidates
  the operation. An older Bridge or a failed RCON call never triggers a direct
  file-write fallback.
- When Minecraft is stopped, Core can replace the player file without RCON.
  Invalid/unknown runtime state, launcher locks and active world locks block the
  direct-write path. The startup/backup gate also covers this transaction.
- Full pre-edit player files are saved under
  `flint-data/inventory-edits/<uuid>/*.dat`. No world or inventory restoration is
  performed automatically. Keep normal world backups too.
- Staged offline item creation validates ID syntax and count (1-64), not the
  running Mod registry or each item's maximum stack size. Use exact installed
  item IDs; unknown IDs can be discarded by Minecraft on load. Moving existing
  stacks retains all original metadata. Mod-specific extra inventory containers
  are not exposed as editable slots.
- The running-server commit supports primary worlds under the server root and
  atomic filesystem replacement. External worlds, custom plugin directories or
  filesystems without atomic replacement fail closed rather than guessing paths.

## Crash navigation

Crash activity includes the report identifier. Core resolves that identifier only
against the active server's crash-report list and validates the real path. On
Windows, Explorer opens with the report selected. Missing/deleted reports report
an error; no arbitrary path from the GUI is opened.

## Verification

`npm test` covers command validation, transport failures, and no automatic retry.
`mvn -f bridge/pom.xml test` covers swap, stale/replay rejection, rollback,
give/delete, equipment/container restrictions, console-only access, and backup failure
using isolated test doubles. GUI tests exercise immediate edits, drag/drop, stale
selection, read-only history, and narrow layouts without touching live inventories.
`scripts/test-inventory-ui.cjs` checks pointer dragging, immediate context deletion,
offline editing and add-item input using browser-demo data. Offline Core and
Bridge tests cover full NBT retention, backups, revision conflicts, staged-file
tampering, login races and off-thread rejection in temporary directories.

End-to-end Mohist/mod-NBT behavior still needs a disposable test server before
using editing on valuable items. A production server is not used for destructive tests.
