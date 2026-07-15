# keyring — Logical Layer (placeholder, rule 0.6)

Judgments flagged **reasoning-based, not formula-verified** until sources land. access_review.py's set arithmetic is deterministic (not flagged); the cadences/heuristics below are.

## Wants
1. **An IAM / zero-trust reference** (NIST 800-63 Digital Identity; zero-trust architecture NIST 800-207) — grounds identity-proofing levels, review cadences, and just-in-time/zero-standing-privilege reasoning.
2. **A secrets-management / cryptographic key-management reference** (NIST 800-57-aligned) — grounds rotation cadences and key scoping.

## Currently flagged reasoning-based (0.6)
- access_review_cadence / privileged_review_cadence (quarterly is convention).
- secret_rotation_cadence and deprovision_sla targets.
- "just-in-time vs standing" thresholds for when standing privilege is justified.

## Extraction protocol
When the IAM/zero-trust + key-management sources land: cadence/proofing/rotation reasoning extracted with page cites; the skills updated to cite them; flags removed where a citation replaces them. The set-diff math in access_review.py is already exact.
