## Release Automation Overview

This repo contains workflows for:

- Monorepo release automation (Release Please) — disabled for submodules.
- Electron build for `openstrand-app` (multi-OS), triggered by:
  - Release created in this repo (tags like `openstrand-app-vX.Y.Z`), or
  - Repository dispatch event `openstrand-app-release` (from the app repo).

### Why monorepo Release Please is disabled for submodules
Release Please fetches files via the GitHub API and cannot look inside Git submodules. Therefore, trying to manage `openstrand-app` changelog from here fails with “Missing required file: openstrand-app/package.json”. We handle releases in the app repo instead.

### openstrand-app repository setup
Inside the `openstrand-app` repo:

- `.github/workflows/release-please.yml` — runs Release Please to create a release PR and tag.
- `.github/workflows/build-electron.yml` — builds installers for Windows/macOS/Linux on published releases.
- `release-please-config.json`, `.release-please-manifest.json` — configure initial version (0.1.0) and changelog.

With Conventional Commits under the app repo, Release Please will generate changelogs and bump versions automatically.

### Building from the monorepo
The monorepo workflow `.github/workflows/build-openstrand-app-electron.yml` supports:

- Release trigger (published) for tags matching `openstrand-app-v*`.
- Repository dispatch event `openstrand-app-release` with client payload `{ "tag": "vX.Y.Z" }` to clone the app repo at that tag and build.

Mobile scaffold (disabled by default):
- Monorepo: `.github/workflows/build-openstrand-app-mobile.yml` (set `if: true` to enable either iOS or Android).
- App repo: `openstrand-app/.github/workflows/build-mobile.yml` (same pattern).

Enabling iOS/Android requires:
- iOS (macOS runners): Xcode, signing certs, provisioning profiles (suggest Fastlane for automation). Configure secrets: `APP_STORE_CONNECT_API_KEY`, `APPLE_TEAM_ID`, etc.
- Android: Java 17, Android SDK, keystore and passwords in repo secrets. Example secrets: `ANDROID_KEYSTORE`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`.

Repository Dispatch from app repo to monorepo (optional):
```yaml
# openstrand-app/.github/workflows/dispatch-monorepo.yml
name: dispatch-monorepo
on:
  release:
    types: [published]
jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Send dispatch
        run: |
          curl -X POST \
            -H "Accept: application/vnd.github+json" \
            -H "Authorization: Bearer ${{ secrets.MONOREPO_DISPATCH_TOKEN }}" \
            https://api.github.com/repos/framersai/openstrand-monorepo/dispatches \
            -d '{"event_type":"openstrand-app-release","client_payload":{"tag":"${{ github.event.release.tag_name }}"}}'
```

Add a fine-scoped PAT in the app repo secrets (`MONOREPO_DISPATCH_TOKEN`) that can trigger repository_dispatch on the monorepo.

### Recommended flow
1. Work in `openstrand-app` repo; use `feat:`/`fix:` commits.
2. Release Please opens a PR; merge it → tag and GitHub Release created in the app repo.
3. App repo’s build workflow packages installers and uploads to that release.
4. (Optional) App repo can send a `repository_dispatch` to this monorepo with the tag to produce mirrored builds here.


