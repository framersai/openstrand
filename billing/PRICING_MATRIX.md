# Pricing & Editions Matrix

Fair, transparent comparison of Community vs Teams/Enterprise features.

| Capability | Community (Free) | Teams/Enterprise |
| --- | --- | --- |
| Offline-first core | Yes | Yes |
| PKMS dashboards | Yes | Yes |
| Runnable snippets (TS/JS/Python) | Yes (bundled) | Yes (bundled) |
| Broad syntax highlighting | Yes | Yes |
| Drag & drop folder → strands (zip) | Yes | Yes |
| Repo import (public) | Yes | Yes |
| Repo import (private with PAT) | — | Yes (per‑user encrypted PAT) |
| Obsidian vault import/export | Yes | Yes |
| CSV/JSON exports | CSV (datasets), JSON | CSV (datasets), JSON |
| Structure approvals & audit | Basic | Advanced queues & SLAs |
| Placeholder policy engine | Yes | Team policies & analytics |
| Visibility cascades | Global workspace only | Per-team + per-project scopes |
| Deterministic data intelligence (vocabulary, NER, stats) | Yes (local, offline) | Yes + RBAC scopes + optional LLM verification |
| Workspace / Loom limits | 1 global Loom for entire catalog | Unlimited Looms/projects per team |
| Multi-user / RBAC | Single user (optional sign-in for sync) | Full teams, guests, external reviewers |
| Custom domains | — | Yes |
| SSO & governance | — | Yes |
| Billing & quotas | — | Usage metering + SLA dashboards |

### Pricing & Licensing

- **Community Edition** – Free, MIT-licensed, and lifetime by default. Feature parity with Teams except for multi-tenant/project/user functionality. Users may optionally sign in (Supabase, email, etc.) for sync/backups, but still operate a **single global workspace**.
- **Teams Edition** – $1,000 lifetime during launch window. Includes unlimited Looms/projects, RBAC, SSO, governance tooling, and admin dashboards.
  - `.edu` student or non-commercial creator discount: **$250** (automatically applied when verifying a school email).
  - Personal power users (non-commercial) can purchase a **50% discounted** seat ($500) to unlock multi-project tooling.
  - After the launch period we will move to annual upgrade/support licenses (~$100/year) for Teams; early lifetime buyers keep perpetual updates.
- Support model:
  - Community: GitHub Discussions / Issues (`https://github.com/framersai/openstrand`).
  - Teams: Dedicated support channel + optional paid success plans.

Notes:
- Runnable Python adds ~20–30MB to app bundle via Pyodide; fully offline on Electron and mobile (Capacitor).
- Private repo imports require a PAT; tokens are never logged or returned by the API.


