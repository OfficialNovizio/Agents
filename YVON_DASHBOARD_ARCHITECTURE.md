# YVON Dashboard Architecture — Two-Tier Design

**Status:** Design Phase
**Date:** 2026-07-16
**Purpose:** YVON Master Dashboard (operators) + Per-Brand Dashboard (business owners)

---

## DESIGN PRINCIPLES

1. **Business owners never see agent internals.** No graphs. No RAG pipelines. No harness gates. They see: is my content being posted? How's engagement? What's coming up?
2. **Operators see everything.** Master dashboard gets the full picture: all brands, all agents, all metrics, all alerts.
3. **"Add new brand" is one button.** Triggers full tenant provisioning under AgentX SaaS.

---

## TIER 1: YVON MASTER DASHBOARD (for operators/you)

```
┌──────────────────────────────────────────────────────────────────────┐
│  YVON MASTER — v1.3.0                           [⚙️]  [🔔 3 alerts] │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┬──────────────┬──────────────┬──────────────────┐  │
│  │ FLEET HEALTH │  RAG HEALTH  │  GRAPH VITALS│  SELF-IMPROVER   │  │
│  │  46/46 ✅    │  285 tests   │  1,482 nodes │  Last run: 12h   │  │
│  │  0 down      │  0 failures  │  3,840 edges │  0 deployed      │  │
│  │  7 depts ok  │  89% savings │  12 comms    │  4 held          │  │
│  └──────────────┴──────────────┴──────────────┴──────────────────┘  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  CONNECTED BRANDS                                    [+ ADD] │    │
│  │                                                              │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐│    │
│  │  │ NOVIZIO      │  │ HOURBOUR     │  │ BOUTIQUE A           ││    │
│  │  │ Owned Brand   │  │ Owned Brand  │  │ AgentX Tenant        ││    │
│  │  │ ◆◆◆◆◇ 82%    │  │ ◆◆◆◆◆ 93%   │  │ ◆◆◆◇◇ 61%          ││    │
│  │  │              │  │              │  │                      ││    │
│  │  │ 3 depts act. │  │ 2 depts act. │  │ 2 depts: Brand+Prod ││    │
│  │  │ 11 agents    │  │ 8 agents     │  │ Growth tier ($149/m) ││    │
│  │  │ IG ✅ Shopify✅│  │ IG ✅        │  │ IG ✅ Shopify ✅     ││    │
│  │  │ [Open →]     │  │ [Open →]     │  │ [Open →]            ││    │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘│    │
│  │                                                              │    │
│  │  ┌──────────────────────┐  ┌──────────────────────┐          │    │
│  │  │ CAFE B               │  │ SAAS CO C            │          │    │
│  │  │ AgentX Tenant        │  │ AgentX Tenant        │          │    │
│  │  │ ◆◆◆◆◇ 78%           │  │ ◆◆◆◆◆ 95%           │          │    │
│  │  │ 1 dept: Social       │  │ 4 depts: Full Stack  │          │    │
│  │  │ Starter tier ($49/m) │  │ Scale tier ($399/m)  │          │    │
│  │  │ IG ✅                │  │ All connectors ✅    │          │    │
│  │  │ [Open →]             │  │ [Open →]             │          │    │
│  │  └──────────────────────┘  └──────────────────────┘          │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  RECENT ALERTS                                               │    │
│  │  ⚠️ Boutique A — Instagram connector degraded (12h ago)      │    │
│  │  ⚠️ Cafe B — Coverage gap: only 1.3 chunks per query         │    │
│  │  ⚠️ Novizio — Spark quality drifting (-0.12 over 4 weeks)    │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────┐  ┌─────────────────────────────┐   │
│  │ OBSIDIAN GRAPH PREVIEW      │  │ CROSS-TENANT LEARNING        │   │
│  │ [graph visualization]       │  │ Boutiques: visual > text     │   │
│  │ 12 communities              │  │ 9am posts: 2.3x engagement   │   │
│  │ Fleet cohesion: 0.87        │  │ Cafes: event posts peak Fri  │   │
│  │ [Open Obsidian Vault →]     │  │ [View Patterns →]            │   │
│  └─────────────────────────────┘  └─────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
```

