# Data Intelligence Marketing & Documentation Guide

## 🎯 Key Marketing Messages

### Headline: "Build Knowledge Without Waiting for AI"

**Subheadline:** Instant vocabulary extraction, entity recognition, and summarization that works offline. Teams can add optional LLM verification.

---

## 🚀 Landing Page Feature Tiles

### 1. Data Intelligence (Core Feature)
**Icon:** Brain  
**Title:** Data Intelligence  
**Description:** Build vocabularies, extract entities, and summarize content with deterministic NLP. Works offline with TF/IDF, NER, and TextRank. Teams can add optional LLM verification.  
**Badge:** Core  
**Learn More Link:** `/features/data-intelligence`

### 2. Multi-Loom Workspaces (Teams Feature)
**Icon:** Network  
**Title:** Multi-Loom Workspaces  
**Description:** Teams Edition: Create unlimited project workspaces (Looms) for storytelling, world-building, research, or custom use cases. Community: One powerful global Loom.  
**Badge:** Team  
**Learn More Link:** `/features/looms`

---

## 📝 Feature Page Content

### /features/data-intelligence

#### Hero Section
**Title:** Turn Documents into Structured Knowledge—Instantly

**Lead Paragraph:**
OpenStrand's Data Intelligence extracts vocabularies, identifies entities, and summarizes content using battle-tested algorithms that work 100% offline. No waiting for AI, no API costs, no privacy concerns.

#### How It Works

1. **Upload Documents**
   - Drop any text, markdown, or import strands
   - Supports batch processing for entire collections

2. **Instant Analysis**
   - TF/IDF term ranking finds important concepts
   - Named Entity Recognition identifies people, organizations, locations
   - Bigram extraction reveals common phrases
   - TextRank summarization extracts key sentences

3. **Review & Refine**
   - Interactive vocabulary explorer
   - Entity relationship graphs
   - Export to JSON, CSV, or knowledge graph

#### Technical Details

**Deterministic NLP Stack:**
- **TF/IDF:** Term frequency–inverse document frequency for importance scoring
- **NER:** wink-nlp for production extraction, regex fallback for browsers
- **Summarization:** TextRank (PageRank on sentence similarity graphs)
- **Languages:** English (more coming soon)

**Performance:**
- Vocabulary extraction: ~45ms for 10 documents
- Entity recognition: ~23ms for 10KB text
- Summarization: ~150ms for 5KB document
- All processing happens locally—zero network latency

#### Edition Comparison

| Feature | Community | Teams |
|---------|-----------|--------|
| Vocabulary extraction | ✓ Unlimited | ✓ Unlimited |
| Entity recognition | ✓ All types | ✓ All types |
| Extractive summarization | ✓ TextRank | ✓ TextRank |
| Background processing | ✓ 1 job | ✓ 1-20 concurrent |
| Analysis triggers | Immediate/Manual | All modes + scheduled |
| LLM verification | ✗ | ✓ Optional |
| Batch windows | Fixed 15 min | 5-120 min configurable |
| Cache control | Fixed 60 min | 10-1440 min configurable |
| API access | ✓ Read-only | ✓ Full CRUD |

---

## 🎓 Tutorial Content

### Interactive Tour Steps

#### 1. Vocabulary Builder
**Target:** `.data-intelligence-button`  
**Content:** Build deterministic vocabularies from your strands using TF/IDF term ranking, bigram extraction, and named entity recognition. Works 100% offline!  
**Tips:**
- Extracts key terms using TF/IDF scoring
- Finds people, organizations, locations automatically  
- Works offline - no LLM required
- Teams can add optional LLM verification

#### 2. Smart Summarization
**Content:** Use TextRank algorithm to extract the most important sentences from your documents.  
**Example:** Select any strand and click "Summarize" to see the top 5 sentences ranked by importance.

#### 3. Loom Workspaces
**Target:** `.loom-switcher`  
**Content:** Teams Edition: Create unlimited project workspaces for different use cases. Community Edition: Everything in one powerful global Loom.  
**Tips:**
- Storytelling: Novel projects, screenplays
- World-building: Fantasy/sci-fi universes
- Research: Academic papers, citations
- Custom: Your own workflow

---

## 🗣️ Sales Talking Points

### For Individuals (Community Edition)
"Get professional-grade text analysis without the professional price tag. Extract vocabularies, find entities, and summarize documents—all running locally on your device. No subscriptions, no API limits, yours forever."

### For Teams
"Give your entire team AI-powered knowledge extraction that scales. Process thousands of documents in parallel, enforce team-wide policies, and optionally verify results with LLMs. One-time $1,000 investment (or $250 for students) includes lifetime updates."

### For Enterprises
"Deploy deterministic NLP across your organization with full control. Air-gapped deployments supported, no data leaves your infrastructure, predictable processing costs. Contact us for volume licensing."

