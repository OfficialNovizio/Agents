# cortex — Logical Layer (placeholder, rule 0.6)

Judgments flagged **reasoning-based, not formula-verified** until sources land.

## Wants
1. **An incident-response / DFIR text** (NIST SP 800-61r2, or equivalent) — grounds the IR lifecycle, severity classification, evidence handling, and notification timing. cortex's primary want.
2. **A detection-engineering methodology text** (detection engineering maturity model, signal-to-noise ratio frameworks) — grounds false-positive classification and rule tuning.
3. **A threat-hunting methodology reference** (SANS hunting maturity model, hypothesis generation frameworks).

## Currently flagged reasoning-based (0.6)
- Severity classification matrix (SEV1-SEV4 definitions are rubric-based, not statistically grounded).
- Patch/response SLAs (time-to-triage, time-to-contain targets are convention).
- Hunting cadence and hypothesis prioritization.

## Extraction protocol
When the IR/DFIR source lands: severity definitions extracted with page cites; notification timing grounded; IR workflow updated to cite the source. Detection-engineering and threat-hunting references handled similarly when they arrive.
