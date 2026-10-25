# Support Packs and Server Configuration

## Entry Points

- Sidebar > AI Pack: a full-page workspace for describing a problem, exporting,
  reviewing and copying the support pack.
- Console and Settings > Advanced: support pack actions open the same workspace.
- A newly observed crash opens AI Pack automatically, but never exports or uploads
  anything automatically. Existing reports at startup do not interrupt navigation.
- Settings > Launcher: edit additional JVM arguments, one complete argument per line.
- Settings > Files: edit existing `server.properties`, `bukkit.yml`, `spigot.yml`,
  `config/paper-global.yml`, `config/paper-world-defaults.yml`,
  `mohist-config/mohist.yml`, and `plugins/FlintBridge/config.yml`.

The GUI uses ServerSession commands only. It does not read local files or contact
Bridge, RCON, or the launcher directly. No server is restarted automatically.
Health and Crash Center pages are removed from the GUI. Core safety checks and
diagnostic data remain available to startup validation and support pack exports.
Dashboard activity still opens crash report locations directly in Explorer.

## Configuration Safety

File reads and writes are restricted to the allowlist above, within the selected
server. Links/junctions below the server root are rejected. Text files are limited
to 256 KiB, with binary/invalid UTF-8 rejected. YAML syntax and duplicate keys are
validated; plugin-specific semantic validity still depends on the plugin.

Saving requires a matching server directory and SHA-256 revision. A conflict keeps
the draft intact. Copy the draft before reloading an externally changed file.
Original bytes are retained under `flint-data/config-history` before atomic
replacement. These local originals can contain passwords; do not share them.
Known server.properties fields are mirrored back into Flint's server config.
YAML changes require a server restart and are not hot-reloaded by Flint.

JVM arguments use the existing launcher configuration and are passed as argument
arrays, not through a shell. Memory uses the Xms/Xmx controls; classpath/entrypoint
overrides through additional flags are rejected. Java agents execute code: only
configure trusted local jars. The preview is display-only, not a shell command.

## Support Pack Privacy

Exports are local Markdown files, optionally accompanied by a ZIP, under
`flint-data/support-packs`. Nothing is uploaded automatically. The existing
left-compose/right-preview workspace is retained. Checkboxes select logs, config,
mods, crashes and history; detailed diagnostics require a separate opt-in. The
progress indicator is indeterminate with elapsed time, not a fabricated percentage.
The result lists successful, failed, unsupported and skipped collectors. The file
picker previews the same masked detailed files that are placed in the ZIP.

ZIP contains `README.md`, `AI-PROMPT.txt`, `manifest.json`, and organized session,
context, config, logs, error windows, crash reports, mod metadata, history and
diagnostic files. Markdown embeds bounded previews; full retained excerpts are in
the ZIP. Log tails are capped at 1 MiB/file, error-context scans at the first
32 MiB/file and 2500 lines / 512 KiB text, and retained artifacts at 10 MiB total.
Reports are capped at two, 512 KiB each. Truncation, missing data and provenance
are explicit. This is not a lossless archive of arbitrarily large logs.

Known secrets, common credential assignments, URL queries/credentials, IPs, UUIDs,
and user paths are masked. Chat/command-chat lines are omitted; player lists,
inventory records, world files, ops/whitelist databases, and arbitrary plugin
configs are not collected. Raw crash reports may still contain personal details.
Redaction cannot guarantee anonymity: review the preview before sending it to AI.

Structured JSON is redacted recursively and remains valid JSON. Launch identifiers
are stable pseudonyms, distinct from redacted player UUIDs. Known sensitive JVM
arguments are masked, including separate flag/value arguments. Local launch
records and diagnostic histories are NOT sharing artifacts and may contain raw
paths/arguments: do not share those directories directly.

## Launch Session Attribution

Strict launcher version 3 starts a new UUID/UTC timestamp for every Java launch,
including automatic/manual restarts. It writes `flint-data/current-launch.json`
and `flint-data/launch-sessions/<id>/session.json`. Exit captures stdout/stderr
into that session directory, exit code, launcher error, observed ready time, phase
and candidate crash-report names. While Java runs, the root redirected logs are
available; archived session outputs appear after exit. Managed mode also records
per-launch output and exit metadata, but failures without a readiness marker remain
conservatively unclassified rather than guessed to be a particular loading stage.

