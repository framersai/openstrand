# Repository and Folder Imports → Strands

This document describes how to map repositories or folders into strands with full metadata and hierarchy.

---

## Approaches

1) Zip and Upload (available now)

- Create a submission (`POST /api/v1/submissions { type: "DATASET" }`).
- Upload a zip file to `POST /api/v1/submissions/:id/upload`.
- The backend profiles content; you can attach metadata and finalize.
- A mapping step can classify files by type and create one strand per file/path, recording parent directories as scopes/hierarchy (service extension).

2) Import by Repo URL (API scaffold)

- `POST /api/v1/submissions/import-repo { repoUrl, branch? }` queues an import job (202 Accepted).
- The job (to be implemented) clones/shallow-fetches the repo, indexes files, and creates strands mirroring directory structure.

---

## Mapping Model

- A repository (or folder root) becomes a top-level scope (team/project).
- Each directory becomes a scope; files become strands with:
  - `strandType`: based on file type (code, note/markdown, dataset, media).
  - `metadata`: language, size, hash, path, repo URL/commit, license headers if found.
  - `hierarchy`: parent scope chain reflecting path segments.
  - `links`: infer references (e.g., imports) as conceptual links.

---

## UI/UX

- Landing and Dashboard menus include “Import Project” which opens the Composer in the dashboard where the Import wizard lives.
- Landing: “Drag & Drop Folders → Strands” product card and tutorial link.
- Composer: “Import Project” wizard supports zip upload and repo URL (with PAT for Teams/Enterprise).
- PKMS: browse hierarchy by scopes (directories); open strands with proper highlighting.
- Obsidian: zip import supported via “Import Obsidian vault (zip)” input in the wizard.

---

## Future Enhancements

- Repo re-sync: detect changes and update strands incrementally.
- Git metadata: authorship, commits, CI status surfaced in strand metadata.
- Source graph: render imports/refs in knowledge graph.

---

## Security & PAT (Teams/Enterprise)

- Private repository imports accept a Personal Access Token (PAT).
- PAT is provided only at submit time and never returned by the server.
- Server must prevent tokens from being logged; audit logs should redact any secret fields.
- Community Edition supports public repositories; PAT fields are disabled in UI.
- Teams/Enterprise users can save a GitHub PAT encrypted at rest (AES‑256‑GCM) in their profile. The UI wizard can use the saved token or a per‑import override. The saved token is never shown back to the user; only presence and last updated time are displayed.


