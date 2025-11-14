# OpenStrand Product Tour Guide

> A narrative walkthrough of the onboarding wizards, dashboard overlays, and storage/sync controls that ship with OpenStrand v2.

## Overview

This tour complements the in-app help overlay that ships with the dashboard. You can launch the same content inside the product by opening the **Help > Product Tour** button in the top navigation. The overlay renders each section of this guide as pageable content, so any update here immediately rolls into the UI.

## Welcome & Orientation

- Locate the tour: **Help > Product Tour** in the dashboard header (or hit `?` to open the Help menu).
- Command palette: `Ctrl/Cmd + K` exposes quick navigation actions for all major surfaces (composer, weave, datasets, admin).
- Docking: click the pin icon to keep the tour visible while you interact with the dashboard--perfect for workshops and onboarding calls.

## Local Workspace & Storage

- Mirrors the "Local workspace storage" card shown in Settings and Profile.
- Explains adapter detection, storage path copy, manual sync, and status chips (last sync / pending changes / conflicts).
- Sync troubleshooting tips:
  - Ensure `OPENSTRAND_SYNC_REMOTE_URL` (or `NEXT_PUBLIC_OPENSTRAND_SYNC_REMOTE_URL`) is set when you expect cloud sync.
  - Check the admin health dashboard for stalled replicas or missing credentials.
  - Run **Sync now** from the overlay to confirm connectivity after environment tweaks.

## Onboarding Wizards

- **LocalOnboarding**: three-step checklist for offline-first deployments, featuring:
  - Storage card quick actions (copy path, sync now, open tour).
  - Links to analog/taxonomy tutorials for initial knowledge capture.
  - Button to jump into the product tour (this guide) mid-onboarding.
- **TeamOnboarding**: cloud-focused wizard covering:
  - Infrastructure readiness checks (API URL, database, AI credentials).
  - RBAC configuration and member invitations.
  - AI automation toggles (publisher, artisan, analytics) with links to detailed docs.
  - "Mark complete" state that hides the overlay once setup is done.

## Collaboration Shortcuts

- Jump into weave manager, composer, or saved visualizations directly from the tour.
- Tips for inviting collaborators or switching to cloud mode (`NEXT_PUBLIC_APP_VARIANT=team`).
- Reminder to leverage the command palette for advanced navigation.

## Support & Docs

- Embedded links to product docs, tutorials, API reference (OpenAPI), and SDK typings.
- Encourages feedback submission via the dashboard feedback pulse widget.
- Highlights admin-only surfaces: `/admin/dashboard`, `/admin/tokens`, `/admin/storage`.

## Tour Controls

1. Click the **Help** icon in the dashboard header and choose **Product Tour**.
2. Use the search box or sidebar to jump between sections. The overlay remembers your last visited section across sessions.
3. Toggle the "Dock" option to keep the tour visible while interacting with the dashboard.
4. Content auto-refreshes when this Markdown changes; no rebuild or redeploy is required.

## Extending the Tour

- Add new sections here (e.g., AI publishing, admin dashboards) and they'll automatically appear in the overlay.
- Embed callout blocks using standard Markdown > quotes or fenced code blocks; the renderer respects headings up to `h3`.
- Keep each section concise--aim for 1-2 screens per page so users aren't overwhelmed inside the modal.

## Related Guides

- [Analog Foundations](./pen-and-paper.md)
- [Metadata Architecture Playbook](./metadata-architecture.md)
- [Developer Experience Blueprint](./dx-ux-blueprint.md)
- [OpenStrand Admin Health Checklist](../DEVELOPMENT_OVERVIEW.md#environment--feature-flags)
