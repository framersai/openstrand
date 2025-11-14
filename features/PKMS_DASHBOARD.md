# PKMS Dashboard (Knowledge)

This page explains how to use the PKMS dashboard in the app (`/[locale]/pkms`) to create, import, and organise your knowledge strands, and how the first‑visit welcome/modal works.

## Overview

- The PKMS page is a dashboard, not a landing page.
- A first‑visit “About PKMS” modal presents high‑level information with a “Show this again next time” option (unchecked by default). You can reopen it via the “About PKMS” button.
- The dashboard is fully themeable and respects light/dark/system themes.

## Quick Actions

- New note: Opens the WYSIWYG Strand Composer.
- Use a template: Opens a template gallery (Zettelkasten Note, Research Summary, Project Brief, Meeting Notes).
- Import or capture: Deep‑links to the Import page for files/media/datasets.

## Creator (WYSIWYG)

- The Strand Composer supports title, summary, note type, tags, difficulty, and rich content authoring.
- Templates can prefill the composer with a structured outline.
- Voice notes and media attachments are integrated elsewhere in the composer module and respect plan limits.

## Strands List

- Lists your strands with search and grid/list view toggles.
- Data is loaded via `openstrandAPI.strands.list`, with graceful fallback to an empty list if the API is offline.
- Each item shows title, summary, tags, and relative time.

## Welcome Modal

- Stored preferences:
  - `pkms:welcome:hasSeen` – whether the user has seen the modal at least once.
  - `pkms:welcome:showAgain` – whether to auto‑show next time (default: false).
- Reopen anytime via the “About PKMS” button.

## Internationalisation (i18n)

- All PKMS UI strings are sourced from the `pkms` i18n namespace.
- If a locale lacks `pkms.json`, English (`en`) is used as a fallback automatically.
- See `docs/I18N_SYSTEM_GUIDE.md` for adding/maintaining translations.

## File Map

- App route: `openstrand-app/src/app/[locale]/pkms/page.tsx`
- Dashboard: `openstrand-app/src/components/pkms/PKMSDashboard.tsx`
- Welcome modal: `openstrand-app/src/components/pkms/PKMSWelcomeModal.tsx`
- Templates: `openstrand-app/src/components/pkms/TemplatesGallery.tsx`
- Composer: `openstrand-app/src/features/composer/components/StrandComposer.tsx`
- Translations (namespace `pkms`): `openstrand-app/src/i18n/locales/*/pkms.json`

## Auto Metadata & Autosave

- Autosave: Enabled by default in the Creator. Edits are saved after periods of inactivity (about 8 seconds) and on explicit Save.
- Auto-tag: After saving, the client triggers concept extraction (`aiAPI.extractConcepts`) and merges results with existing tags (deduplicated, capped). You can toggle this on/off.
- Auto-backlinks: After saving, the client fetches `getRelated` and creates lightweight `related` edges (capped). Toggleable; sensible default is off.
- Inline visualizations: Wizard can export a PNG and attach to the strand while storing JSON config/data in `content.metadata.visualizations`.
- Background behavior: Failures are silent and non-blocking; the UI shows lightweight status updates. All operations are idempotent backend-side.

### Configuration

- Per-user toggles: Autosave, Auto-tag, Auto-backlinks (with max backlink count).
- Team policy can override defaults (future server support).

### Preferences & Review Suggestions

- Preferences Modal: Access from the PKMS header (Preferences). Toggles persist per device:
  - Autosave, Auto-tag, Auto-backlinks, Review suggestions before applying, Max backlinks.
- Review Flow (optional): When enabled, after save you'll see a modal with suggested tags and backlinks to apply.
  - Tags are derived from AI concept extraction.
  - Backlinks are derived from related strands. You can select some or all to apply.

### API Touchpoints

- Tagging: `openstrandAPI.ai.extractConcepts(plainText)`
- Backlinks: `openstrandAPI.strands.getRelated(strandId, limit)` + `openstrandAPI.strands.createRelationship(...)`
- Save/Update: `openstrandAPI.strands.create` / `openstrandAPI.strands.update`

### Data Model

- Tags: `strand.metadata.tags` (array of strings)
- Visualizations: `strand.content.metadata.visualizations[]` with `{ id, type, title, config, data, createdAt }`

## Glossary (PKMS Terms)

- Strands: Atomic units of knowledge (notes, documents, media, datasets). Everything is a strand.
- Weave: The knowledge graph—how strands connect via typed relationships.
- Looms: Curated sets/views (collections, threads, patterns) for workflows and publishing.
- Datasets: Structured/tabular data represented as strands that can generate notes and visualizations.
- Entities: Named people, places, projects, or concepts (authored or extracted) used as anchors in the weave.
- Relationships: Typed edges (e.g. references, prerequisites, part-of) that power recommendations and graph exploration.

All of these are accessible from the Strands dashboard via quick capture, composer, templates, importer, and the Knowledge Graph view. Tooltips and the in-app help modal provide concise definitions and links.

### Data Intelligence Panel
- Backed by `packages/openstrand-teams-backend/src/services/data-intelligence/DataIntelligenceService` (deterministic heuristics) plus matching SDK helpers—no LLM is required for community/offline usage.
- Displays vocabulary summaries (TF/IDF, bigrams, co-occurrence pairs) for the active workspace or Loom. Team edition scopes results via RBAC and shows trend deltas; personal edition only surfaces local collections.
- Extracts lightweight entities (people, orgs, projects, locations) via regex/patterns and writes them to `strand.metadata.entities`.
- Enterprise workspaces can toggle an optional “LLM verification” step that runs after heuristics to double-check or annotate the generated metadata before it is published to Looms or exports.

> **Edition behavior**
> - Community Edition displays a single “Global Loom” selector that is effectively the entire workspace; all strands live inside this one collection. Creating extra Looms/projects is disabled until the user upgrades.
> - Teams/Enterprise Edition shows the full Loom selector (storytelling projects, world-building threads, PKMS notebooks, etc.). Each Loom maps to a dedicated scope/team so users can juggle multiple initiatives safely.

## Weave Preview & Looms

- Weave Preview: A compact knowledge graph preview appears on the Strands dashboard. Click “Open graph” to explore the full Weave page.
- Looms: Curated sets such as threads (and patterns) show up as a list. Click through to manage or publish collections.

## Teams Integration

- Team selector (if available) appears in the Strands header; filtering the strands list by team ID is supported via API `strands.list({ teamId })`.
- Team admin API is used to discover teams (`team.list()`). If unavailable, the selector is hidden and personal workspace is used.

## Deletion & Publishing Policy

- Visibility: private, team, unlisted, public. Publishing is a visibility change; unpublish reverts to private. Public links return 404 when unpublished. A minimal public tombstone is used internally for graph continuity.
- Delete flow: Two-step confirmation + 3-second undo (UI). If not undone, deletion proceeds. Defaults keep placeholders and avoid graph churn; hard-delete is admin-only.
- Datasets: Deleting archives the dataset strand and removes auto-generated derivatives by default; user-authored notes are preserved. Seeded datasets are protected (admin removal only).
- GDPR: Retention and deletion behavior follow GDPR defaults—documented in Terms/Privacy and product pages.




