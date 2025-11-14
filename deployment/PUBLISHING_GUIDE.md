# Publishing OpenStrand Packages

This guide covers publishing the OSS packages to npm, tagging releases, and deploying the frontend.

---

## 1. Prerequisites

- npm account with access to the `@framers` scope.
- Two-factor authentication enabled on npm.
- GitHub CLI (`gh`) authenticated for `framersai/openstrand`.
- Clean git working tree (`git status` should report no changes).

Login to npm if you haven’t already:

```bash
npm login --scope=@framers
npm whoami
```

---

## 2. Version Bumping

We use SemVer for the backend + SDK. Keep versions aligned:

```bash
# Example bump to 0.2.0 across packages
npm version 0.2.0 --workspace @framers/openstrand-sdk
npm version 0.2.0 --workspace @framers/openstrand-teams-backend

# Commit the bumps
git commit -am "chore(release): v0.2.0"
```

If using Changesets, run `npm run changeset version` instead and follow the prompts.

---

## 3. Build Artifacts

```bash
npm install    # ensure lockfile is up to date
npm run clean  # optional: remove old build output
npm run build  # builds every workspace
```

This produces:
- `packages/openstrand-teams-backend/dist`
- `packages/openstrand-sdk/dist`
- `openstrand-app/.next`

---

## 4. Publish to npm

Publish workspace packages explicitly:

```bash
# SDK first (clients may install it before the backend)
npm publish --workspace @framers/openstrand-sdk --access public

# Backend bundle
npm publish --workspace @framers/openstrand-teams-backend --access public
```

Verify:

```bash
npm view @framers/openstrand-sdk version
npm view @framers/openstrand-teams-backend version
```

---

## 5. Tag & GitHub Release

```bash
git tag v0.2.0
git push origin main --tags

gh release create v0.2.0 \
  --title "OpenStrand v0.2.0" \
  --notes "• Summary of changes\n• Breaking changes (if any)\n• Upgrade instructions"
```

Attach artifacts if you generate Docker images or tarballs in CI.

---

## 6. Deploy Frontend

### Vercel (recommended)

1. Connect the `framersai/openstrand` repository.
2. Set environment variables in the Vercel dashboard (`NEXT_PUBLIC_API_URL`, etc.).
3. Trigger a deploy from the tagged commit (`v0.2.0`).

### Static Export (self-hosted)

```bash
npm run build --workspace openstrand-app
npm run start --workspace openstrand-app  # preview locally
```

Upload `.next` or `out/` (if using `next export`) to your hosting provider.

---

## 7. Release Checklist

- [ ] Versions bumped and committed.
- [ ] `npm run build` completed successfully.
- [ ] Packages published (`@framers/openstrand-sdk`, `@framers/openstrand-teams-backend`).
- [ ] Git tag pushed and GitHub release created.
- [ ] Frontend deployed (Vercel / static host).
- [ ] Documentation updated (if necessary).
- [ ] Announce the release (community, changelog, etc.).

---

## 8. Troubleshooting

| Issue | Fix |
|-------|-----|
| `You do not have permission to publish` | `npm login --scope=@framers` and confirm your npm role. |
| `Package already exists` | Bump the version before publishing (`npm version patch`). |
| Git reports dirty tree | Commit or stash changes before bumping/publishing. |
| CI/CD deploy fails | Re-run `npm install`, ensure lockfile committed, retry workflow. |

---

Questions? Open an issue in [github.com/framersai/openstrand](https://github.com/framersai/openstrand/issues) or reach out on the community Discord.
