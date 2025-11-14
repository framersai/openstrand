# Developer & Experience Blueprint

> Align engineering, design, and product workflows for OpenStrand extensions.

## Goals

1. Provide a shared map of UI surfaces and underlying services.  
2. Document coding standards, component libraries, and testing approaches.  
3. Outline UX research touchpoints and accessibility checks.  
4. Offer sample roadmaps for extending tutorials, composer, and dashboard modules.

## System Overview

```
openstrand-app (Next.js, Tailwind, Radix)
  ├─ features/composer      -> strand creation, attachments, AI hooks
  ├─ features/dashboard     -> data upload, visualisation, activity
  ├─ app/[locale]           -> i18n routes, tutorials, marketing
  ├─ components/navigation  -> unified header/footer, toggles
  └─ services/*             -> client wrappers for backend

packages/openstrand-teams-backend (Fastify + Prisma)
  ├─ routes/*               -> REST endpoints for strands, AI, attachments
  ├─ services/*             -> business logic (AI, speech, cache, etc.)
  └─ prisma/schema.prisma   -> data models

packages/openstrand-sdk     -> public client (types, fetch helpers)
```

## Code Standards

- **TypeScript everywhere**: explicit interfaces for API payloads, narrow types for metadata.  
- **React Hooks**: maintain strict dependency arrays; prefer `useCallback` + stable refs for event handlers.  
- **Styling**: use Tailwind primitives with SCSS theming. Custom icons go under `components/icons`.  
- **Docs**: add TSDoc comments on exported functions and components.  
- **Testing**: use Vitest + Testing Library; for backend use Vitest integration tests + mocked Prisma.  
- **Linting/Formatting**: repo-wide Prettier config + ESLint (`npm run lint`, `npm run format`).

## UX / UI Process

1. **Research**: capture workshops or interviews as strands (see pen-and-paper guide).  
2. **Design tokens**: add new colours/theme variants via `_theme-engine.scss` + `ThemeSwitcher`. Ensure contrast ratios meet WCAG 2.1 AA.  
3. **Component design**: create Figma/Storybook references if possible; align with shadcn/Radix composables.  
4. **Interaction design**: define states (idle/loading/success/error) and animations.  
5. **Accessibility**: ensure keyboard navigation, focus states, ARIA roles.  
6. **Localization**: wrap text with `useTranslations`, update `i18n/locales/*.json`.  
7. **Documentation**: update relevant tutorial docs + in-app pages.

## DX Roadmaps

### A. Extending Tutorials Section

1. Create Markdown guide under `docs/tutorials/`.  
2. Add corresponding page under `app/[locale]/tutorials/{slug}/page.tsx`.  
3. Update tutorial index page with card + metadata.  
4. Ensure SEO: update `siteMetadata`, `marketing-routes.json`.  
5. Add tests verifying `generateMetadata` returns expected fields.

### B. Enhancing Strand Composer

1. Introduce new metadata panels or AI toggles.  
2. Update `StrandComposer` state, extend DTOs in `sdk/types.ts`.  
3. Wire API calls via `openstrandAPI`.  
4. Provide unit tests mocking `openstrandAPI`.  
5. Update docs referencing new metadata/AI features.

### C. Dashboard Upgrades

1. Build or modify components in `features/dashboard/components`.  
2. Update `DashboardPage` to register new panels/fabs.  
3. Provide story-style fixtures in tests.  
4. Document behaviour in tutorials (DX/UX sections) and release notes.

## Collaboration Rituals

- Weekly triage of `docs/tutorials` additions; ensure parity between docs and UI.  
- Pair programming for backend schema changes + frontend metadata updates.  
- Design reviews focusing on theme/contrast + responsiveness.  
- Testing matrix covering desk/laptop/tablet/mobile breakpoints.  
- Accessibility audits (keyboard, screen reader) before releases.

## Tooling Checklist

- [ ] `npm run dev` (backend + frontend) runs lint-free.  
- [ ] Prettier + ESLint integrated in IDE.  
- [ ] Env loader configured (`env-loader.ts`).  
- [ ] Prisma migrations run (`npx prisma migrate dev`).  
- [ ] Storybook (optional) or Chromatic for visual regression.  
- [ ] Monitoring dashboards for AI usage and queue backlog.

## Further Reading

- `docs/DEVELOPMENT_OVERVIEW.md` – environment loader, brand asset pipeline.  
- `docs/ARCHITECTURE.md` – service topology, AI stack.  
- `docs/tutorials/offline-analytics-guide.md` & `llm-augmentations.md` – data + AI workflows.  
- `docs/tutorials/metadata-architecture.md` – metadata standards.


