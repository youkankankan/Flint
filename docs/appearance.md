# Appearance presets

Settings > Appearance provides Flint Glass, Liquid and Graphite. Presets share
the same components and navigation; they change material, color and shape
tokens rather than maintaining separate applications. Console, file explorer,
inventory and map interactions are unchanged.

Choose a preset to preview it across the app. A small unsaved-preview notice
remains visible when navigating to another page. Save commits the current
settings draft. The preview notice's Undo action restores only appearance,
preserving unsaved path/server settings. Discard restores the whole settings
draft. Reset appearance to defaults changes only the appearance draft.

Color is independent of material. Eight palettes are available: Lavender, Ice
Blue, Ocean, Mint, Rose, Champagne, Silver and Mono. Glass/Liquid adopt restrained
background tints; Graphite keeps neutral surfaces and changes accents only.
Success, warning and danger colors keep their original meanings. Color changes
use the same live-preview/save/undo path and preserve other dirty settings.

Options include comfortable/compact density, translucent/opaque surfaces and
reduced motion. Graphite is always opaque; the translucency preference is kept
for switching back to Glass or Liquid. OS reduced-motion preferences are always
respected. Blur is limited to existing small chrome/overlay surfaces; native OS
window transparency and continuously animated filters are not enabled.

Canonical settings live under `appearance` in the app-level `flint-app.json`:

```json
{
  "preset": "glass",
  "palette": "lavender",
  "density": "comfortable",
  "motion": "system",
  "translucency": true
}
```

Older profiles use default values without a migration write. Core validates
preset/option values and supports partial updates with optimistic revisions.
Appearance-only saves do not write server, launcher or server.properties files.
The GUI uses a validated localStorage visual cache for startup; Tauri's actual
app settings win once the helper responds. Only committed settings reach the
cache; abandoned previews are not persisted. Browser demo saves update this
visual cache only, not server files.

Verification:
- `node scripts/test-palette-ui.cjs`: verifies all 24 palette/material pairs,
  primary-button label contrast, fixed semantic status colors, persistence and
  dirty-path preservation. Screenshots and measured contrast are saved under
  `.build/palette-ui/`. These are browser-demo tests, not native-server tests.
- `npm test`: validation, partial merge, legacy defaults, app-only persistence,
  revision conflicts and server/path preservation in temporary profiles.
- `node scripts/test-appearance-ui.cjs`: requires the Vite dev server and
  Playwright/Edge; uses browser demo, not real server commands. Covers all three
  presets, navigation previews, save/reload, undo with a dirty path, responsive
  layouts, and OS reduced motion. Screenshots: `.build/appearance-ui/`.
- `FLINT_TEST_HELPER=.build/flint-core-helper.exe` with
  `npx tsx scripts/tests-documents-rpc.ts`: packaged-helper save/snapshot checks
  and unchanged server settings in a temporary profile, plus existing document
  and support-pack integration checks.
