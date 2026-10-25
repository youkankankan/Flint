# Mods Explorer

The Mods workspace uses Explorer-style chrome while keeping Flint's glass,
purple tokens, radii and shared context menus. The default is a compact file-name
table with the details pane closed. Mod display names, versions and dependency
metadata belong to the optional property pane, rather than repeated two-line rows.

- Back/forward history and Up to the virtual all-mods view.
- Enabled (`mods`) and disabled (`flint-data/disabled-mods`) physical folders;
  all-mods and needs-attention are virtual views, not extra filesystem directories.
- Search by filename, display name and Mod ID; details/list views; sortable and
  resizable columns (double-click a column boundary to reset its width).
- Click, Ctrl-click, Shift-click, blank-space marquee, Ctrl+A, Ctrl+arrows for
  focus-only movement, Shift navigation, Home/End, Page Up/Down and type-ahead.
- Drag unselected file names, empty cells or background to marquee-select.
  Drag an already selected file label into an enabled/disabled navigation folder
  to request a move. Ctrl/Shift-drag always selects instead of moving.
  Native text selection/dragging is suppressed only in the file table; search and
  the property pane remain selectable.
- Cut / paste (Ctrl+X / Ctrl+V) is an Flint-local move queue between these two
  managed folders. It is NOT the Windows filesystem clipboard. Escape cancels it.
  No move occurs until the destination and file list are confirmed.
- Right-click or Shift+F10 opens file operations. Delete always requires explicit
  confirmation and is permanent, not the Windows Recycle Bin.
- Native Tauri file drops and the file picker copy JARs into the selected physical
  folder; all/attention views default to `mods`. Dropping over a physical navigation
  folder chooses that destination. Disabled imports never enter the loader folder.
- F5 refreshes through Core, Ctrl+F focuses search. External drag-out to Windows
  Explorer and arbitrary path navigation are not implemented. Open in Explorer
  remains available when those OS-level operations are needed.

All mutations still go through scoped ServerSession/CommandRouter commands. Core
validates fixed destination roots, duplicate IDs, file types, symlinks, collisions
and import limits. Folder changes do not restart the server. Active Mod changes
apply on its next launch; existing worlds, plugin jars and player data are untouched.

Verification: isolated Core import/disable/enable tests and browser-demo Playwright
mouse/keyboard tests. Browser QA confirms move requests/cancellation, not native
Windows drag payload delivery or real-server hot loading.
