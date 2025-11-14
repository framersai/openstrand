# LLM-Augmented Strand Workflows

> Combine OpenStrand’s AI services with curated, reviewable human workflows.

## Scope

This guide assumes AI permissions are available. It explains how to:

1. Design prompt chains for summarisation, rewriting, and insight extraction.  
2. Balance cost, latency, and accuracy using the caching/job queue infrastructure.  
3. Integrate voice notes, media attachments, and metadata enrichment with AI validators.  
4. Keep humans-in-the-loop and respect auditing/rollback requirements.

## Components Involved

- **AI Service Layer** – `ai.service.ts`, provider adapters (OpenAI, Anthropic, Ollama).  
- **Prompt Chaining** – `PromptChain`, `PromptStep`, `PromptRun`, `PromptRunLog` models in Prisma schema.  
- **Queue + Cache** – Redis caching, BullMQ offline processing, fallback to in-memory.  
- **Frontend Hooks** – `useAIComposer`, `FloatingVoiceRecorder`, `MediaAttachmentWizard`, AI toggles in Dashboard.

## Architectural Diagram

```
Strand Composer ─┐
                 ├─► Prompt Chain (pre-commit analysis) ─► AI Provider
Media Wizard ────┘                    │                      │
                                      └─ logs + metadata ───┘

Jobs/Queues ► asynchronous LLM tasks (summaries, entity extraction)
Cache ► deduplicate prompts / reuse embeddings
UI ► review panes + diff viewer + revert options
```

## Prompt Chain Design

1. **Define Prompt Chain** – a collection representing the workflow (e.g., “Research Note Review”).  
2. **Add Prompt Steps** – each has a type (`summarise`, `rewrite`, `classify`, `metadata`). Include deterministic fallback text if AI unavailable.  
3. **Execute Chain** via `ai.service.runPromptChain(chainId, payload)`.  
4. **Log Output** – store tokens, provider, cost, user who initiated.  
5. **Expose Review UI** – show results alongside original text with highlight/diff.

### Step Template Example

```ts
const promptSteps: PromptStepInput[] = [
  {
    order: 1,
    label: 'Key takeaway summary',
    provider: 'openai:gpt-4o-mini',
    prompt: 'Summarise the following strand in 120 words, bullet the findings.',
    outputType: 'markdown',
  },
  {
    order: 2,
    label: 'Risk considerations',
    provider: 'anthropic:haiku',
    prompt: 'List three potential risks or caveats mentioned in the strand.',
    outputType: 'bullet-list',
  },
];
```

## Cost & Performance Controls

- **Caching**: store previous prompt inputs + outputs keyed by `hash(prompt + contentId)`. Skip duplicates.  
- **Queue Priorities**: Use BullMQ concurrency caps; e.g., high priority for summarising meeting notes, low for batch rewriting.  
- **Fallback**: If provider unreachable, revert to `offline-analytics` methods or skip gracefully with log entry.  
- **Token Controls**: Set per-plan limits; track via `CostRecord` model.

## UX Guidelines

1. **Preview Panels** – show AI output next to original. Provide accept/reject buttons.  
2. **Confidence Badges** – display provider + engine + timestamp.  
3. **Diff Visualisation** – highlight inserted/removed text for rewrites.  
4. **Audit Trail** – log `PromptRun` and attach to activity feed for accountability.  
5. **User Overrides** – allow manual edits to become authoritative; persist them as new `PromptRun` with `override` flag.

## Metadata Enrichment

- **Entities** – run NER to extract people, organisations, locations; store under `metadata.entities`.  
- **Taxonomy Suggestion** – propose note types or tags by evaluating strand content.  
- **Sentiment / Tone** – apply optional sentiment scoring.  
- **Action Items** – transform transcripts into task lists; push to `metadata.tasks`.

## Voice & Media Attachments

1. Voice notes are transcribed via `SpeechService` (OpenAI Whisper or custom).  
2. LLM summarises transcripts, generating meeting highlights.  
3. Media wizard can request ALT text, object detection, and captions.  
4. Use `metadata.mediaInsights` to store detection results.

## Collaboration & RBAC

- **Role-based gating** – use `FeatureFlag` and `StrandPermission` to restrict AI actions to editors.  
- **Review queue** – create `prompt_review` events that require approval from senior editors before merging.  
- **Transparency** – show `createdBy`, `updatedBy`, `PromptRunLog` details.

## Developer Notes

- **SDK** – extend `openstrand-sdk` to expose `runPromptChain` for automation scripts.  
- **Testing** – use mocked providers for unit tests; assert fallback path when provider disabled.  
- **Docs** – update `docs/DEVELOPMENT_OVERVIEW.md` when new providers added.  
- **Monitoring** – log provider latency, capture errors in `PromptRunLog`.

## Rollout Checklist

1. Configure providers + env vars (API keys, rate limits).  
2. Migrate database to include prompt chain models.  
3. Wire frontend to show review UI with manual override.  
4. Write runbooks covering failover, cost audits, and data retention policies.

## Next Steps

- Pair with [Metadata Architecture Playbook](./metadata-architecture.md) to ensure AI outputs align with taxonomies.  
- Revisit [Developer Experience Blueprint](./dx-ux-blueprint.md) for integrating AI toggles seamlessly into dashboards and editors.