---

## 📊 Performance Benchmarks

### Speed Comparison
| Operation | Our Stack | GPT-4 API | Claude API |
|-----------|-----------|-----------|------------|
| 10-doc vocabulary | 45ms | 3,000ms | 2,500ms |
| Entity extraction | 23ms | 1,500ms | 1,200ms |
| Summarization | 150ms | 2,000ms | 1,800ms |
| **Cost** | **$0** | $0.30 | $0.25 |

### Accuracy Notes
- Deterministic NLP: 85-90% accuracy on standard benchmarks
- LLM verification (Teams): Brings accuracy to 95-98%
- Trade-off: Instant + free vs. slower + paid

---

## 🎬 Demo Script

### 30-Second Elevator Pitch
"OpenStrand turns your documents into structured knowledge instantly. Drop in any text, and our offline NLP extracts vocabularies, finds entities, and summarizes content in milliseconds. No waiting for AI, no API costs, no privacy concerns. Teams can add optional LLM verification for critical documents."

### Live Demo Flow (3 minutes)

1. **Upload** (30s)
   - Drag folder of research papers
   - Show instant import with metadata

2. **Analyze** (60s)
   - Click "Build Vocabulary"
   - Show live term extraction
   - Hover over entities to see types
   - Generate summary of collection

3. **Visualize** (60s)
   - Open knowledge graph view
   - Show entity relationships
   - Filter by term importance
   - Export vocabulary as JSON

4. **Teams Features** (30s)
   - Switch between Looms
   - Show team-wide vocabulary
   - Enable LLM verification toggle

---

## 🔧 Implementation Checklist

### Landing Page Updates
- [x] Added Data Intelligence to features section
- [x] Added Multi-Loom Workspaces to features section  
- [x] Updated hero highlights with NLP badge
- [ ] Create dedicated feature pages
- [ ] Add performance comparison chart
- [ ] Include demo video/GIF

### Documentation Updates
- [x] API Reference includes all 15 new endpoints
- [x] NLP & Summarization technical guide created
- [x] Pricing matrix updated with edition differences
- [x] Data Intelligence implementation guide
- [ ] User guide for vocabulary builder
- [ ] Video tutorials

### In-App Updates
- [x] Interactive tour includes NLP steps
- [x] Onboarding wizard has data intelligence step
- [x] Settings page wired to backend
- [x] Loom switcher connected to API
- [ ] Vocabulary visualization component
- [ ] Real-time job progress UI

### Admin Panel Updates
- [x] Data Intelligence admin section created
- [x] Sidebar navigation updated
- [x] Global defaults configuration
- [x] Team policy management
- [ ] Job monitoring dashboard
- [ ] Cost tracking integration

---

## 📱 Social Media Copy

### Twitter/X Thread
```
1/ 🧵 Tired of waiting for AI to analyze your documents? OpenStrand now extracts vocabularies, finds entities, and summarizes content INSTANTLY—all offline.

2/ Our deterministic NLP stack:
• TF/IDF for term importance
• wink-nlp for entity recognition  
• TextRank for summarization
• 100% local, 0% API costs

3/ Community Edition: Full NLP power, free forever
Teams Edition: Add parallel processing + optional LLM verification

Launch pricing: $1,000 lifetime (normally $100/year after launch)
Students: $250 with .edu email

Try it now: openstrand.com
```

### LinkedIn Post
```
Introducing Data Intelligence in OpenStrand 🧠

We've integrated professional-grade NLP that runs entirely offline:

✅ Vocabulary extraction with TF/IDF scoring
✅ Named entity recognition (people, orgs, locations)  
✅ Extractive summarization with TextRank
✅ Zero API dependencies or costs

Perfect for researchers, writers, and teams who need instant text analysis without compromising privacy or breaking the bank.

Community Edition: Free forever with full NLP features
Teams Edition: $1,000 lifetime (launch special)

See it in action: openstrand.com/features/data-intelligence
```

---

## 🎯 SEO Keywords

### Primary Keywords
- offline text analysis
- local NLP tools
- vocabulary extraction software
- entity recognition without AI
- deterministic text summarization
- privacy-first knowledge management

### Long-tail Keywords
- extract entities from documents offline
- build vocabulary from research papers
- summarize text without GPT
- local named entity recognition
- TF/IDF term extraction tool
- TextRank summarization algorithm

### Meta Descriptions
**Homepage:** "OpenStrand - Offline knowledge management with instant vocabulary extraction, entity recognition, and summarization. No AI waiting, no API costs. Free Community Edition."

**Data Intelligence Page:** "Extract vocabularies, identify entities, and summarize documents instantly with OpenStrand's offline NLP. Works without internet, costs nothing to run. Try free."
