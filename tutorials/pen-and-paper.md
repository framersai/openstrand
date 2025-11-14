# Analog Foundations - Pen & Paper to Strand

> Carry a strand from whiteboard or notebook into collaborative OpenStrand spaces.

## Audience
## What you can do now (end-to-end)

- Capture voice notes with the floating recorder (Composer)
  - Preview -> Upload -> Server transcription (Whisper/OpenAI by default) -> Auto-insert transcript on completion
  - Controls: attach transcript, generate summary, language select (auto/en/es)

- Import sketches/photos/video with the Media Attachment Wizard
  - Drag & drop, device camera capture (Capacitor on native, capture hint on mobile web)
  - Optional OCR for images (handwriting/print) with language select
  - AI summary for media
  - Auto-embed attachments into the WYSIWYG using signed URLs

- Whiteboard handwritten notes
  - Draw, export PNG, upload with OCR + summary
  - Auto-embed in Composer as figure with caption

## Backend endpoints (Teams backend)

- List attachments
  - GET `/api/v1/strands/:strandId/attachments`
- Upload audio (voice)
  - POST `/api/v1/strands/:strandId/attachments/audio`
  - multipart fields: `file`, `attachTranscript` (bool), `generateSummary` (bool), `language` (`auto|en|...`), `durationSeconds`
- Upload media (image/video)
  - POST `/api/v1/strands/:strandId/attachments/media`
  - multipart fields: `file`, `type` (`image|video`), `generateSummary` (bool), `runOCR` (bool), `ocrLanguage` (iso)
- Transcription status / signed URL
  - GET `/api/v1/attachments/:attachmentId` -> returns `transcriptStatus` and a signed `url` (when applicable)

## Configuration

- Speech (OpenAI Whisper default)
  - `SPEECH_PROVIDER=openai`, `OPENAI_API_KEY`

- OCR (Tesseract)
  - `ENABLE_OCR=true`, `OCR_LANGUAGE=eng`, `OCR_PREPROCESSING=true`

## UX notes

- Composer shows plan-aware recording time limits; dashboards surface results with actions (Open, Open in Graph, Pin).
- Attachments are embedded as HTML figure/audio/video links; future iterations may use TipTap nodes for richer controls.


- Researchers, students, analysts who prefer jotting by hand.  
- Facilitators running workshops or user interviews.  
- UX leads documenting field studies before digitising.

## Learning Outcomes

1. Capture a main note, references, and structure outline on paper using the OpenStrand taxonomy.  
2. Model relationships offline and design co-author attribution.  
3. Digitise the strand with metadata, attachments, and RBAC-ready authorship.  
4. Optionally enhance the result with voice notes, AI summaries, and publish-ready structure notes.

## Quick Start (10 min)

1. **Index Card Setup** - label the top-left with a short slug (e.g., `23-08-workshop-insight`). Title goes top-centre, note type top-right (main/reference/structure).  
2. **Core Insight** - in the first third of the card, write the main statement in one or two sentences.  
3. **Reference Hooks** - bottom-left reserve for source citations; mark with `R1`, `R2` to convert into reference strands.  
4. **Relationship Margin** - right margin: arrows or bullet linking hints ("supports R1", "related to project/timeline").  
5. **Digitise in Composer** - open `StrandComposer`, pick note type, copy text, assign `noteType` and `coAuthorIds`, attach photo of the card if helpful.  
6. **Link Using Weave** - for every margin relationship, create a weave edge with a `note` justification.  
7. **Publish or Share** - keep private, or assign structure workspace for further collaboration.

## Deep Dive Curriculum

### 1. Analog Taxonomy Grid

| Zone | Purpose | Digital Field |
| ---- | ------- | ------------- |
| Top strip | Identifier + title + note type | `slug`, `title`, `noteType` |
| Middle pane | Narrative / insight | `content` (TipTap JSON + `plainText`) |
| Bottom strip | Source metadata, citations | `references`, `attachments` |
| Right margin | Relationship hints, tasks | `weave` edges (`note`, `createdBy`) |
| Back of card | Structure summary, follow-ups | `summary`, `activityLog`, `tasks` |

Design note: Use coloured stickers to encode authorship. During digitisation, set `createdBy`, `coAuthorIds`, and `update` actions to credit contributions.

### 2. Offline Relationship Modeling

1. On the back of the card, draw a mini knowledge graph.  
2. Use dotted lines for ideas that require validation or data verification.  
3. For team workshops, assign each participant a colour and initial; this becomes the `coAuthorIds` array later.  
4. Document questions (e.g., "Need dataset benchmark") in a separate sticky that you convert into tasks or new strands.

### 3. Digitisation Workflow

1. **Scan or Photo Upload** - capture the card image and upload via the **Media Attachment Wizard**. Attach transcript (auto or manual).  
2. **Composer Fields** - map analog sections:
   - Title -> `title`
   - Main text -> `content.data`
   - Summary/back-of-card -> `summary`
   - Relationship hints -> `weave` edges using `/strands/:id/relationships`
   - Authors -> `createdBy`, `coAuthorIds`
3. **Voice Reinforcement** - record supplemental voice notes beside the composer (floating recorder) for nuance; transcripts auto-link to attachments.
4. **Sync when ready** - once the remote URL is configured, run **Settings -> Local workspace storage -> Sync now** (or use the profile "Local storage" card) to push cached changes upstream and confirm the timestamp/conflict badges clear.

### 4. Metadata Checklist

| Metadata | Analog Source | Digital Field | Notes |
| -------- | ------------- | ------------- | ----- |
| Author | Colour/initial on card | `createdBy` | Use `coAuthorIds` for collaborators. |
| Difficulty / Tags | Circle icons or margin keywords | `metadata.tags` | Good for filtering by learning path. |
| Source reliability | Mark with star or rating | `metadata.credibility` | Map to numeric or enum. |
| Follow-up tasks | Back-of-card TODO | `metadata.tasks`, activity feed | Sync with task runners if needed. |
| Privacy sensitivity | Use red highlight | `visibility`, `teamId` gating | Ensure sensitive notes remain private.

### 5. Optional AI Enhancements

- **Tone normalisation** - feed transcript to `SpeechService` summariser for consistent voice.  
- **Extractive summarisation** - run the non-LLM pipeline (`offline-analytics-guide.md`) for regulatory environments.  
- **Rewrite/expand** - use AI prompt chaining (Teams plan) to expand the strand into a structure note or blog draft.  
- **Embedding generation** - queue vector representations for faster discovery (teams can integrate with Supabase or OpenAI embeds).

### 6. Publishing and Collaboration

1. Convert multiple main notes into a structure workspace.  
2. Use `FeatureFlag` gating so team-only features remain hidden in community builds.  
3. Publish to the knowledge base via `publisher` service or export to Markdown/PDF.  
4. Track feedback in the dashboard (`FeedbackPulsePanel`) and update the strand accordingly.

## Reference Implementations

- Frontend: `StrandComposer`, `FloatingVoiceRecorder`, `MediaAttachmentWizard`.  
- Backend: `strand.service.ts` authorship logic, `attachments.routes.ts`, `speech.service.ts`.  
- Docs: `docs/Collaborative-Zettelkasten.md` for taxonomy, `docs/DEVELOPMENT_OVERVIEW.md` for monorepo setup.  

## Next Steps

- Proceed to [Modeling Without LLMs](./offline-analytics-guide.md) to learn deterministic enrichment.  
- Explore the in-app walkthrough under `/tutorials/pen-and-paper-strand` for visual cues.  
- For team onboarding, pair this guide with the [Metadata Architecture Playbook](./metadata-architecture.md).


