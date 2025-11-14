## OpenStrand App Releases (Electron) - v0.1.0 and beyond

This document explains how app releases are produced across platforms, how changelogs are generated, and how tags are managed in the monorepo.

### TL;DR
- Conventional Commits drive changelog and version bumps.
- Release Please (monorepo mode) creates release PRs and tags for `openstrand-app`.
- When a release is published (tag like `openstrand-app-vX.Y.Z`), a build workflow packages Electron apps for Windows, macOS, and Linux and attaches artifacts.

---

### 1) Versioning and Changelog (Release Please)
We use Google’s Release Please in monorepo mode:
- Config: `release-please-config.json` and `.release-please-manifest.json` at repo root.
- Component: `openstrand-app` is tracked, initial version set to `0.1.0`.
- It scans commit messages that touch `openstrand-app/**` for Conventional Commits:
  - `feat:` increments minor (X.Y+1.0)
  - `fix:` increments patch (X.Y.Z+1)
  - `BREAKING CHANGE:` increments major

Output:
- A release PR with generated `openstrand-app/CHANGELOG.md` changes and version bump.
- Upon merge, Release Please creates a GitHub release + tag named `openstrand-app-vX.Y.Z`.

### 2) Build and Publish (Electron, all OS)
Workflow: `.github/workflows/build-openstrand-app-electron.yml`
- Triggers on published release events for tags starting with `openstrand-app-v`.
- Matrix builds on: `ubuntu-latest`, `macos-latest`, `windows-latest`.
- Steps:
  1. `npm ci` in `openstrand-app`
  2. `npm run export:static` (Next -> `out/`)
  3. `npm run electron:build` (electron-builder)
  4. Uploads `.dmg/.zip` (mac), `.nsis/.zip` (win), `.AppImage/.deb` (linux) as artifacts.

Notes:
- By default builds publish back to GitHub Releases via `GH_TOKEN` (Actions token).
- For notarization or signing, configure platform secrets in the workflow.

### 3) Local Dev / Testing
- Dev: `npm run dev` (Next) or `npm run electron:dev` (Next dev + Electron shell).
- Static export preview (optional): `npm run export:static` then open `out/index.html`.

### 4) Initial Release v0.1.0
- Manifest sets `openstrand-app` to `0.1.0`.
- First release PR will be based on commits; if needed, manually tag `openstrand-app-v0.1.0` and publish to trigger builders.

### 5) Conventional Commits
Use:
- `feat(app): add X`
- `fix(app): correct Y`
- `refactor(app): ...`

Scope is optional; affecting files under `openstrand-app/**` ensures inclusion.

### 6) Monorepo and Other Packages
- Additional components (e.g., backend) can be added to `release-please-config.json` packages section with their own changelog paths.

### 7) Roadmap
- Mobile builds (Capacitor iOS/Android) in a separate workflow.
- Optional code signing/notarization steps per platform.

---

### Files Added/Updated
- `.github/workflows/release-please.yml` – creates release PRs and tags.
- `.github/workflows/build-openstrand-app-electron.yml` – packages Electron for all OS on releases.
- `release-please-config.json` + `.release-please-manifest.json` – monorepo release config.
- `openstrand-app/electron/main.js` – Electron main entry; loads Next dev or static export.
- `openstrand-app/electron-builder.yml` – packaging targets and publish config.
- `openstrand-app/CHANGELOG.md` – changelog managed by Release Please.
- `openstrand-app/package.json` – scripts for `export:static`, `electron:dev`, `electron:build`.

---

### FAQ
Q: Can we attach the built installers to the GitHub release automatically?
A: Yes. electron-builder publishes to GitHub when `GH_TOKEN` is set. The workflow already exposes `GITHUB_TOKEN`. For private releases or advanced flows, set a personal access token.

Q: How do we change the app ID/product name?
A: Update `openstrand-app/electron-builder.yml` (appId/productName) and `capacitor.config.ts` (appId/appName) if you also target mobile later.

Q: How does Release Please pick the correct package?
A: The monorepo config maps the `openstrand-app` component and analyzes commits that touched files beneath that directory.


