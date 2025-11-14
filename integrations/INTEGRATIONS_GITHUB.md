# GitHub Integration (Teams/Enterprise)

This document covers how OpenStrand integrates with GitHub using per‑user Personal Access Tokens (PATs) stored encrypted at rest.

## Overview

- Each user in Teams/Enterprise can save a GitHub PAT.
- Tokens are encrypted (AES‑256‑GCM) and never returned via the API.
- The app proxies selected GitHub endpoints so tokens are not exposed to browsers.

## Endpoints

- GET /api/v1/integrations/github/token
  - Returns: `{ success, data: { hasToken: boolean, updatedAt?: string } }`
- PUT /api/v1/integrations/github/token
  - Body: `{ token: string }`
  - Stores token encrypted; returns presence metadata
- DELETE /api/v1/integrations/github/token
  - Clears saved token

Proxy (read‑only) endpoints using saved PAT:

- GET /api/v1/integrations/github/user
- GET /api/v1/integrations/github/repos

## Security

- Tokens are encrypted with AES‑256‑GCM via `INTEGRATIONS_SECRET_KEY` (32 bytes).
- Tokens are never logged or returned from any API.
- Proxy endpoints prevent tokens from being exposed to browsers or CORS.

## UI

- Composer → Import Project wizard offers:
  - “Use saved GitHub token” toggle (if present) with last updated time
  - Save/Clear token actions
  - Optional one‑off PAT per import (not stored)

## Notes

- Community Edition supports public repos only (PAT UI disabled).
- Future: additional proxy endpoints can be added as needed (branches, contents, tree).


