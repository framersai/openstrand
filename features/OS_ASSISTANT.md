# OS Assistant ("Weaver")

Calm, contextual assistant that helps you plan, reflect, and organize — without locking you into AI. It works great with heuristics and static NLP, and optionally augments with LLMs when enabled.

## What it does

- Per-weave/loom awareness: adapts behavior if you're actively executing a project vs collecting knowledge
- Daily check-ins (all users), hourly insight passes (paid plans)
- Recommendations to the feed; natural language chat/tasks in a floating modal
- Prompts for insights, reflections, backlinks/tags, organization, and roadmap alignment

## Where it appears

- Feed (nudges): goals, recs, check-ins, insights
- Assistant modal (chat/tasks): freeform questions, step-by-step planning, structured prompts

## Settings

- User: enable/disable assistant, frequency mode (automatic/scheduled/manual), quiet hours, tone
- Admin (team/tenant): defaults, modules on/off, frequency granularity to the minute, team announcements, per-team overrides

## Scheduling

- Automatic: evaluate hourly (configurable, minute-level); send only if useful
- Scheduled: set times/day; manual: user-triggered
- Back-off and priority thresholds reduce noise

## Plans & quotas

- Daily check-ins: all plans
- Hourly insight evaluations: paid plans
- Sensible caps and rate limits by plan/team/user

## Privacy & visibility

- Private by default; admin-curated team announcements are separate and clearly labeled
- Per-user memory stored with SQL Storage Adapter; TTL + auto‑summarization and one-click clear

## Implementation notes

- Works without LLMs (heuristics + static NLP). LLM is optional plugin.
- Orchestrator service: context builder (weaves/looms, journal, streaks, quick capture), LLM wrapper, action parser, scheduler (BullMQ), delivery to feed/modal.
- Message types: recommendations, check-ins, insights; actions are suggestions only (no task model yet).

## Keyboard shortcut

- Global shortcut (configurable): open assistant modal for chat and quick actions.


