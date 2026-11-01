# Bridge creation and startup verification (2026-10-31)

These checks use disposable local servers, not the user's existing server,
worlds, mods or player data. EULA consent was explicitly obtained for these
tests. Each test uses an isolated app profile, unique loopback management/game
ports, its own generated world, and a newly downloaded official distribution.

## Verified scenarios

| Brand | Minecraft | Bridge mechanism | Scope |
| --- | --- | --- | --- |
| Forge | 1.20.1 | Forge mod | Create, Done, HTTP, save, stop, restart; TPS/MSPT/maxPlayers checked |
| NeoForge | 1.21.1 | NeoForge mod | Create, Done, HTTP, save, stop, restart; TPS/MSPT/maxPlayers checked |
| Fabric | 1.20.1 / 1.21.1 | Fabric mod with nested lifecycle/base API | Create, Done, HTTP, save, stop, restart; TPS/MSPT/maxPlayers checked |
| Vanilla | 1.20.1 / 1.21.1 | Version-mapped Java agent | Create, Done, HTTP, save, stop, restart; TPS/MSPT/maxPlayers checked |
| Paper | 1.20.1 | Bukkit plugin | Create, Done, HTTP, save, stop, restart |
| Purpur | 1.20.1 | Bukkit plugin | Create, Done, HTTP, save, stop, restart |
| Mohist | 1.20.1 | Bukkit plugin | Create, Done, HTTP, save, stop, restart |
| Spigot | 1.20.1 | Bukkit plugin | Official BuildTools build, Done, HTTP, save, stop, restart |
| CraftBukkit | 1.20.1 | Bukkit plugin | Official BuildTools build, Done, HTTP, save, stop, restart |

Raw local evidence: `.build/bridge-startup-qa/results.json`,
`results-forge-neoforge-fabric-fabric121-vanilla-vanilla121.json`,
per-case creation/result JSON and console logs. These ignored build artifacts
are not published or included in the installer.

## Bugs caught by verification

- Vanilla's HTTP dispatcher initially prevented process exit. The agent now
  stops the HTTP server from the mapped shutdown method; clean stop and restart
  passed for both target versions.
- Forge's maximum-player method originally read the online count. The mapping
  was corrected against the installed official mapping and tested against the
  configured limit (2), independently of zero connected players.
- Native player detail now uses the same `{ok, player}` envelope as the existing
  HTTP client. Missing players return 404. A Java test checks a synthetic real
  field payload and verifies execution on the server executor.
- Native bridges without inventory fields no longer record those fields as an
  empty inventory. Persisted inventory remains the fallback when available.
- The 1.21.1 smoothed tick-time accessor differs from 1.20.1. Both mappings were
  added and measured rather than substituting TPS-derived MSPT.

## Boundaries

This is not a test of every Minecraft version, modpack, authenticated player
join, inventory edit, Dynmap, or GUI/PowerShell-managed lifecycle. Real startup
tests spawn Java directly with the same generated launcher configuration and
argument builder. GUI layout/draft tests and packaged-helper RPC tests are
separate fixtures. Native inventory editing, chunk/ticket diagnostics, automatic
native updates and automatic native Dynmap deployment remain unsupported.

Spigot/CraftBukkit's build adapter requires Git and a JDK. Downloads use official
HTTPS; BuildTools does not provide a signed server-jar artifact verified here.
Native dependencies include their Apache/BSD licenses and attribution notices.
No production server process was stopped or restarted for this verification.
