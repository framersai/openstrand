# Frontend Type Reference

> Generated from `src/types/openstrand.ts` using `frontend/scripts/generate-types-docs.js`.

## Type `AccessRole`
```ts
type AccessRole = 'owner' | 'editor' | 'commenter' | 'viewer';
```

## Interface `AIArtisanQuota`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `remaining` | `number` | No |  |
| `limit` | `number` | No |  |
| `plan` | `string` | No |  |
| `resets_at` | `string` | Yes |  |

## Interface `AIArtisanResult`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `code` | `{ html?: string; css?: string; js: string; }` | No |  |
| `sandboxConfig` | `{ libraries: string[]; sandbox: string[]; csp: string; }` | No |  |
| `cost` | `number` | No |  |
| `generationTime` | `number` | No |  |
| `modelUsed` | `string` | No |  |

## Type `AnalysisStatus`
```ts
type AnalysisStatus = | 'not_started'
  | 'queued'
  | 'in_progress'
  | 'completed'
  | 'failed';
```

## Interface `AssessmentData`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `questions` | `Record<string, unknown>[]` | No |  |
| `answer_key` | `Record<string, unknown>` | Yes |  |
| `rubric` | `Record<string, unknown>` | Yes |  |
| `auto_grade` | `boolean` | No |  |
| `attempts_allowed` | `number` | No |  |

## Interface `CapabilityMatrix`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `analysisPipeline` | `boolean` | No |  |
| `documentAnalysis` | `boolean` | No |  |
| `mediaAnalysis` | `boolean` | No |  |
| `dynamicVisualizations` | `boolean` | No |  |
| `generativeVisualizations` | `boolean` | No |  |
| `topContent` | `boolean` | No |  |
| `aiArtisan` | `boolean` | No |  |
| `knowledgeGraph` | `boolean` | No |  |

## Interface `ContentEnhancement`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `strand_id` | `string` | No |  |
| `suggestions` | `string[]` | No |  |
| `learning_objectives` | `LearningObjective[]` | No |  |
| `prerequisites` | `string[]` | No |  |
| `related_concepts` | `string[]` | No |  |
| `difficulty_analysis` | `Record<string, unknown>` | No |  |
| `estimated_reading_time` | `number` | No |  |
| `key_takeaways` | `string[]` | No |  |

## Interface `DailySchedule`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `date` | `string` | No |  |
| `user_id` | `string` | No |  |
| `items` | `ScheduleItem[]` | No |  |
| `total_duration` | `number` | No |  |
| `review_items` | `ScheduleItem[]` | No |  |
| `new_items` | `ScheduleItem[]` | No |  |
| `practice_items` | `ScheduleItem[]` | No |  |

## Interface `DocumentAnalysis`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `status` | `AnalysisStatus` | No |  |
| `provider` | `string` | Yes |  |
| `pages` | `DocumentPageSummary[]` | No |  |
| `outline` | `string[]` | No |  |
| `detected_languages` | `string[]` | No |  |
| `entities` | `Record<string, unknown>[]` | No |  |
| `embeddings_generated` | `boolean` | No |  |
| `last_analyzed_at` | `string` | Yes |  |

## Interface `DocumentPageSummary`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `page_number` | `number` | No |  |
| `summary` | `string` | Yes |  |
| `keywords` | `string[]` | No |  |
| `thumbnail_url` | `string` | Yes |  |

## Interface `GrantPermissionPayload`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `principal_id` | `string` | No |  |
| `principal_type` | `PrincipalType` | No |  |
| `role` | `AccessRole` | No |  |
| `expires_at` | `string` | Yes |  |

## Interface `LearningObjective`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `objective` | `string` | No |  |
| `bloom_level` | `"remember" | "understand" | "apply" | "analyze" | "evaluate" | "create"` | No |  |
| `measurable` | `boolean` | No |  |

## Interface `LearningProgress`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `user_id` | `string` | No |  |
| `strand_id` | `string` | No |  |
| `phase` | `LearningPhase` | No |  |
| `mastery_level` | `number` | No |  |
| `total_time_spent` | `number` | No |  |
| `review_count` | `number` | No |  |
| `last_accessed` | `string` | No |  |
| `sm2_item` | `Record<string, unknown>` | Yes |  |
| `scaffold_level` | `string` | No |  |

