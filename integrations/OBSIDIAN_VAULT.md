# Obsidian Vault Import/Export

OpenStrand supports moving content to/from Obsidian with minimal friction.

## Export to Obsidian

- Use `/api/v1/export/collection` or `/api/v1/export/strand/:id` with:
  - `format: "zip"`
  - `archiveLayout: "obsidian"` (and optionally `markdownFlavor: "obsidian"`)
- Output: a `.zip` containing:
  - `vault/notes/*.md` — each note has YAML frontmatter (`id`, `title`, `created`, `updated`, `type`, `tags`)
  - `vault/metadata.json` — export summary
  - `vault/.obsidian/*` — minimal app scaffolding

Notes:
- Content is exported in Markdown; HTML and plain text fields are converted where possible.
- Attachments export is planned; set `includeAttachments: true` for future support.

## Import from Obsidian

- Use `/api/v1/submissions/import-obsidian` (multipart/form-data with `file: zip`).
- Returns `202 Accepted` with `jobId` for async processing.
- The service maps markdown files into strands:
  - Reads YAML frontmatter (`title`, `tags`, `created`, `updated`) when present
  - Uses folder hierarchy to create scopes and relationships
  - Parses wiki-links/markdown links for suggested relations (future enhancement)

UI:
- In the dashboard Composer, “Import Project” wizard provides a file input for Obsidian zip uploads.

## Compatibility

- Frontmatter keys are a superset of common Obsidian practices; downstream tools can ignore unknown keys.
- Markdown flavor uses Obsidian-compatible frontmatter and basic syntax.


