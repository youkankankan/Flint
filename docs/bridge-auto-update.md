# Bridge Auto-update

Flint's production Core embeds the freshly built FlintBridge JAR, version and
SHA-256 checksum. `helper:build` runs `bridge:build` first and refuses to package
a failed build. Publishing an app update therefore distributes its Bridge update
without a separate unauthenticated plugin download. Increment Bridge's plugin
version whenever releasing changed Bridge code; equal versions are preserved.

## Runtime Behavior

- On snapshot refresh/server selection, Core checks the selected server, at most
  once per minute. Start/restart orchestration also requests that check.
- Only an already installed plugin named `FlintBridge` is managed. Renamed JAR
  filenames are supported by reading bounded plugin.yml metadata. Servers without
  Bridge, including pure Forge/Fabric/NeoForge, are not given a Bukkit plugin.
- If the bundled version is newer, original bytes are retained in
  `flint-data/bridge-backups/<version>-<sha256>.jar`. Configuration and the active
  plugin JAR are untouched.
- An atomic update is staged with the installed filename under the Bukkit update
  folder, respecting `bukkit.yml`'s `settings.update-folder` (default `update`).
  Bukkit consumes it at the next plugin load/startup. Flint does not issue reload,
  restart or stop just to install an update. The queue survives closing Flint.
- Dashboard shows the pending version and next-restart notice. Blocked updates
  show a concise deferred state with the reason in the tooltip.
- Native folder behavior is documented by
  [Bukkit Server.getUpdateFolder](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/Server.html#getUpdateFolder()).
  The installed Mohist 1.20.1 SimplePluginManager bytecode was also inspected:
  loadPlugin calls checkUpdate, using the installed JAR filename in that folder.

## Safety and Limits

The updater verifies bundled identity/hash, compares semantic versions, refuses
downgrades, preserves same-version custom builds, detects duplicate Bridge plugins,
and refuses unrelated queued updates. Empty/disabled, unsafe or linked update
paths are rejected. Its own obsolete queue is removed when a newer installation
is observed. No world, Mod, player data, Bridge config or RCON/HTTP monitoring
architecture changes are involved. The updater never installs other plugins.

Only selected servers are checked; select each existing server after updating
Flint to queue its Bridge update. An externally replaced active JAR is outside
Flint's control; do not manually modify plugins while a server is loading them.
The installed/on-disk version is not asserted to be the currently loaded version.
Other maintenance tools and manual changes can cause the update to be deferred.

Tests use temporary fixtures and simulate Bukkit consuming the queue. Real live
restart/application is not claimed verified; no production server was restarted
or plugin deployed during implementation. `core:build` requires an already built
`bridge/target/FlintBridge.jar`; use `bridge:build` after changing Bridge source.
Unbundled source-mode Core does not invent a downloadable bundle.