## Interface `MediaAnalysis`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `status` | `AnalysisStatus` | No |  |
| `provider` | `string` | Yes |  |
| `transcript_text` | `string` | Yes |  |
| `transcript_url` | `string` | Yes |  |
| `scenes` | `MediaScene[]` | No |  |
| `keyframes` | `MediaKeyframe[]` | No |  |
| `audio_waveform_url` | `string` | Yes |  |
| `detected_objects` | `string[]` | No |  |
| `last_analyzed_at` | `string` | Yes |  |

## Interface `MediaContent`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `url` | `string` | No |  |
| `type` | `string` | No |  |
| `size` | `number` | Yes |  |
| `duration` | `number` | Yes |  |
| `dimensions` | `{ width: number; height: number; }` | Yes |  |
| `thumbnail` | `string` | Yes |  |
| `transcript` | `string` | Yes |  |
| `alt_text` | `string` | Yes |  |

## Interface `MediaKeyframe`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `timestamp` | `number` | No |  |
| `description` | `string` | Yes |  |
| `thumbnail_url` | `string` | Yes |  |

## Interface `MediaScene`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `start_time` | `number` | No |  |
| `end_time` | `number` | No |  |
| `summary` | `string` | Yes |  |
| `tags` | `string[]` | No |  |

## Interface `OpenStrandStore`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `strands` | `Strand[]` | No |  |
| `currentStrand` | `Strand` | No |  |
| `weave` | `Weave` | No |  |
| `schedule` | `DailySchedule` | No |  |
| `progress` | `Record<string, LearningProgress>` | No |  |
| `permissions` | `Record<string, StrandPermission[]>` | No |  |
| `qualitySnapshots` | `Record<string, QualityMatrix>` | No |  |
| `capabilities` | `CapabilityMatrix` | No |  |
| `topVisualizations` | `Strand[]` | No |  |
| `topDatasets` | `Strand[]` | No |  |
| `artisanQuota` | `AIArtisanQuota` | No |  |
| `loading` | `boolean` | No |  |
| `error` | `string` | No |  |
| `loadStrands` | `(filters?: Record<string, unknown>) => Promise<void>` | No |  |
| `loadStrand` | `(id: string) => Promise<void>` | No |  |
| `createStrand` | `(strand: Partial<Strand>) => Promise<void>` | No |  |
| `updateStrand` | `(id: string, updates: Partial<Strand>) => Promise<void>` | No |  |
| `deleteStrand` | `(id: string) => Promise<void>` | No |  |
| `uploadContent` | `(file: File, metadata?: Record<string, unknown>) => Promise<void>` | No |  |
| `loadWeave` | `(filters?: Record<string, unknown>) => Promise<void>` | No |  |
| `findPath` | `(source: string, target: string) => Promise<void>` | No |  |
| `getRecommendations` | `() => Promise<void>` | No |  |
| `loadSchedule` | `(date?: string) => Promise<void>` | No |  |
| `recordProgress` | `(strandId: string, quality: number, timeSpent: number) => Promise<void>` | No |  |
| `enhanceContent` | `(strandId: string) => Promise<ContentEnhancement>` | No |  |
| `loadPermissions` | `(strandId: string) => Promise<StrandPermission[]>` | No |  |
| `grantPermission` | `(strandId: string, payload: GrantPermissionPayload) => Promise<void>` | No |  |
| `revokePermission` | `(strandId: string, permissionId: string) => Promise<void>` | No |  |
| `createShareLink` | `(strandId: string, role: AccessRole, expires_at?: string) => Promise<ShareLinkResponse>` | No |  |
| `loadQuality` | `(strandId: string) => Promise<QualityMatrix>` | No |  |
| `submitQualityVote` | `(strandId: string, payload: QualityVotePayload) => Promise<QualityMatrix>` | No |  |
| `refreshQuality` | `(strandId: string) => Promise<void>` | No |  |
| `loadCapabilities` | `() => Promise<CapabilityMatrix>` | No |  |
| `loadArtisanQuota` | `() => Promise<AIArtisanQuota>` | No |  |
| `loadTopVisualizations` | `(limit?: number) => Promise<Strand[]>` | No |  |
| `loadTopDatasets` | `(limit?: number) => Promise<Strand[]>` | No |  |
| `setError` | `(error: string) => void` | No |  |
| `clearError` | `() => void` | No |  |

