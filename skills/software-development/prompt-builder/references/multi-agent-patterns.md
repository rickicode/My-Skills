# Multi-Agent System Patterns for Prompt Builder

## Pattern 1: Generator-Judge with Scoring Loop

**Use case:** PRD generation, code review, content creation where quality matters.

```
User Input → Brainstorming Agent → Context Document
                                          ↓
                                    Generator Agent → Output v1
                                          ↓
                                      Judge Agent → Score + Critiques
                                          ↓
                                    Score ≥ threshold? → YES → Review UI → Approve
                                          ↓ NO
                                    Split critiques by type:
                                    ├─ OBJECTIVE (answerable from context) → Auto-Revision
                                    └─ AMBIGUOUS (needs user input) → Ask User → User Answers
                                          ↓
                                    Generator Agent → Output v2 (revised)
                                          ↓
                                    Judge Agent → Re-evaluate → Loop
```

**Key decisions:**
- Score threshold: typically 7/10 (configurable)
- Max iterations: plan-dependent (free=1, paid=5+)
- Critique classification prevents unnecessary user interruptions
- Auto-revision for objective issues keeps flow smooth

**XML representation in prompt:**
```xml
<agents>
  <agent id="brainstorming" role="context_gathering" />
  <agent id="generator" role="output_creation" />
  <agent id="judge" role="quality_evaluation" />
</agents>
<flow>
  brainstorming → generator → judge → {score ≥ 7? → done : revise → judge}
</flow>
```

---

## Pattern 2: Multi-Step Pipeline

**Use case:** Data processing, ETL, content transformation.

```
Input → Step 1 (validate) → Step 2 (transform) → Step 3 (enrich) → Output
```

**When to use:** Linear flow with clear stages, no feedback loops needed.

---

## Pattern 3: Debate/Consensus

**Use case:** Complex decisions, risk analysis, architecture choices.

```
Input → Agent A (pro) ─┐
                       ├→ Aggregator → Consensus Output
Input → Agent B (con) ─┘
```

**When to use:** When multiple perspectives improve quality. Each agent argues a position, aggregator finds consensus.

---

## Pattern 4: Specialist Routing

**Use case:** Multi-domain tasks (full-stack apps, multi-service systems).

```
Input → Router Agent → Domain A Specialist → Output A
                     → Domain B Specialist → Output B
                     → Domain C Specialist → Output C
                                           ↓
                                    Assembler Agent → Final Output
```

**When to use:** Task has distinct domains that benefit from specialized handling.

---

## Pattern 5: Iterative Refinement with Memory

**Use case:** Learning systems, personalization, long-term improvement.

```
Input → Agent → Output → Feedback → Memory Store
                                  ↓
Next Input → Agent (with memory context) → Better Output
```

**Key:** Memory store (mem0, vector DB, etc.) captures patterns from past interactions.
Each new input benefits from accumulated knowledge.

---

## Common Agent Capabilities

| Capability | Tools | Safety Guard |
|-----------|-------|-------------|
| Web search | exa, pinchtab | Domain whitelist, rate limit |
| Documentation lookup | reftools, pinchtab | Content filter |
| HTTP requests | curl | Purpose logging, data sanitization |
| File operations | terminal, read_file | Path validation, no system files |
| Memory read/write | mem0, vector DB | PII filter, no raw transcripts |

---

## Pattern 6: Auto-Blog Pipeline (Writer → Judge → SEO + Enrichment)

**Use case:** Automated content generation from keywords — auto-blogging platforms, content farms, SEO blogs.

**Concrete implementation:** AI auto-blogging platform with 3 agents + enrichment steps.

```
Keyword + Config (tone, length, target blog)
      ↓
  AI Writer → Full article draft (Markdown)
      ↓
  AI Judge → Naturalness review (score 1-10)
      ↓ NO (score < 7) ──→ Feedback to Writer → Revise → Judge (max 3 retries)
      ↓ YES (score ≥ 7)
  AI SEO → Meta title, description, slug, JSON-LD schema
      ↓
  Image Fetcher → DuckDuckGo image search → Download relevant images
      ↓
  Auto Categorizer → Assign/create category (with semantic dedup)
      ↓
  Publish Gate:
  ├─ Auto mode → Publish immediately → Frontend picks up
  └─ Review mode → Save as draft → Admin reviews → Approve/Reject
```

**Key decisions:**
- Judge score threshold: 7/10 (configurable per blog)
- Max retry: 3 (after 3 rejections, flag for manual review)
- AI naturalness enforcement: Judge must detect common AI patterns ("In today's world...", "Let's dive in...", uniform sentence structure)
- Image sourcing: DuckDuckGo (no API key needed), fallback to no-image
- Category dedup: semantic similarity check before creating new categories
- Provider fallback: Provider A → Provider B on failure → manual flag

**Multi-provider fallback chain:**
```
Primary Provider (e.g., OpenAI) → on HTTP error/rate limit →
Secondary Provider (e.g., Anthropic via OpenRouter) → on failure →
Flag for manual review
```

**XML representation in prompt:**
```xml
<ai_pipeline>
  <providers>
    <provider id="primary" api_type="openai-compatible" env_prefix="AI_PROVIDER_1" />
    <provider id="secondary" api_type="openai-compatible" env_prefix="AI_PROVIDER_2" />
  </providers>
  <agents>
    <agent id="writer" role="content_generation" provider="primary|fallback" />
    <agent id="judge" role="naturalness_review" provider="primary|fallback" />
    <agent id="seo" role="metadata_generation" provider="primary|fallback" />
  </agents>
  <flow>
    keyword → writer → judge → {score ≥ 7? → seo → enrich : revise (max 3) → judge}
  </flow>
  <enrichment>
    <step name="image_fetch" source="duckduckgo" />
    <step name="categorize" dedup="semantic_similarity" />
  </enrichment>
  <publish_gate modes="auto|review" per_blog="true" />
</ai_pipeline>
```

**When to use:** Any system where content is generated automatically from inputs and must read as human-written. The Judge agent is critical — without it, AI-generated content feels robotic and detectable.
