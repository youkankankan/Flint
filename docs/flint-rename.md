# Flint Storage and Identity

Canonical names use Flint throughout: `dev.youkan.flint`, `FLINT_*`,
`flint-app.json`, `flint-server.json`, `flint-launcher-config.json`, `flint-data`,
`flint-backup.lock`, `backups/flint`, `flint.console.history`, `flint-inventory`
and `FLINT_INVENTORY`. The repository and updater URLs use
`https://github.com/youkankankan/Flint`. Existing signing material is imported into
`%USERPROFILE%/.flint-updater`; the signing key itself is not regenerated.

## One-Way Import

Old names exist only in migration code and migration-test fixtures. They are not
new output names or environment-variable aliases.

- Existing Flint app configuration wins. Malformed originals are reported instead
  of resetting paths. Original app configuration is retained at its source.
- Server configuration is imported without changing Java paths, JVM arguments,
  credentials, selected servers or custom backup locations.
- Storage directories and the default backup root are renamed only when the
  server is stopped and has no live PID, launcher lock or backup lock. Active
  sessions keep their current storage tree until a stopped migration.
- Original server configuration is archived in `flint-data/brand-migration`.
  Conflicting old/new directories are never merged or overwritten silently.
- A missing Flint browser profile is imported before WebView creation. Existing
  Flint profiles win; locked originals remain untouched. Console history in an
  imported profile is copied into the canonical local-storage key.

Close the old application before starting Flint. Do not run two versions against
the same server. No world, installed Mod, plugin JAR or player file is edited.

## Bridge Deployment

The plugin registers Flint commands/permissions only. Inventory editing requires
FlintBridge with the canonical command and response marker. HTTP monitoring
endpoints are unchanged. Deploy `bridge/target/FlintBridge.jar` during a planned
server stop and remove the old plugin JAR so two bridges do not compete for the
same port. This refactor does not replace a live plugin or publish a new release.

## Verification

Unit tests cover config precedence, malformed input, live-session deferral,
stopped migration, archive preservation, conflict refusal and browser history.
Isolated helper RPC tests cover app-directory migration and scoped commands.
Tests do not use real server or user configuration.
