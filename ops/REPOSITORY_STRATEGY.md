# OpenStrand Repository Strategy Options

_Private working draft & interview preparation_

## 1. Current Topology (Status Quo)

- **Single Git history**: `openstrand-monorepo` contains app, admin console, SDK, and teams backend under one `.git` directory.
- **Subtree exports**: When we need standalone repos (`openstrand-app`, `openstrand-admin`, etc.), we run `git subtree push …` to copy the relevant directories’ history into those remotes.
- **Pros**
  - One checkout for day-to-day development, tests, and dependency management.
  - Cross-project refactors (e.g. API contract change that touches app + backend) happen atomically in a single commit.
  - Simple CI pipeline: build/test matrix can run from the monorepo entry point.
- **Cons / pain points**
  - Standalone repos only update when we remember to run `git subtree push`, and their history diverges immediately afterward.
  - Rewriting commit history (lowercasing subjects, etc.) is tricky because the monorepo and subtree remotes must be kept in sync.
  - Git reports the entire workspace as one history; difficult to reason about ownership or repo size if each team only cares about a subdirectory.
- **Important cleanup (Nov 2025)**: We removed the extra nested `openstrand/` clone that previously lived inside the repo. There is now exactly one working tree; the only nested Git repos are the declared submodules (`openstrand-app`, `openstrand-admin`, `packages/openstrand-*`, `docs/openstrand`). This prevents edits from accidentally landing in the mirrored copy.

## 2. Option A – Stay With Monorepo + Subtree (Tuned)

- **What changes**
  - Keep current structure but invest in automation/helper scripts.
  - Add hook-enforced conventional commits (already done) and CLI tools to rewrite history in bulk when needed.
  - Treat `promote-openstrand.sh` as the canonical “sync script” that pushes all subtrees.
- **Add-ons / best practices**
  - Wrapper script that clones each standalone repo temporarily → runs lint/rewrite → force-pushes → deletes clone.
  - CI job that verifies each subtree is up to date (diff between monorepo folder and exported repo).
  - Documented release checklist that includes `git subtree push` for all artifacts.
- **Pros**
  - Minimal structural disruption; no re-cloning or reconfiguring build/CI pipelines.
  - Keeps development friction low (one workspace, one set of tooling).
  - Straightforward to onboard new engineers: “clone monorepo, run scripts.”
- **Cons**
  - Subtree repos remain “mirrors” rather than first-class projects. Contributors who only want the app still see rewritten history when we sync.
  - Git commands like `git log packages/openstrand-teams-backend` include unrelated commits unless we filter carefully.
  - Larger repository size over time (node modules, built assets, etc. accumulate unless we prune).
- **Teams that do this**
  - Many product orgs with tight coupling (e.g. mobile app + backend) use monorepos with automation to publish artifacts/packages.
  - Tooling-heavy companies (Meta, Google) use monorepos but also invest heavily in build tooling and code ownership metadata to mitigate the pain.
- **Recommendation**
  - Short-term keep this model, because it avoids retooling and the branch rewrite script now handles most policy enforcement.

## 3. Option B – Convert Key Directories to Git Submodules

- **What changes**
  - Replace the `openstrand-app`, `openstrand-admin`, `packages/openstrand-sdk`, `packages/openstrand-teams-backend` folders with submodule pointers to their respective repos.
  - The monorepo now tracks submodules as separate Git repositories.
- **Pros**
  - Each submodule is an independent repo (has its own `.git`), so history rewrites, hooks, and commit style enforcement are simple.
  - Clear ownership boundaries: teams can clone and work only on their submodule without pulling the entire workspace.
  - CI can skip cloning modules that aren’t required for a given job.
- **Cons**
  - Day-to-day Git workflow is more complex. Every change in a submodule requires a commit inside the submodule **and** a pointer update in the parent repo.
  - Cloning the monorepo requires `git submodule update --init --recursive`. Forgetfulness leads to weird build/test failures.
  - Tools/scripts need updating (paths change; automation must run `git submodule` commands first).
  - Branch discipline matters: if submodules point at `master` but the app team works on feature branches, the parent repo must track the correct refs.
- **Teams that do this**
  - OSS projects with optional components (e.g. documentation or vendor SDKs) that are versioned separately.
  - Companies that must mirror third-party code verbatim while still keeping it embedded in a main repo.
