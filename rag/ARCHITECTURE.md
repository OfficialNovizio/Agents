# YVON RAG — Architecture & Scaling Model

## The Core Design Decision

Most RAG systems split documents by token count. Every 512 tokens, cut. This produces chunks that start mid-sentence with no heading context. In our domain, where documents follow strict schemas (9-section SKILL.md, 6-section agent.md), this is catastrophic. A chunk containing only the second half of `## Principles` loses the heading that tells the retriever it IS principles.

**Our approach:** The chunker understands document types. It splits at heading boundaries and preserves section hierarchy, agent assignment, department context, and priority tier as structured metadata.

## Document Types & Their Structures

### SKILL.md (custom + marketplace) — 9 sections
```
Introduction | Purpose | When to Use | Structure/Protocol | Instructions
| Output Format | Principles | Fallback | Boundaries
```
~350 files in Teams/. Average 3-8 KB.

### agent.md — 6 sections
```
Summary | Purpose | Position in the Org | Skill Roster
| Identity/Operational/Logical Status | Workflow Structure
```
46 files. Average 1-3 KB.

### logical/book-requirements.md — 5 sections
```
Purpose | Scripts Table | Inherited Scripts
| Flag Clearance Summary | Skills → Script Mapping
```
46 files. Average 2-4 KB.

### DEPARTMENT-WORKFLOW.md — ~8 sections
```
Summary | Purpose | Working Structure | Working Tree
| Working Instructions | Department Status | Cross-Department
```
7 files. Average 6-12 KB.

### Operational subfolder files — variable
```
principles.md | commands.md | agent-config.md | skill-routing.md | tool-requirements.md
```
~230 files. Average 1-3 KB.

### Route D wisdom — variable
```
Structured frameworks with extensive prose + citations
```
7 files in Shared OS/logical/. Average 15-40 KB.

### Python scripts — not chunked
```
Formula libraries with embedded docstrings as documentation.
We embed docstrings per function, not the full script.
```
35 files in Shared OS/logical/. Not chunked — function-level extracted.

## Chunking Strategy

| Document Type | Chunking Method | Avg Chunks per File |
|--------------|----------------|---------------------|
| SKILL.md | Per heading section (# and ##) | 8-10 |
| agent.md | Per heading section | 5-7 |
| book-requirements.md | Per heading section + table compression | 4-6 |
| DEPARTMENT-WORKFLOW.md | Per heading section | 6-10 |
| Operational | Per file (already small) | 1 |
| Route D wisdom | Per heading section (# and ##) | 15-30 |
| Python scripts | Per function docstring | 5-20 per script |

**Estimated total chunks: 4,000-5,500** across 651 files.

## Chunk Metadata

Every chunk carries:

```json
{
  "chunk_id": "exec-office/marcus/custom/decision-critic/principles",
  "source_file": "Teams/Executive Office/marcus/custom/decision-critic/SKILL.md",
  "section": "Principles",
  "parent_heading": "## Principles",
  "depth": 2,
  "department": "Executive Office",
  "assigned_agents": ["marcus"],
  "document_type": "SKILL.md",
  "priority_tier": 1,
  "char_count": 842,
  "toon_available": true,
  "last_modified": "2026-07-08T14:22:00Z",
  "chunk_text": "The full text of this section...",
  "toon_text": "pr:no fabrication · escalate close calls...",
  "references": [
    {"type": "script", "name": "capital_budgeting.py"},
    {"type": "skill", "name": "okr-cascade"}
  ]
}
```

## Priority Tiers

| Tier | Description | Max Budget Share |
|------|-------------|-----------------|
| 1 | Load-bearing — principles, gate rules, formulas | 50% |
| 2 | Structural — instructions, protocols, workflows | 30% |
| 3 | Supplementary — examples, notes, templates | 20% |

## Retrieval Cascade

At query time, the retrieval cascade operates in four tiers:

```
Tier 1 (5ms): Metadata pre-filter
  → WHERE tenant_id = 'x' AND (department IN (...) OR assigned_agents LIKE '%agent%')
  → Reduces from 5,000 chunks to ~300

Tier 2 (10ms): Dense vector search
  → Cosine similarity on filtered set with local ONNX embeddings
  → Returns top-40 candidates

Tier 3 (15ms): Sparse BM25 scoring
  → Computed ONLY on the 40 candidates, not all 300
  → Combined dense+sparse score returns top-20

Tier 4 (50ms): Cross-encoder re-rank
  → 20 × ~2.5ms per comparison on CPU
  → Returns top-8 for injection

Total: ~80ms at query time regardless of corpus size
```

## Scaling Model — Four-Tier Data Architecture

```
TIER 1: GLOBAL (tenant_id = '*')
  - 35 Shared OS logical scripts
  - 25 marketing laws
  - OECD governance standards
  - NIST cybersecurity frameworks
  → One copy. All tenants share. Read-only.

TIER 2: DEPARTMENT TEMPLATES (tenant_id = '_template_')
  - Brand Studio content pipeline
  - Governance 4-gate cycle
  - Engineering security charter
  → One copy per department. Tenant inherits structure.

TIER 3: TENANT CONFIG (tenant_id = 'novizio' / 'hourbour' / 'client-x')
  - Active departments and agents
  - Custom thresholds (spend gates, risk acceptance)
  - Brand guidelines per brand
  - Tenant-specific book-requirements.md
  → Each tenant has their own. SQL WHERE isolates.

TIER 4: TENANT MEMORY (tenant_id = 'novizio' / 'hourbour' / 'client-x')
  - Agent output logs
  - Decision history
  - Feedback quality scores per chunk
  → Each tenant has their own. Never shared.
```

## Knowledge Update Cascade

```
1. Operator updates Shared OS logical script
2. File watcher detects change → re-index Global tier only
3. All tenants get updated embeddings on next query
4. Quality scores preserved (linked by chunk_id, not content)
5. Tenant-specific quality scores unaffected
```

## Why This Architecture Scales

| Dimension | At 7 Depts (now) | At 13 Depts (future) | At 1,000 Tenants |
|-----------|-----------------|---------------------|-------------------|
| Docs indexed | 651 | ~1,200 | ~50,000 |
| Chunks | ~5,000 | ~10,000 | ~500,000 |
| Query latency | 80ms | 80ms | 85ms |
| Embedding cost | $0 (local) | $0 (local) | $0 (local) |
| Tenant isolation | N/A | N/A | SQL WHERE clause |
| Knowledge updates | Manual re-index | Auto-cascade | Auto-cascade, zero per-tenant cost |
| Feedback signal | Per chunk | Per chunk | Per chunk PER tenant |

## Build Status

| # | Element | File | Tests | Status |
|---|---------|------|-------|--------|
| 1 | Semantic Chunker | `chunkify.py` | 17/17 | ✅ Complete |
| 2 | Hybrid Embedder | `embed.py` | 12/12 | ✅ Complete |
| 3 | Dynamic Optimizer | `optimizer.py` | 24/24 | ✅ Complete |
| 4 | Full Retrieval Pipeline | `retriever.py` | 20/20 | ✅ Complete |
| 5 | Re-index Worker | `feedback.py` (--watch) | — | ✅ Complete |
| 6 | Feedback Loop | `feedback.py` | 13/13 | ✅ Complete |
| 7 | CAOS Graph Integration | Pending | — | ⏳ Design phase |