---

## TIER 2: PER-BRAND DASHBOARD (for business owners)

### What the business owner sees:

```
┌──────────────────────────────────────────────────────┐
│  BOUTIQUE A — Your Dashboard          [⚙️ Settings]  │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────────────┬─────────────────┬────────────┐  │
│  │ THIS WEEK       │ YOUR CONTENT    │ ENGAGEMENT │  │
│  │ 5 posts ready   │ 3 Instagram     │ 1.2k views │  │
│  │ 2 reviews done  │ 2 Stories       │ 84 clicks  │  │
│  │ Next: Thursday  │ Drafts: 4       │ 3.2% CTR   │  │
│  └─────────────────┴─────────────────┴────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │  UPCOMING CONTENT — THIS WEEK                │    │
│  │                                              │    │
│  │  MON  │ New arrivals Instagram post    ✅    │    │
│  │  TUE  │ Behind-the-scenes Story        ✅    │    │
│  │  WED  │ Customer spotlight carousel    ⏳    │    │
│  │  THU  │ Sale announcement + link       📝    │    │
│  │  FRI  │ Weekend style inspiration      📝    │    │
│  │                                              │    │
│  │  [Approve] [Request Changes] [Skip Week]     │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │  RECENT ACTIVITY                             │    │
│  │  📸 Posted: "Summer Collection Drop" — 340♥  │    │
│  │  ✏️ Spark reviewed 3 drafts                   │    │
│  │  📊 Weekly report ready — 12% engagement ⬆   │    │
│  │  🔗 Shopify connected — 14 new products       │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │  CONNECTIONS            [Manage →]           │    │
│  │  ✅ Instagram   ✅ Shopify   ✅ Mailchimp     │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  Plan: Growth ($149/mo) · Next bill: Aug 1           │
└──────────────────────────────────────────────────────┘
```

### What the business owner NEVER sees:
- Agent names (spark, lena, pixel) — they see "Your Creative Team"
- RAG pipeline metrics — they see "Content Quality: Good ✅"
- Harness gates — they see "All systems working"
- Graph databases — they see their content feed
- Token savings — they see "Optimized for speed"

---

## "ADD NEW BRAND" FLOW

```
YVON MASTER DASHBOARD
    │
    │  [ + ADD BRAND ] clicked
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  ADD NEW BRAND                                       │
│                                                      │
│  Brand Name: [________________]                      │
│  Industry:   [Fashion Retail ▾]                      │
│  Website:    [________________]                      │
│                                                      │
│  What do you need help with?                         │
│  ☑ Social Media Management                          │
│  ☐ Brand Design & Identity                           │
│  ☐ Customer Support                                  │
│  ☐ E-Commerce Operations                             │
│                                                      │
│  Plan: [Growth ($149/mo) ▾]                          │
│        2 departments · 8 agents                      │
│                                                      │
│  [Create & Provision]                                │
└──────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  PROVISIONING...                                     │
│                                                      │
│  ✅ Created tenant graph vault                        │
│  ✅ Deployed Brand Studio (spark, lena, pixel, pulse) │
│  ✅ Applied industry overrides (fashion retail)       │
│  ✅ Created brand voice profile                       │
│  ⏳ Connecting Instagram...                          │
│  ⏳ Connecting Shopify...                            │
│  ⬜ Running smoke test...                            │
│  ⬜ Generating welcome content...                     │
└──────────────────────────────────────────────────────┘
```

---

## TECHNICAL ARCHITECTURE

