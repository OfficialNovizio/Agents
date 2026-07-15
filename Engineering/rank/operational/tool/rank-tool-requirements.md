---
name: rank-tool-requirements
type: operational/tool
status: specifies needs, does not grant them — grants happen at deployment via config/connectors
assigned_agent: rank (Engineering / Technical SEO)
date_added: 2026-07-09
---

## Purpose

What rank needs, and what happens without each. Every external tool call is plan-locked (Rail 1) and sandboxed (Rail 2); rank runs no data changes (Rail 3) and edits no production directly.

## Requirements

| Need | Tool / access | Used by | Without it |
|---|---|---|---|
| claude-seo plugin | AgriciDaniel/claude-seo (MIT) — runtime install, proposed connector §5 | claude-seo-integration | Method-only SEO (siblings), labeled reduced-automation |
| Site read access (crawl the site, read robots/sitemap) | HTTP read of the site | technical-seo-execution, structured-data-geo | Can't audit; escalate |
| Google SEO APIs (optional) | GSC / PageSpeed / CrUX / Indexing / GA4 creds | claude-seo-integration (seo-google) | Free/estimated data only, labeled |
| Plugin extensions (optional) | Firecrawl, DataForSEO, image-gen (separate installers) | claude-seo-integration | Core plugin only |
| Schema validation | Schema.org / Google structured-data tooling | structured-data-geo | Manual validation, labeled |

## Explicit non-needs / prohibitions (by design)

- **No direct production edits** — rank diagnoses and specs; mia/raj implement through dev review + quinn's gate. The plugin analyzes/generates; it never auto-applies to the live site.
- **No SEO strategy or business-facing measurement ownership** — that's kai (boundary skill).
- **No data change execution** (Rail 3).
- **No shipping the plugin's community-promo footer** to operator output.

## Notes

- The claude-seo plugin is treated as a dated tool (its SEO facts — INP, schema deprecations — have dates); verify high-stakes facts against current Google guidance (ops's volatility-split discipline).
- Plugin + Google API creds are proposed connectors surfaced to the operator at deployment (plan §5).
