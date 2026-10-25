# Local item textures and slot menu

The desktop GUI requests item visuals through ServerSession. React does not read
Minecraft files or open Bridge/RCON connections. Visuals are loaded separately
from monitoring snapshots so PNGs are not resent on every poll.

## Asset Sources

Assets are read locally, in this precedence order:

1. ZIP packs placed explicitly in `<server>/flint-assets/resourcepacks/`.
2. `<server>/flint-assets/client.jar`, when provided.
3. `%APPDATA%/.minecraft/versions/<server-version>/<server-version>.jar`, then
   installed client profiles explicitly inheriting that version (up to 32 jars).
4. The selected server's `mods/*.jar` archives.

The version is detected from the launcher jar name or the server log header.
Unrelated client resource-pack selections are not imported automatically. Users
without a local Minecraft installation can supply their own matching client jar.
Minecraft/third-party texture files are not redistributed with Flint.

## Rendering

- Generated/handheld items use the original PNG layers with nearest-neighbor scaling.
- Standard JSON block models inherit parent textures, element faces, UVs, rotations,
  and GUI transforms. One reusable offscreen Three.js renderer creates cached icons;
  there is no WebGL context or animation loop per inventory slot.
- Animated PNG sheets show their first frame.
- FlintBridge 0.3.2 supplies CustomModelData and leather/potion colors. Older
  Bridges still supply base item IDs, but cannot select all metadata-dependent looks.
- Static CustomModelData-only model overrides are supported. Predicate-dependent
  game state, custom entity/OBJ/Forge renderers, dynamic mod renderers, emissive
  shaders, biome-dependent colors, and all resource-pack extensions are not fully
  reproduced. When possible a custom renderer falls back to its actual local
  texture (model texture references, item PNG, then block PNG); these previews
  carry `approximate: true`. No substitute Minecraft item texture is invented.
  Missing assets use a neutral package glyph instead of a question mark. Untyped
  block tint uses a default grass tint.
- Generated icons are centered by their visible alpha bounds. Concurrent renders
  of the same icon share one promise. Moving stacks does not reload the same
  texture set, and old visual responses are retained only until a new set arrives.

ZIP extraction uses bounded entry sizes and validates namespaces/resource paths.
PNG signatures/dimensions, model depth, element count and request count are bounded.
Archive indexes, visual responses, and rendered icons have size/count limits.
No archive is extracted into the server/world filesystem.

## Slot Menu

Right-click or Shift+F10 opens the slot menu. Available actions include details,
copy ID, move/swap from the previously selected slot, adding an item into
an empty slot, and delete the entire stack. Menus support arrow keys and Escape,
close on outside click/scroll, and clamp to the viewport.

History is read-only; current offline player files can be edited through Core.
Revision checks and pre-edit backups still apply. Dragging or choosing Delete
commits immediately without an edit toggle or second confirmation. The Add item
menu focuses the ID/count inputs rather than creating a default item immediately.
A right-click alone never changes an item.
