# GUI server creation

Open **Servers > Create a new server** (Japanese: サーバー > 新しいサーバーを作成).
The initial existing-server Setup Wizard also offers this entry point.

The five steps are brand, Minecraft/build selection, basic settings, explicit EULA
consent/review, and progress. Going back or closing/reopening the wizard retains
the draft. Running jobs continue while the wizard is closed, not after Flint
exits. The completed job offers a dashboard handoff and another-server action.
Creating a server does not select it, stop another server, or start Minecraft.
Only the explicit dashboard handoff selects the newly prepared server. Use Start
and Console there to follow the real first startup; preparation is not an online
status or a successful gameplay test.

## Supported adapters

| Brand | Source | Installation | Verification |
| --- | --- | --- | --- |
| Vanilla | Mojang release manifest/version metadata | Official server jar | Published SHA-1 and byte count |
| Paper | PaperMC Fill v3 | Stable server builds only | Published SHA-256 and byte count |
| Purpur | Purpur v2 | Numbered server build with SUCCESS result | Published MD5 (transfer integrity, not a signature) |
| Fabric | Fabric Meta v2 | Official generated Server Launcher, pinned loader/installer | HTTPS and ZIP/JAR magic; API does not publish a launcher hash |
| Forge | Forge official JSON catalog + Maven | Execute official installer, then generated Windows argument file | Published SHA-1 and installer Minecraft profile |
| NeoForge | NeoForged official Maven JSON index | Execute official installer, then generated Windows argument file | Published SHA-256 and installer Minecraft profile |
| Mohist | Official builds-raw archive joined to official API metadata | Numbered bootstrap jar | Published API SHA-256, archive link membership and jar format |

Only release Minecraft versions are offered. Builds are resolved and checked
again in Core when creation begins; GUI-provided artifact URLs are never used.
Metadata is cached for five minutes. Provider failures are displayed rather than
inventing versions or silently falling back to a different brand/build.

Fabric's launcher fetches its runtime libraries on the first real startup. Mods
and Fabric API are not automatically installed. Bukkit FlintBridge is not a
Fabric mod; advanced Bridge-based monitoring is unavailable on that brand.
No brand receives an automatically installed Bridge plugin in this phase.

Forge supports release Minecraft versions from 1.17 onward. NeoForge supports
stable numbered builds in the current net.neoforged:neoforge Maven group (20.x,
21.x and four-component 26.x releases), filtered against Mojang's release list.
NeoForge's legacy 1.20.1 forge-group builds and beta-only releases are not offered.
Both run `java -Xmx1G -Djava.awt.headless=true -jar <installer> --installServer`
inside the isolated staging directory, after verifying the published installer
checksum and checking `install_profile.json.minecraft` against the selected game
version. Successful exit is not sufficient: the expected nonempty win_args.txt
must exist and must not refer to the staging directory by absolute path.

The launcher stores `launchMode: "argfile"` and the argument-file relative path
in its existing `jarName` entry-point field. PowerShell and managed launch paths
both pass `-Xms`, `-Xmx`, Flint's additional JVM options, `@<absolute win_args.txt>`
and `nogui`. No installer jar is used as the server entry point, and generated
batch files are not executed. Additional entries in user_jvm_args.txt are not
implicitly imported; configure additional JVM flags in Flint Settings instead.
PID identity matching includes the exact @argument-file path and creation time.
Settings round trips retain launchMode; the launch target field is labeled as an
argument file in the GUI. Existing jar configs continue to default to jar mode.

Mohist supports the official archive versions from 1.18.2 onward that also have
numbered API build metadata with SHA-256. At verification time, the legacy API's
newest Git-hash download returned an empty body. Flint therefore lists explicitly
labeled archive builds only when their exact jar link exists in the archive and
their number has a published SHA-256. It neither accepts empty downloads nor
silently substitutes a different selected build. These are historical builds,
not necessarily the latest maintained release. Current installations are never
updated or downgraded by this new-creation flow. Versions without a matching
archive and checksum are not offered. Spigot/Bukkit remains Coming Soon.

Forge and NeoForge do not support the Bukkit FlintBridge plugin; use unavailable
monitoring states rather than fabricated metrics. Mohist can use a separately
installed, compatible FlintBridge as before.

## Boundaries

- `src/shared/serverCreation.ts`: serializable model shared with the GUI.
- `src/core/creation/BrandAdapters.ts`: versions, builds, resolution and install
  interface. Jar/bootstrap adapters share only their applicable transfer code.
- `OfficialTransport.ts`: HTTPS host allowlist (including every redirect), bounded
  metadata reads, streaming transfers, hashes, cancellation and three attempts.
- `InstallerRunner.ts`: bounded installer profile ZIP reads, direct Java
  subprocess execution, progress output, an 8 MiB install log and a failure tail.
  Cancellation/15-minute timeout terminates only the owned installer tree and
  waits for that termination before staging cleanup. It does not use a java.exe
  name filter, active server PID, RCON stop or launcher action file. The official
  installer itself downloads libraries according to its own trusted profile;
  it is third-party executable code, not an OS-sandboxed operation. Unexpected
  helper/process termination can leave a staging directory or installer process;
  do not treat forced app termination as a completed cancellation.
- `JavaDiscovery.ts`: PATH/JAVA_HOME/common Windows vendor folders and explicit
  java.exe selection. Invokes Java properties/version directly with a timeout,
  rejects 32-bit output, and records java.home's executable when available.
