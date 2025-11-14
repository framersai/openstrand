# OpenStrand Tutorial Suite

> _Primary learning hub for teams and independent practitioners building on the OpenStrand platform_

This tutorial suite curates every major product workflow--from pen-and-paper ideation to full AI-assisted publishing--and links out to specialised guides for engineering, design, and research teams. Each guide is written to be consumed independently or stitched into onboarding curricula.

## Navigating the Guides

| Guide | Audience | What you will learn |
| ----- | -------- | ------------------- |
| [Product Tour Guide](./product-tour.md) | New teammates, admins, facilitators | Walk through the in-app tour overlay, onboarding wizards, storage/sync surfaces, and admin touchpoints. |
| [Analog Foundations](./pen-and-paper.md) | Researchers, knowledge workers | Capture strands on paper, model relationships offline, convert them into collaborative notes. |
| [Modeling Without LLMs](./offline-analytics-guide.md) | Data practitioners, compliance-focused teams | Build analytical strands with statistical tooling, extractive summaries, and rule-based enrichment when AI is restricted. |
| [LLM-Augmented Workflows](./llm-augmentations.md) | Product teams, power users | Blend OpenStrand's AI services with manual curation, manage cost controls, and design responsible review loops. |
| [Metadata Architecture Playbook](./metadata-architecture.md) | Content strategists, librarians | Apply note taxonomies, metadata templates, retention policies, and authorship tracking with repeatable recipes. |
| [Developer Experience Blueprint](./dx-ux-blueprint.md) | Frontend/backend engineers, designers | Extend the UI, build SDK integrations, and align DX + UX around strands, weaves, and dashboards. |

Each guide contains:

- **Quick start** instructions (5-10 minute version) and a **deep-dive** curriculum (60+ minute).  
- **DX call-outs** explaining APIs, SDKs, and CLI tasks.  
- **UX/Research call-outs** detailing interaction design, accessibility, and testing heuristics.  
- **Operator checklists** covering RBAC, privacy, and audit logging where relevant.  

## How to Use This Suite

1. **Select the Persona** that matches your current goal. Individuals often read the analog guide first, then the metadata playbook, and finally an automation-focused guide.  
2. **Pair with Tutorials in the App.** Every Markdown guide has a mirrored walkthrough in the `/tutorials` section of the web app, enriched with interactive comps, screenshots, or embedded videos. The dashboard's **Help > Product Tour** overlay streams these docs directly so teams can follow along without leaving the canvas.  
3. **Revisit During Onboarding.** Teams typically incorporate these guides into onboarding Notion/Wiki spaces. The metadata playbook and DX blueprint are particularly useful as living documents.  

> **Tip:** When you open a doc locally you can run `npm run docs:types` inside `openstrand-app` to regenerate UI/SDK type reference tables that many guides link to.

## Related Reference Material

- `docs/Collaborative-Zettelkasten.md` - taxonomy and authorship design.  
- `docs/DEVELOPMENT_OVERVIEW.md` - monorepo, environment loader, build system.  
- `docs/ARCHITECTURE.md` - service topology (AI, caching, queue, storage).  
- `docs/LANDING_PAGE_AND_PROMPT_CHAINING_PLAN.md` - future marketing + prompt chaining roadmap.  

Continue to the [Analog Foundations guide](./pen-and-paper.md) or jump to a topic of interest.


