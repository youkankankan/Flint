# Flint Auto Update

Flint uses the Tauri v2 updater.

Update source:

```text
https://github.com/youkankankan/Flint/releases/latest/download/latest.json
```

The desktop app checks silently 4 seconds after startup, hourly while visible,
and on returning to the app / network after at least 15 minutes. Checks never
overlap. Browser demo does not check or install updates.

Available updates show their version, release notes and an Update / Later action.
The user must explicitly confirm download, installation and app restart. Downloads
have a 120-second timeout. Tauri verifies the signed package. Installation waits
for the Core shutdown handshake; active jobs prevent installation. Minecraft is
not stopped. Polling pauses while Core is suspended. Failed preparation or install
resumes Core, retains the update for retry, and displays the error. A relaunch
failure after successful install asks the user to reopen Flint, without installing twice.

## Signing Key

The updater public key is stored in `src-tauri/tauri.conf.json`.

The private key must stay local and must never be committed:

```text
%USERPROFILE%\.flint-updater\flint-updater.key
%USERPROFILE%\.flint-updater\flint-updater.password.txt
```

If the key is missing, generate it:

```powershell
npx tauri signer generate --write-keys "$env:USERPROFILE\.flint-updater\flint-updater.key" --password "<strong-password>" --ci
```

Keep the same key for future releases. If the private key or password is lost, already-installed apps cannot verify future updates signed with a new key.

## Release Flow

1. Update `src-tauri/tauri.conf.json` `version`.
2. Build:

```powershell
npm run tauri:build
```

3. Upload these files to the matching GitHub release tag, for example `v0.1.0`:

```text
src-tauri/target/release/bundle/nsis/Flint_<version>_x64-setup.exe
src-tauri/target/release/bundle/nsis/Flint_<version>_x64-setup.exe.sig
src-tauri/target/release/bundle/latest.json
```

The release must be the latest GitHub release because the app reads `releases/latest/download/latest.json`.

## Draft Upload Helper

Versioned release notes live in `docs/releases/v<version>.md` (for example,
`docs/releases/v0.1.1.md`). `updater:manifest` reads that file into `latest.json`
and `release:draft` uses the same text for the GitHub Release body. The app shows
the manifest notes in its update notice. `FLINT_UPDATE_NOTES` remains an explicit
override; when neither override nor file exists the fallback is the version title.
After editing notes, regenerate the manifest; a stale notes draft is refused.

`npm run release:draft` validates the built version, manifest URL and matching
signature, then uploads the NSIS installer, signature and manifest to a **draft**
GitHub Release. The public repository must contain only README, documentation,
images and license notices. The release tag targets its public `main` commit;
the local source checkout is never pushed. The script does not publish a release
or overwrite an existing release. Review the draft and publish it as Latest to
activate distribution.

The public distribution repository is `youkankankan/Flint`. Existing source history
is preserved in the separate private `Flint-source` repository. Never push source
branches, source-derived tags, or a mirror of that repository into public Flint.

Always increase the app version (`src-tauri/tauri.conf.json` and the Flint crate)
before building: clients do not receive an update with the same version number.
The current update-enabled build is 0.1.1. GitHub Release publication is a separate
step; local build success alone does not make the endpoint available.

Older clients without updater support need this installer once. Legacy Overseer
clients with a different application identity, endpoint or signing key must not
be assumed to upgrade automatically into Flint. Keep the existing signing key.

Distribution moved to `youkankankan/Flint`. The former `ouiegbfwvdsj/Flint`
updater URL no longer serves a public manifest. Install this release once over
clients using that old endpoint, including previously shared 0.1.1 builds. The
current installer points directly at the new public distribution repository.