```
┌───────────────────────────────────────────────────────────┐
│  FRONTEND: HTML Artifacts (Cowork sidebar)                │
│  • Master Dashboard: One artifact, updates real-time      │
│  • Per-Brand Dashboard: One artifact per brand            │
│  • Add Brand: Modal overlay, calls provisioner            │
└───────────────────────┬───────────────────────────────────┘
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
┌──────────────┐ ┌────────────┐ ┌──────────────┐
│ DATA SOURCES │ │ MONITORING │ │ PROVISIONING │
│              │ │            │ │              │
│ field_       │ │ harness.py │ │ platform/    │
│ monitor.py   │ │ trace API  │ │ scaffold.py  │
│ • attractors │ │ • auth     │ │ (planned)    │
│ • degradation│ │ • reliab.  │ │              │
│ • coverage   │ │ • conflict │ │ tenant_      │
│ • drift      │ │ • quaran.  │ │ provisioner  │
│              │ │            │ │ .py          │
│ feedback.py  │ │            │ │ (planned)    │
│ • traces     │ │            │ │              │
│ • outcomes   │ │            │ │              │
│              │ │            │ │              │
│ graph        │ │            │ │              │
│ builder.ts   │ │            │ │              │
│ • nodes      │ │            │ │              │
│ • edges      │ │            │ │              │
│ • comms      │ │            │ │              │
└──────┬───────┘ └─────┬──────┘ └──────┬───────┘
       │               │               │
       └───────────────┼───────────────┘
                       │
                       ▼
┌───────────────────────────────────────────────────────────┐
│  API LAYER: bridge.py extended with --mode dashboard       │
│  • GET  /dashboard/master  → fleet health, brand list     │
│  • GET  /dashboard/:brand  → per-brand metrics            │
│  • POST /dashboard/brands  → create & provision new brand │
└───────────────────────────────────────────────────────────┘
```

---

## DASHBOARD METRICS — What Feeds What

### Master Dashboard

| Card | Data Source | Update Frequency |
|------|------------|-----------------|
| Fleet Health (46/46 agents) | `field_monitor.py` drift signals | Hourly |
| RAG Health (tests, savings) | `field_monitor.py` + test runner | Daily |
| Graph Vitals (nodes/edges) | `src/graphs/builder.ts` `getGraphStats()` | Daily |
| Self-Improver Status | `self_improver.py` `improvement_log.jsonl` | Weekly |
| Brand Cards (health score) | Per-brand `field_monitor` aggregation | Hourly |
| Recent Alerts | `field_monitor.py` degradation + drift + harness quarantine log | Real-time |
| Obsidian Graph Preview | Obsidian vault API / graph stats | On load |
| Cross-Tenant Learning | `cross_tenant_learner.py` patterns (planned) | Weekly |

### Per-Brand Dashboard

| Card | Data Source | Update Frequency |
|------|------------|-----------------|
| This Week (posts, reviews) | Per-agent session logs | Hourly |
| Your Content (output feed) | Content pipeline output | Real-time |
| Engagement (views, clicks) | Connector API (Instagram, Shopify) | Daily |
| Upcoming Content (calendar) | Content pipeline scheduling | Real-time |
| Recent Activity (feed) | Aggregated agent session logs | Hourly |
| Connections (integrations) | relay MCP registry (per-tenant) | On load |
| Plan (tier, billing) | `platform/billing_tiers.py` (planned) | Monthly |

---

## WHAT GETS BUILT FOR THE DASHBOARD

### New Files

| File | Purpose |
|------|---------|
| `dashboard/master_dashboard.html` | YVON Master Dashboard — self-contained artifact |
| `dashboard/brand_dashboard.html` | Per-Brand Dashboard — self-contained artifact |
| `dashboard/dashboard_api.py` | API endpoints for dashboard data (calls existing modules) |
| `dashboard/add_brand.py` | Add Brand wizard — collects info, triggers provisioning |

### What Already Exists (No Changes Needed)

| Data | Module | Status |
|------|--------|--------|
| Fleet health | `field_monitor.py` | ✅ Ready — drift signals per agent |
| RAG health | Test runner + `field_monitor.py` | ✅ Ready — savings %, quality scores |
| Graph stats | `src/graphs/builder.ts` | ✅ Ready — `getGraphStats()` |
| Self-improver log | `self_improver.py` | ✅ Ready — `improvement_log.jsonl` |
| Alerts | `field_monitor.py` + `harness.py` | ✅ Ready — degradation, quarantine |
| Content feed | Content pipeline output | ❌ Needs per-tenant log (Phase 2) |
| Engagement | Connector SDK (planned Phase 4) | ❌ Phase 4 |