## Interface `Pattern`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `name` | `string` | No |  |
| `description` | `string` | No |  |
| `selection` | `Record<string, unknown>` | No |  |
| `ordering` | `"topological" | "difficulty" | "explicit" | "chronological"` | No |  |
| `decoration` | `Record<string, unknown>` | No |  |
| `editions` | `string[]` | No |  |
| `validation` | `Record<string, unknown>` | No |  |
| `created` | `string` | No |  |
| `modified` | `string` | No |  |

## Interface `Prerequisite`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `level` | `PrerequisiteLevel` | No |  |
| `title` | `string` | Yes |  |
| `reason` | `string` | Yes |  |

## Type `PrincipalType`
```ts
type PrincipalType = 'user' | 'team' | 'link';
```

## Interface `QualityMatrix`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `llm_rating` | `number` | Yes |  |
| `human_ratings` | `Record<number, number>` | No |  |
| `composite_score` | `number` | Yes |  |
| `confidence` | `number` | Yes |  |
| `last_updated` | `string` | Yes |  |

## Interface `QualityVotePayload`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `rating` | `number` | No |  |
| `weight` | `number` | Yes |  |
| `rationale` | `string` | Yes |  |

## Interface `Relationship`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `target_id` | `string` | No |  |
| `type` | `RelationshipType` | No |  |
| `weight` | `number` | No |  |
| `description` | `string` | Yes |  |
| `metadata` | `Record<string, unknown>` | Yes |  |

## Interface `RepresentationalVariant`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `mode` | `RepresentationMode` | No |  |
| `content` | `Record<string, unknown>` | No |  |
| `media_type` | `string` | Yes |  |

## Interface `ScaffoldVariant`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `level` | `ScaffoldLevel` | No |  |
| `content` | `Record<string, unknown>` | No |  |
| `hints` | `string[]` | No |  |
| `examples` | `Record<string, unknown>[]` | No |  |

## Interface `ScheduleItem`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `strand_id` | `string` | No |  |
| `user_id` | `string` | No |  |
| `scheduled_for` | `string` | No |  |
| `type` | `"review" | "new" | "practice"` | No |  |
| `status` | `"in_progress" | "completed" | "scheduled" | "skipped" | "overdue"` | No |  |
| `phase` | `LearningPhase` | No |  |
| `estimated_duration` | `number` | No |  |
| `actual_duration` | `number` | Yes |  |
| `quality` | `number` | Yes |  |
| `notes` | `string` | Yes |  |
| `completed_at` | `string` | Yes |  |
| `sm2_data` | `Record<string, unknown>` | Yes |  |

## Interface `ShareLinkResponse`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `token` | `string` | No |  |
| `role` | `AccessRole` | No |  |
| `expires_at` | `string` | Yes |  |

## Interface `SpiralMetadata`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `stage` | `number` | No |  |
| `revisit_windows` | `number[]` | No |  |
| `interleave_with` | `string[]` | No |  |
| `mastery_threshold` | `number` | No |  |
| `estimated_time` | `number` | No |  |

## Interface `Strand`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `type` | `StrandType` | No |  |
| `title` | `string` | No |  |
| `slug` | `string` | No |  |
| `summary` | `string` | No |  |
| `created` | `string` | No |  |
| `modified` | `string` | No |  |
| `difficulty` | `Difficulty` | Yes |  |
| `learning_objectives` | `(string | LearningObjective)[]` | No |  |
| `prerequisites` | `Prerequisite[]` | No |  |
| `spiral_metadata` | `SpiralMetadata` | Yes |  |
| `scaffold_variants` | `ScaffoldVariant[]` | No |  |
| `representational_variants` | `RepresentationalVariant[]` | No |  |
| `viz_config` | `VisualizationConfig` | Yes |  |
| `assessment_data` | `AssessmentData` | Yes |  |
| `content` | `StrandContent` | No |  |
| `metadata` | `StrandMetadata` | No |  |
| `relationships` | `Relationship[]` | No |  |
| `analysis_status` | `AnalysisStatus` | No |  |
| `analysis_provider` | `string` | Yes |  |
| `document_analysis` | `DocumentAnalysis` | Yes |  |
| `media_analysis` | `MediaAnalysis` | Yes |  |
| `analysis_notes` | `string[]` | No |  |
| `derivatives` | `string[]` | No |  |
| `visibility` | `"public" | "private" | "unlisted" | "premium"` | No |  |
| `owner_id` | `string` | Yes |  |
| `permissions` | `StrandPermission[]` | No |  |
| `shared_with` | `string[]` | No |  |
| `quality` | `QualityMatrix` | No |  |
| `views` | `number` | No |  |
| `likes` | `number` | No |  |
| `completion_rate` | `number` | No |  |

