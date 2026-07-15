---
name: mia-tool-requirements
type: operational/tool
status: specifies needs, does not grant them — grants happen at deployment via config/connectors
assigned_agent: mia (Engineering / Frontend Web)
date_added: 2026-07-09
---

## Purpose

What mia needs, and what happens without each. Every external tool call is plan-locked (Rail 1) and sandboxed (Rail 2); mia runs no data changes (Rail 3).

## Requirements

| Need | Tool / access | Used by | Without it |
|---|---|---|---|
| Repo read/write for frontend code | repo scope (per stack-profile) | all skills | Core; without write, design-only |
| Frontend framework + build | per stack-profile | all skills | Method-only guidance |
| Brand kit access | atlas's kit (Brand Studio) | design-tokens | Provisional tokens, labeled; atlas is pending source of truth |
| Token tooling | Style Dictionary / CSS vars (per stack-profile) | design-tokens | Manual token management, labeled |
| Agentation MCP | agentation.com (proposed connector, §5) | frontend-verification | Feedback from description, labeled less precise |
| Browser verification | quinn's Reticle + Playwright | frontend-verification | Manual checklist; E-tier UNMET |
| Perf tooling | Lighthouse-class (per stack-profile) | frontend-performance | Emulated throttling, labeled fidelity gap |

## Explicit non-needs (by design)

- **No data change execution** — mia builds UI; data changes are dana + operator (Rail 3).
- **No production deploy control** — ops ships; mia's code passes quinn's gate first.
- **No secrets in client code** — anything in the frontend bundle is public (aegis's concern).

## Notes

- mia is a primary builder with genuine code write access — every change passes dev's review (incl. integrity checks: mock data, hardcoded brand values), quinn's gate, and aegis for client-security surfaces.
- Agentation + Reticle/Playwright are proposed connectors (plan §5), surfaced to the operator at deployment; skills degrade to method/manual without them.