- **Recommendation**
  - Only adopt if each component has largely independent release cadence and engineering ownership. For OpenStrand’s current size/coupling, submodules may create more friction than value.

## 4. Option C – Full Polyrepo (Split Completely)

- **What changes**
  - Break the monorepo apart; each component is an independent Git repo.
  - Introduce tools (e.g. GitHub Actions, Jenkins, NX, Lerna) to orchestrate cross-repo change testing and version bumps.
- **Pros**
  - Each repo history is clean, targeted, and small. New contributors just clone the piece they need.
  - Continuous delivery per component is easier (each repo has its own release pipeline).
  - Access controls can be fine-grained (backend repo private, docs repo public, etc.).
- **Cons**
  - Coordinating cross-cutting changes is harder—you must open multiple PRs and merge in the right order.
  - Shared configuration/dependencies risk diverging unless you invest in centralized package/infra management (e.g. npm packages, Terraform modules).
  - Requires heavier tooling to ensure the app + backend stay compatible (integration tests across repos, release dashboards, etc.).
- **Teams that do this**
  - Organizations with mature platform teams and very clear service boundaries (microservice architectures, microfrontend shops).
- **Recommendation**
  - Long-term option once OpenStrand’s components stabilize and have dedicated owners. For now, the overhead likely outweighs the benefit.

## 5. Evaluating Our Needs

| Requirement                          | Monorepo + Subtree (current) | Submodules                  | Full Polyrepo |
|--------------------------------------|------------------------------|-----------------------------|---------------|
| Cross-component refactors            | ✅ One commit                 | ⚠️ Need pointer updates     | ❌ Multi-project PRs |
| Commit enforcement                   | ✅ via hooks & rewrite tool   | ✅ per submodule            | ✅ per repo    |
| Standalone repo cleanliness          | ⚠️ Requires subtree sync      | ✅ Always matched           | ✅ By design   |
| Onboarding friction                  | ✅ Single clone               | ⚠️ Need submodule init      | ❌ Multi-repo setup |
| CI setup                             | ✅ Single pipeline            | ⚠️ Mixed submodule builds   | ❌ Multiple pipelines |
| Future scaling / team boundaries     | ⚠️ Manage via codeowners      | ✅ Natural boundaries        | ✅ Strong boundaries |

Key observations:
- Most of our pain today centers around commit style and syncing the exported repos. Both issues are addressable with tooling in the monorepo.
- We don’t yet have independent delivery cadences per component that warrant submodules or full polyrepo.
- Interview talking point: emphasize we evaluated all three and chose the least disruptive path while setting ourselves up to migrate later if needed.

## 6. Recommended Plan (Short-to-Mid Term)

1. **Stay with the monorepo + subtree model** but automate more of it:
   - Keep the `rewrite-commit-history.js` CLI + hook enforcement.
   - Add a script to iterate across exported repos (clone → rewrite → push → clean) for when we need lower-level hygiene.
   - Codify a release checklist that includes running `promote-openstrand.sh` and verifying the standalone repos.
2. **Document ownership + contributions** using CODEOWNERS or a docs page so commits/PRs have context.
3. **Re-evaluate in 6–12 months**. If components grow and become independently staffed, consider submodules or a gradual polyrepo migration.

### Interview Soundbite
> “Today we run a focused monorepo with automated subtree exports. It keeps cross-component changes easy while still publishing standalone artifacts. We evaluated submodules and a full polyrepo, but chose to invest in tooling first—the payoff of a more complex topology wasn’t worth the workflow friction yet.”

## 7. Migration Checklist (if we ever opt for submodules)

1. Freeze main development, ensure all branches are merged.
2. For each component:
   - Clone the respective standalone repo.
   - Remove the component directory from the monorepo (commit).
   - Add submodule pointing to the repo: `git submodule add git@github.com:framersai/openstrand-app.git openstrand-app`.
   - Commit the submodule pointer change.
3. Update CI/build scripts to run `git submodule update --init --recursive` before building.
4. Communicate new workflow to the team (double-commit model, pointer updates, etc.).

## 8. References & Further Reading

- Git subtree vs. submodule comparison: https://www.atlassian.com/git/tutorials/git-subtree
- Git filter-repo (recommended history rewrite tool): https://github.com/newren/git-filter-repo
- “Scaling Git workflows at $COMPANY” blog posts (Stripe, Shopify, Airbnb) — many highlight monorepo automation rather than splitting.

---

_Last updated: 2025-11-05_