- `ServerCreationService.ts`: validation, independent jobs, owned staging,
  configuration, consent, commit and library registration.
- `ServerSession`: app-level creator requests are deliberately independent of
  the active server's command scope. Metadata work is dispatched asynchronously
  and polled by ID, so it cannot hold the daemon's command queue while waiting
  for a provider. Active-server commands retain their existing scope protection.
- GUI: calls only `ServerSessionClient.creationRequest`; no direct server HTTP,
  RCON, launcher, log, backup or file access. Native dialogs/open-link commands
  remain UI services. Browser mode is explicit preview with no fake installation.

## Safety and limitations

The target must not already exist, even if empty. Linked/junction ancestors,
network shares, Windows reserved names, nested existing servers/worlds and unsafe
world names are rejected. The only automatically deleted path is this job's
unique `.flint-create-<uuid>` sibling staging directory. Failed/cancelled jobs
never publish partial server files. A forced process termination may leave that
hidden staging directory; it is excluded from the library and not automatically
deleted on the next launch. Review it manually before removing it.

Files are prepared before an atomic same-volume directory rename. The extra
library registry `flint-server-library.json` allows arbitrary local destinations
without changing `serversDir` or the active profile. If registration itself fails
after commit, completed files are retained and the error identifies their path.

Fresh launcher/server configs do not inherit legacy agents, commands, Java
arguments or RCON passwords. A random RCON secret and independent currently free
loopback RCON/Bridge port numbers are generated. Ports are not reserved until
startup, and the requested Minecraft game port is not changed automatically.
Multiple servers still need nonconflicting game ports. If Bridge is installed
later, configure its HTTP port to match Settings > Bridge. Minecraft's RCON
listener may bind more broadly than the GUI client; keep it private with Windows
Firewall. No firewall/UPnP/public Internet configuration is performed.

Java requirements use Mojang version metadata plus Fabric loader metadata.
Old metadata without a Java requirement uses Java 8 only for known <=1.16
releases; Java 8 releases conservatively require Java 8 rather than an unverified
newer JVM. Newer runtimes meeting a minimum are not a compatibility guarantee for
every plugin/mod. Forge and NeoForge official server installer execution is
implemented; automatic Java runtime installation is not. The wizard links to official
Eclipse Temurin downloads.

Metadata timeout covers the body (20 seconds); jar transfer timeout covers the
whole attempt (10 minutes, maximum three attempts, <=1 GiB). There is no range
resume; a failed attempt restarts from zero. Fabric's bootstrap library transfer
is handled by its official launcher later, outside this download progress.

## Verification

- `npm test`: isolated temp directories, mock official providers, hashes,
  consent/paths, queue cancellation, transaction cleanup, custom registration,
  configuration isolation and unchanged active profile. No real world is touched.
- `node scripts/test-creation-ui.cjs`: requires the Vite dev server. Uses the
  explicit UI-only fixture, Playwright/Edge and screenshots in `.build/creation-ui`.
  Covers 540/980/1280/1920 widths, draft/build retention, input focus across parent
  polling renders, consent, progress and dashboard handoff.
- `node scripts/test-creation-rpc.cjs`: verifies the packaged helper's new RPC
  catalog/query path, ping, EULA refusal and unchanged active configuration using
  a temporary app profile. It does not connect to a Minecraft server.
- Official metadata was also smoke-tested for all four brands with 1.20.1. This
  is not a real server startup, gameplay, save/restart or mod compatibility test.
- `FLINT_TEST_REAL_INSTALLERS=1 npx tsx scripts/test-real-loader-installation.ts`:
  optional live download/installation test. Verified Forge 1.20.1-47.4.28, NeoForge
  21.1.264 for Minecraft 1.21.1, and Mohist 1.20.1 archive #923 in disposable local
  directories. Both official installers exited successfully; committed argument
  files and launcher configs were checked, and Mohist's actual jar matched its
  SHA-256. Reports are in `.build/loader-installation-qa`. No Minecraft server was
  started and no world was created. Temporary installations were removed.
- `FLINT_TEST_JAVA_HOME=<JDK directory> npx tsx scripts/test-installer-process.ts`:
  optional local Java fixture verifies parent/child cancellation, timeout and
  executable-not-found handling without running an actual installer/server.
- Unit tests also compare PowerShell's generated argument builder with managed
  arguments, including a server path with spaces, and test launchMode retention,
  incorrect PID modes/timestamps, archive source filtering and invalid builds.

The shared response-body reader also explicitly races abort signals against
pending reads. This keeps a stalled metadata response (and the existing map
discovery response) from holding a request forever on affected Node runtimes.

Official references:
https://piston-meta.mojang.com/mc/game/version_manifest_v2.json
https://docs.papermc.io/misc/downloads-service/
https://purpurmc.org/docs/purpur/
https://meta.fabricmc.net/
https://fabricmc.net/use/server/
https://adoptium.net/temurin/releases/
https://www.minecraft.net/eula
https://docs.neoforged.net/user/docs/server/
https://github.com/MinecraftForge/Installer
https://files.minecraftforge.net/net/minecraftforge/forge/maven-metadata.json
https://maven.neoforged.net/api/maven/details/releases/net/neoforged/neoforge
https://mohistmc.com/builds-raw/