## Interface `StrandContent`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `markdown` | `string` | Yes |  |
| `html` | `string` | Yes |  |
| `data` | `unknown` | Yes |  |
| `media` | `MediaContent` | Yes |  |
| `code` | `Record<string, string>` | Yes |  |
| `metadata` | `Record<string, unknown>` | No |  |

## Interface `StrandMetadata`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `author` | `string` | Yes |  |
| `source` | `string` | Yes |  |
| `license` | `string` | Yes |  |
| `language` | `string` | No |  |
| `tags` | `string[]` | No |  |
| `keywords` | `string[]` | No |  |
| `concepts` | `string[]` | No |  |
| `version` | `string` | No |  |
| `citations` | `string[]` | No |  |

## Interface `StrandPermission`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `principal_id` | `string` | No |  |
| `principal_type` | `PrincipalType` | No |  |
| `role` | `AccessRole` | No |  |
| `granted_by` | `string` | Yes |  |
| `granted_at` | `string` | No |  |
| `expires_at` | `string` | Yes |  |
| `share_token` | `string` | Yes |  |

## Interface `Thread`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `title` | `string` | No |  |
| `slug` | `string` | No |  |
| `description` | `string` | No |  |
| `strand_order` | `string[]` | No |  |
| `intro` | `string` | Yes |  |
| `outro` | `string` | Yes |  |
| `metadata` | `StrandMetadata` | No |  |
| `created` | `string` | No |  |
| `modified` | `string` | No |  |
| `nav_level` | `"part" | "chapter" | "section" | "lesson"` | No |  |
| `include_in_toc` | `boolean` | No |  |

## Interface `VisualizationConfig`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `type` | `string` | No |  |
| `spec` | `Record<string, unknown>` | No |  |
| `data_source` | `string` | Yes |  |
| `interactive` | `boolean` | No |  |
| `remixable` | `boolean` | No |  |
| `kind` | `VisualizationKind` | Yes |  |
| `prompt` | `string` | Yes |  |
| `style_preset` | `string` | Yes |  |
| `provider` | `string` | Yes |  |
| `source_strand_id` | `string` | Yes |  |

## Interface `Weave`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `name` | `string` | No |  |
| `domain` | `string` | No |  |
| `nodes` | `WeaveNode[]` | No |  |
| `edges` | `WeaveEdge[]` | No |  |
| `metrics` | `{ centrality?: Record<string, number>; communities?: string[][]; density?: number; diameter?: number; }` | Yes |  |
| `communities` | `string[][]` | Yes |  |
| `created` | `string` | No |  |
| `modified` | `string` | No |  |

## Interface `WeaveEdge`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `source` | `string` | No |  |
| `target` | `string` | No |  |
| `type` | `string` | No |  |
| `weight` | `number` | No |  |
| `metadata` | `Record<string, unknown>` | Yes |  |

## Interface `WeaveNode`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `id` | `string` | No |  |
| `type` | `string` | No |  |
| `title` | `string` | No |  |
| `importance` | `number` | No |  |
| `summary` | `string` | Yes |  |
| `metadata` | `Record<string, unknown>` | Yes |  |

## Interface `WeaveViewerData`
| Property | Type | Optional | Description |
| --- | --- | --- | --- |
| `nodes` | `{ id: string; type: string; title: string; importance: number; summary?: string; }[]` | No |  |
| `edges` | `{ source: string; target: string; type: string; weight: number; }[]` | No |  |
| `metrics` | `{ centrality: Record<string, number>; communities: string[][]; density: number; }` | Yes |  |