The baseline size/mtime of latest.log is recorded before launch. Unchanged old logs
and older reports are labelled previous-session references, not the current cause.
Other correlations are `current-session-candidate`, never proof of causation.
Session-scoped archived stdout/stderr is stronger attribution. File timestamps
alone cannot prove ownership if another process writes the same directory.
Servers launched by older Flint versions have no retrospective session identity.
The new launcher is generated on the NEXT start, never installed over a live run.
Raw session output is retained until manually managed; no automatic deletion of
diagnostic evidence is performed, so monitor disk usage.

## Lightweight History

Core records at most one sample every 10 seconds, keeping up to 360 samples with a
3 MiB disk cap. Normal snapshots are reused. While those are not arriving, a
guarded Core timer samples a verified server process and local Bridge /status
(1500 ms HTTP deadline, no RCON). This works while the GUI is unfocused/hidden,
but stops when the helper/app exits. Each sample identifies session, source,
method and capture time. No values are interpolated through gaps.

New Bridge /status adds JVM heap, uptime and cumulative GC count/time. CPU/RAM
come from the existing process monitor; TPS/MSPT remain Bridge observations.
Old Bridge versions cannot supply new fields and those remain null/unavailable.
Config hashes and jar size/mtime are compared every 60 seconds; at most 30 change
events are retained. These are bounded fingerprints, NOT exact change times or
proof a jar's contents are unchanged. The first observation is a baseline.
Pack exports additionally compare selected config/mod evidence with the previous
pack. Neither mechanism can reconstruct events before monitoring began.

## On-Demand Diagnostics and Limits

FlintBridge 0.4.0 adds loopback-only `GET /diagnostics` with a 30-second cooldown.
It collects JVM heap/GC and up to 128 thread states/stacks (24 frames/thread)
independently of server-thread scheduling. Worlds/plugins are read on the server
thread with a 2-second scheduling deadline. Failure of that component preserves
the independent JVM/thread evidence. World count APIs are feature-detected;
unsupported server implementations return explicit unsupported records.

Per-world information includes implementation chunk/entity/tile-entity counts,
already-loaded Bukkit chunk counts, and Bukkit plugin-ticket samples from at most
256 loaded chunks, with a shared 75ms between-call budget and at most 32 worlds.
Individual API calls cannot be preempted by that budget. No chunk-at/load API is
used and no world/entity/block-entity full census is performed. These are not
Forge Mod tickets, ticking chunks, startup readiness futures or global plugin
ticket totals. Native forced-chunk totals, Forge/Mohist distance-manager tickets
and unfinished chunk futures remain unsupported pending a version-verified
read-only adapter. A discrepancy between differently-defined counts is not
automatically flagged as abnormal.

External fallback uses JDK `jcmd Thread.print -l`, `GC.heap_info`, and
`VM.command_line` only after fresh PID/creation-time/jar identity verification.
Each attach has a 7-second timeout and bounded output. jcmd must exist beside the
configured Java, under JAVA_HOME/bin or on PATH. A JRE alone, disabled attach,
different privileges or a dead JVM may make it unavailable. Attach itself can
briefly pause the JVM; it is opt-in, never periodic. No forced GC, heap dump,
process kill or automatic restart is performed. A single thread dump is not proof
of a freeze. Cached GUI snapshots carry their actual observation timestamp and
are not relabelled as fresh evidence during export.

The AI prompt asks for facts versus hypotheses, evidence citations, failure-stage
classification, time/definition-aware comparisons, missing measurements and
backup-first copy-environment fixes. Pack content is untrusted evidence, not AI
instructions. No server/plugin update, world mutation or AI upload is automatic.

Browser mode uses in-memory demo settings/files and labels exports as previews;
real file editing/export requires the desktop app. Tauri failures never fall back
to demo data.
