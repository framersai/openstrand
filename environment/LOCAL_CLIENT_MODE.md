# OpenStrand Local Client Mode

OpenStrand ships with an offline-friendly posture so individuals or teams can explore data without creating a hosted account.

## Goals

- **Zero-reg**: No Supabase or hosted login is required. Users can opt into a default username/password stored locally or configure their own credentials.
- **Local persistence**: All strands, datasets, and visualization metadata are stored in an embedded SQLite database. Export/import commands let people back up or migrate between devices.
- **Feature parity**: Tier classification, heuristic parsing, schema intelligence, and AI Artisan generations are available as long as the user provides API keys via the settings dialog. When you bring your own keys (BYOK) AI Artisan runs are unlimited; only the shared OpenStrand global keys carry platform quotas.
- **Upgrade path**: When users decide to sync with hosted OpenStrand, the same `openstrand-sdk` models are posted to the premium backend so migration is lossless.

## Implementation Notes

- Authenticate via `POST /api/v1/auth/local/login` when running without Supabase. Provide the `LOCAL_AUTH_USERNAME` / `LOCAL_AUTH_PASSWORD` and the API returns a bearer token (or the derived token when no value is configured).

- The Node backend (`packages/openstrand-teams-backend`) runs in "local" mode when `OPENSTRAND_ENVIRONMENT=local`. No external auth provider is queried.
- Configure `OPENSTRAND_AUTH_MODE=local` (default) and optionally set `LOCAL_AUTH_USERNAME`, `LOCAL_AUTH_PASSWORD`, and `LOCAL_AUTH_TOKEN` to gate access when running locally.
- The Fastify auth plugin resolves providers based on `OPENSTRAND_AUTH_MODE` (see `src/middleware/auth.middleware.ts`).
- SQLite-compatible PGlite databases live under `~/.openstrand` by default. Override via `OPENSTRAND_LOCAL_DB_PATH`.
- Exports are delivered as signed bundles (`.ostr`) containing the SQLite database plus dataset assets. Imports validate schema versions using the shared SDK.
- The settings dialog exposes the offline toggle and surfaces the current storage path so users can quickly reveal it in Finder/Explorer.
- The local client reuses the same React app; the only differences are the auth provider, quota enforcement, and optional Supabase integrations.

## Next Steps

1. Implement the local auth backend and wire it through the modular dependency graph.
2. Add export/import actions to the settings dialog with background worker feedback.
3. Document CLI utilities for scripted exports (e.g., `npx @framers/openstrand-sdk export --path my.strand`).

