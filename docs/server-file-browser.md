# Server file workspace

The sidebar Files entry replaces the former Mods entry. Its default view browses
the selected server root. Mod analysis and dependency checks remain in the adjacent
Mod analysis tab. Both views use Flint's existing glass/purple styling and
Explorer pointer-selection logic. The file area has left/right/bottom gutters so
marquee selection can begin outside file labels.

## Operations

- Browse physical folders with breadcrumbs, history and Up; search the current
  folder and sort folders-first by name, date, type or size.
- Click, Ctrl/Shift-click, marquee, Ctrl+A, arrow/Home/End navigation and Enter.
- Cut/copy/paste within the selected server, label drag to folder destinations,
  F2 rename, folder creation, and right-click actions.
- Native file/folder drop and file/folder picker imports copy into the destination.
  Existing names are never intentionally overwritten. Imports are not executed.
- UTF-8 text files up to 256 KB open in the built-in editor. Saves check content
  revisions and preserve originals in `flint-data/config-history`. Existing
  supported config documents still use ServerFilesService and its mirrored config
  handling. Binary files/JARs are not edited or launched.
- Remove moves entries to `flint-data/file-trash/<id>` and records a JSON
  manifest with original/destination paths. This is not the Windows Recycle Bin.
  Recovery currently uses Explorer and the manifest, not an in-app restore button.

## Boundaries

All disk access goes through scoped ServerSession/CommandRouter requests using
ServerContext. The GUI never opens server files directly. Browser preview is
in-memory demo data; the packaged app never falls back to it.

Core rejects traversal, ADS, Windows device names, trailing-dot/space aliases and
symlinks/junctions in child segments. Copies/imports also reject linked descendants.
The server root cannot be renamed/removed. Flint-managed configuration/runtime
files and `flint-data` are protected against generic mutation; use Settings or
the existing specialized tools. Running/starting/unknown runtime is read-only in
the GUI. Core additionally refuses mutation for live state PIDs, launcher locks,
nonterminal or corrupt state files. A manually launched server with neither state
nor launcher lock cannot be reliably identified by file operations alone; stop
externally managed servers before editing their files.

Batch mutations require confirmation and are capped at 256 roots, 10,000 tree
entries and 2 GB total. Copy operations stage into unique temporary paths. Batch
copy work also has a 30-second deadline so slow disks do not leave a helper request
waiting indefinitely; partial staged copies are cleaned up on failure. Batch
failures attempt rollback and report paths that could not be recovered. External
filesystem races cannot be made fully transactional across other applications.
Large world transfers beyond these limits should use Explorer while stopped.

## Verification

`npm test` includes temporary-directory Core safety tests. The document RPC suite
also checks the new commands through the Node/packaged helper. The browser GUI
test covers gutters/marquee, folder navigation, runtime read-only state, mkdir,
rename, copy, editor save/draft retention, confirmation and responsive widths.
It does not claim native OS drag/drop verification or modify a live server.
