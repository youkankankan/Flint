# Dynmap Automatic Installation

Core automatically prepares a missing Dynmap when the selected server is known
to be Paper, Purpur, Spigot, Bukkit or Mohist, with a Minecraft version in the
explicit supported list (1.20 through 1.21.11, released versions only). New-server
metadata is preferred. Existing servers without metadata require a real configured
JAR with a recognizable brand/version filename; defaults alone are not evidence.
Unknown servers and pure Forge/NeoForge/Fabric do not receive a Bukkit plugin.
Mohist compatibility is not guaranteed for every Mod combination.

## Source and Integrity

The pinned artifact is the author's
[Dynmap 3.8 Spigot release](https://dev.bukkit.org/projects/dynmap/files/7460127),
downloaded directly from the official ForgeCDN file 7460/127. Published MD5:
`d79b989e4a16321e57a09685e3640b04`. The retrieved artifact was checked against that
MD5; Flint pins its SHA-256:
`4771dbe8cbb3ec6a3ee080141361de6d822eac55b55aad7c9e5de031423a6dd7`
and exact size 11819070 bytes. The official file's internal plugin.yml actually
reports `3.7-SNAPSHOT-1015`; this is checked explicitly, not relabelled as 3.8.
Future file/version changes require a reviewed catalog/code change.

Downloads reuse Core's HTTPS allowlisted, streaming, retrying transport with a
90-second overall deadline. Temporary files are removed on cancellation/failure.
The JAR must pass size/hash/plugin identity checks before it is installed. No
classes are executed by the downloader. Dynmap is downloaded from its distribution,
not rehosted or included as a new unsigned external helper in Flint's installer.
Its original bundled licenses remain intact. See
[the author's source and license](https://github.com/webbukkit/dynmap).

## Behavior and Safety

- Existing plugin descriptors are inspected, including renamed JARs. An existing
  Dynmap is left untouched: this feature installs missing Dynmap, not upgrades it.
- Snapshot monitoring starts the operation without awaiting the download. Progress
  is shown in Map. Start/restart waits for preparation; download failure does not
  prevent Minecraft from starting. No RCON or automatic restart/reload is sent.
- A new JAR is placed atomically in plugins using exclusive hard-link creation;
  an existing file is never overwritten. It becomes effective on the next plugin
  load/startup, even if Flint has been closed. No live Dynmap is replaced.
- Initial configuration uses the full original template, changing only the bind
  address to `127.0.0.1` and a local TCP port from 8123-8186. Existing configuration,
  databases, tiles and worlds are not changed. Existing public binds remain the
  user's configuration; Flint does not silently rewrite them.
- Port selection skips listeners and configured ports of other registered Flint
  servers, serialized by an app-wide allocation lock. This is not a permanent
  reservation against unrelated programs starting later. Conflicts can still occur.
- Commit shares the existing startup/backup mutation gate. A conflicting operation
  defers installation. Linked or unsafe paths are rejected. A visible Dynmap Mod
  JAR prevents a second Bukkit installation; arbitrarily renamed Mod JARs cannot
  reliably be identified by this check.
- No fullrender command is sent. Maps may initially be blank and fill as chunks
  are rendered normally. Full-world rendering remains an explicit user action.

Tests cover temporary fixtures and the actual downloaded official JAR, without
launching Minecraft or modifying the production server. Real Dynmap loading,
rendering and embedded-map operation after first startup remain separate runtime
verification. Production/helper build and GUI operation tests do not prove those.
