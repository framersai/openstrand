# Exports and Imports: Detailed Guide

This guide explains every export and import capability in OpenStrand, including CSV behavior for datasets and Obsidian vault workflows.

## Exports

Supported across single strand, multiple strands (collection), and weave exports (varies by type).

- Markdown (`markdown`)
  - Per‑strand headings, optional frontmatter summary, and TOC when requested.
  - Ideal for static sites, Obsidian, and docs.
- HTML (`html`)
  - Clean, self‑contained HTML with default CSS. Optionally add custom CSS.
- PDF (`pdf`)
  - Professionally paginated with optional title page, TOC, and page numbers.
- DOCX (`docx`)
  - Word‑compatible with headings, paragraphs, and metadata table.
- JSON (`json`)
  - Raw strand/weave objects (the API responds with parsed JSON in `data`).
- CSV (`csv`) — datasets focused
  - Single dataset strand with tabular rows:
    - If `strand.content.data` (array of objects) exists, we export it with a union of row keys as headers.
    - If no tabular rows are found, we fall back to a strand metadata table.
  - Multiple strands: CSV falls back to a metadata table across strands.
  - Note: CSV is best for single dataset strands with inline rows. For bulk datasets or multiple tables, prefer JSON or ZIP.
- ZIP (`zip`)
  - Archive of per‑strand Markdown and a `metadata.json` manifest.
  - Obsidian vault layout supported via `archiveLayout: "obsidian"`.

Weave exports additionally support `graphml` (returned as text in `data`) for graph analysis tools.

### CSV Details (Datasets)

- Headers: union of all keys seen in `content.data` rows.
- Values: strings are quoted as needed; objects/arrays are JSON‑stringified.
- Fallback: if no rows present, we export a metadata CSV with columns: `id,title,type,created,tags`.
- Limitations:
  - Does not paginate or chunk large datasets; supply pre‑sampled/preview rows in `content.data` for best results.
  - Multi‑table datasets should be exported as JSON or ZIP.

### Obsidian Vault Export

- Request: `format: "zip", archiveLayout: "obsidian"`.
- Output structure:
  - `vault/notes/*.md`: each note includes a YAML frontmatter block with `id,title,created,updated,type,tags` and the Markdown body.
  - `vault/metadata.json`: export summary.
  - `vault/.obsidian/*`: minimal scaffolding for immediate open in Obsidian.

## Imports

- Repo import
  - `POST /api/v1/submissions/import-repo { repoUrl, branch?, projectName?, authToken? }`
  - Private repos require a PAT (Teams/Enterprise). Token is never logged/returned.
- Obsidian vault import
  - `POST /api/v1/submissions/import-obsidian` (multipart with `file: zip`).
  - Maps YAML frontmatter and folders into strand metadata and scopes. Wiki‑links parsing planned.
- Folder/Zip import (generic)
  - Upload a zip of files and map to strands; directory structure preserved. UI under Import Project wizard.

## UI Surface (Product Cards)

- Export & Import (Landing → Features):
  - “Export strands to Markdown, HTML, PDF, DOCX, JSON, CSV (datasets), or an Obsidian‑ready vault (ZIP). Import repos via URL (PAT for private), zips, and Obsidian vaults.”
- Composer → Import Project Wizard:
  - Obsidian: file input for zipped vault; confirmation message on queueing.
  - Repo: URL + optional PAT (Teams/Enterprise); saved PAT toggle; save/clear token actions.

## API Usage

- Strand export:
  - `POST /api/v1/export/strand/:id`
  - Body: `{ format, includeMetadata?, includeRelationships?, markdownFlavor?, archiveLayout? }`
  - Returns binary for file formats; `data` for JSON/GraphML.
- Collection export (multiple strands):
  - `POST /api/v1/export/collection`
  - Body: `{ strandIds, format, options? }`
  - Returns ZIP/CSV/Markdown/PDF/HTML/DOCX; JSON returned parsed in `data`.
- Weave export:
  - `POST /api/v1/export/weave/:id` with `{ format }`

## Tips

- CSV: Best for single dataset strands with inline `content.data`. For complex datasets, use JSON.
- Obsidian: Use archiveLayout=obsidian to open immediately in Obsidian with metadata preserved.
- PDF/DOCX: Include TOC and title pages via style options for polished exports.


