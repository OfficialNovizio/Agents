# relay — Logical Layer (placeholder)

No logical skills yet — relay's decisions (scope ladders, retry budgets, thresholds) are **reasoning-based, not formula-verified** (rule 0.6 flag active).

**What's needed:** a rigorous source on distributed-systems reliability (e.g. a release-engineering/SRE text with actual queueing/retry math) and/or an access-control formalism reference (least-privilege as lattice/RBAC math).

**What it would ground:** retry/backoff parameters derived from load math instead of convention; circuit-breaker thresholds from failure-rate statistics; grant-audit sampling from risk scoring.

**How to fill:** operator supplies the book → extract formulas with citations (playbook §8.2) → un-flag affected defaults in relay-config.
