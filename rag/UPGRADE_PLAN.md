#!/usr/bin/env python3
"""
YVON Harness Upgrade — Master Implementation Plan
===================================================
Build Order: Module → Test → Integrate → Validate → Proceed

Each module builds in /tmp/harness_sandbox/ first. Only after passing
all tests does it move to rag/. This prevents breaking the live pipeline.

Rollback: If any module fails integration, all changes from that phase
are reverted via git checkout. The user gets a report: what broke, why,
can we go back to the previous state.

Multi-LLM Architecture:
  hermes + claude → primary reasoning + code generation
  deepseek → adversarial verification (different perspective)
  chatgpt → content/creative quality assessment

Self-Improving Loop:
  Every Sunday 00:00 UTC: self_improver.py runs
  1. Analyzes field_monitor data from the past week
  2. Proposes parameter adjustments (budget multipliers, thresholds)
  3. Tests proposals in sandbox against 10 benchmark scenarios
  4. If all pass → auto-deploy to rag/
  5. If any fail → report to operator, do NOT deploy
  6. Logs all changes with rationale to improvement_log.jsonl

SANDBOX RULES (every test runs inside these):
  1. No write access to Teams/ source files
  2. Test data is synthetic — never reads real agent files
  3. SQLite test DB created in /tmp, destroyed after tests
  4. If any test touches a real file → HALT + report
  5. Import isolation: new modules import from /tmp copies, not live rag/
"""

import sys, os

# ═══════════════════════════════════════════════════════════════════
# PHASE 0: PRE-FLIGHT CHECKS
# ═══════════════════════════════════════════════════════════════════

PHASE_0_CHECKS = """
1. Verify all 111 existing tests pass before touching anything.
   → python3 rag/injector.py --test     (22 tests)
   → python3 rag/strategy.py --test      (23 tests)
   → python3 rag/destructor.py --test    (35 tests)
   → python3 rag/unified_pipeline.py --test (31 tests)

2. Create git checkpoint:
   → git add -A && git commit -m "pre-harness-upgrade checkpoint"

3. Create sandbox directory:
   → /tmp/harness_sandbox/

4. Copy dependencies into sandbox:
   → cp rag/injector.py rag/embed.py rag/feedback.py → /tmp/harness_sandbox/
"""

# ═══════════════════════════════════════════════════════════════════
# PHASE 1: CORE HARNESS (harness.py)
# ═══════════════════════════════════════════════════════════════════

PHASE_1 = {
    'build': 'rag/harness.py',
    'gates': ['source_authentication', 'reliability_scoring', 'conflict_detection',
              'priority_assembly', 'quarantine_recovery'],
    'tests': 35,
    'sandbox': True,
    'depends_on': ['injector.py', 'embed.py', 'feedback.py', 'staleness_economics.py'],
    'modifies': [],
    'risk': 'MEDIUM — modifies how chunks enter context. If broken, chunks still pass through but with no verification.',
    'rollback': 'git checkout rag/harness.py  (new file, safe to delete)',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 2: OPTIMIZER FIXES (optimizer.py modifications)
# ═══════════════════════════════════════════════════════════════════

PHASE_2 = {
    'build': 'modify rag/optimizer.py',
    'changes': [
        'compute_chunk_quality() → multiplicative formula',
        'source_authority mapping (book=1.0, standard=0.9, dept_doc=0.7, etc.)',
        'wire calibration_weight() into retrieval confidence',
    ],
    'tests': 'existing 25 + 5 new ≈ 30',
    'sandbox': True,
    'depends_on': ['Phase 1 (needs source_authority from harness)'],
    'modifies': ['rag/optimizer.py'],
    'risk': 'HIGH — optimizer is in the critical path. If multiplicative formula has a bug, all chunk scores change.',
    'rollback': 'git checkout rag/optimizer.py',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 3: RETRIEVER PLAN-LOCK (retriever.py modification)
# ═══════════════════════════════════════════════════════════════════

PHASE_3 = {
    'build': 'modify rag/retriever.py',
    'changes': [
        'Plan-lock: hash execution plan before retrieval starts',
        'Authorization check: agent_id → allowed departments',
        'Source scope: which knowledge sources are in plan',
        'Deviation detection: any retrieval outside plan → HALT',
    ],
    'tests': 'existing + 10 new ≈ 30',
    'sandbox': False,  # needs real Teams/ to verify auth
    'depends_on': ['Phase 1 (needs plan-lock log from harness)'],
    'modifies': ['rag/retriever.py'],
    'risk': 'LOW — plan-lock is additive, doesn\'t change retrieval behavior, just adds authorization gate.',
    'rollback': 'git checkout rag/retriever.py',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 4: PROGRESSIVE DISCLOSURE (new module)
# ═══════════════════════════════════════════════════════════════════

PHASE_4 = {
    'build': 'rag/progressive_disclosure.py',
    'features': [
        'Load skill descriptions only (not full SKILL.md)',
        'Match query against skill triggers',
        'On-demand full load for matched skills',
        'Inactive skills → one-line summaries (~8 tokens each)',
        'Skill registry parser (reads agent.md Skill Roster section)',
    ],
    'tests': 15,
    'sandbox': True,
    'depends_on': ['None (standalone module)'],
    'modifies': [],
    'risk': 'LOW — standalone module, no existing pipeline modified.',
    'rollback': 'git checkout rag/progressive_disclosure.py',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 5: POST-HOC VERIFIER (new module)
# ═══════════════════════════════════════════════════════════════════

PHASE_5 = {
    'build': 'rag/verifier.py',
    'features': [
        'Grounded citation check: every factual claim vs injected chunks',
        'Embedding similarity between claim and source chunks',
        'Self-consistency check: does response contradict itself?',
        'Constitution check: does response comply with context constitution?',
        'Agent delegation: high-stakes → quinn/precedent/sentinel',
    ],
    'tests': 20,
    'sandbox': True,
    'depends_on': ['embed.py', 'Phase 1 (needs reliability scores from harness)'],
    'modifies': [],
    'risk': 'LOW — standalone module, called after LLM responds, doesn\'t block the pipeline.',
    'rollback': 'git checkout rag/verifier.py',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 6: UNIFIED PIPELINE INTEGRATION (wire everything together)
# ═══════════════════════════════════════════════════════════════════

PHASE_6 = {
    'build': 'modify rag/unified_pipeline.py',
    'changes': [
        'Call harness gates before strategy routing',
        'Inject conflict flags into context text',
        'Add grounded citation markers on every chunk',
        'Wire progressive disclosure into skill context assembly',
        'Pass conflicts + reliability to injection formatter',
    ],
    'tests': 'existing 31 + 10 new ≈ 41',
    'sandbox': True,
    'depends_on': ['Phase 1, 2, 3, 4 — everything must be built before wiring'],
    'modifies': ['rag/unified_pipeline.py'],
    'risk': 'HIGH — unified_pipeline is the production entry point. Integration bugs break all queries.',
    'rollback': 'git checkout rag/unified_pipeline.py',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 7: FEEDBACK LOOP EXTENSIONS
# ═══════════════════════════════════════════════════════════════════

PHASE_7 = {
    'build': 'modify rag/feedback.py',
    'changes': [
        'Source-level feedback: if source X produces bad outcomes, down-weight globally',
        'Budget feedback: if task_type consistently needs more budget, adjust multiplier',
        'Harness trace in Lasswell log',
        'Verification results in feedback record',
    ],
    'tests': 'existing + 5 new ≈ 20',
    'sandbox': True,
    'depends_on': ['Phase 5 (needs verification results)'],
    'modifies': ['rag/feedback.py'],
    'risk': 'MEDIUM — feedback changes affect quality scores globally, slow-moving (5% weight).',
    'rollback': 'git checkout rag/feedback.py',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 8: FIELD MONITOR (new module)
# ═══════════════════════════════════════════════════════════════════

PHASE_8 = {
    'build': 'rag/field_monitor.py',
    'features': [
        'Attractor detection: which chunk combinations produce consistently good/bad outcomes',
        'Degradation alerts: quality score dropping over time (sliding window)',
        'Coverage gaps: queries that consistently get too few chunks',
        'Drift detection: agent behavior changing as source documents evolve',
        'Weekly report generation (markdown + JSON)',
    ],
    'tests': 15,
    'sandbox': True,
    'depends_on': ['Phase 7 (needs feedback data)'],
    'modifies': [],
    'risk': 'LOW — read-only analysis module, generates reports, doesn\'t modify pipeline.',
    'rollback': 'git checkout rag/field_monitor.py',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 9: SELF-IMPROVER (new module — runs weekly)
# ═══════════════════════════════════════════════════════════════════

PHASE_9 = {
    'build': 'rag/self_improver.py',
    'features': [
        'Weekly schedule: Sunday 00:00 UTC (configurable)',
        'Phase 1 — ANALYZE: read field_monitor data from past week',
        'Phase 2 — PROPOSE: generate parameter adjustments (budget multipliers, strip thresholds, recovery triggers)',
        'Phase 3 — SANDBOX TEST: run proposed changes against 10 benchmark scenarios',
        'Phase 4 — DECIDE: if all pass → auto-deploy; if any fail → report + hold',
        'Phase 5 — DEPLOY: atomically swap parameters (file-based, not in-memory)',
        'Phase 6 — LOG: append to improvement_log.jsonl with rationale and test results',
        'Multi-LLM: hermes analyzes patterns, deepseek stress-tests proposals, chatgpt evaluates creative quality impact',
    ],
    'tests': 15,
    'sandbox': True,
    'depends_on': ['Phase 8 (needs field_monitor data)'],
    'modifies': [],
    'risk': 'HIGH — auto-deploying changes without human review. Mitigation: strict sandbox testing, no deploy on any failure, full rollback capability.',
    'rollback': 'git checkout rag/self_improver.py; revert parameter file to previous version',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 10: BRIDGE + CIE INTEGRATION
# ═══════════════════════════════════════════════════════════════════

PHASE_10 = {
    'build': 'modify rag/bridge.py + src/cie/classifier.ts',
    'changes': [
        'bridge.py: add harness trace to JSON response',
        'bridge.py: add verification results to JSON response',
        'classifier.ts: domain keyword priority fix (GDPR → legal_review)',
    ],
    'tests': 'integration tests across bridge protocol',
    'sandbox': True,
    'depends_on': ['Phase 6, 7 (harness + feedback must be wired before bridge exposes them)'],
    'modifies': ['rag/bridge.py', 'src/cie/classifier.ts'],
    'risk': 'LOW — additive changes to JSON response, classifier fix is one function.',
    'rollback': 'git checkout rag/bridge.py src/cie/classifier.ts',
}

# ═══════════════════════════════════════════════════════════════════
# PHASE 11: END-TO-END VALIDATION
# ═══════════════════════════════════════════════════════════════════

PHASE_11 = {
    'build': 'rag/e2e_validation.py',
    'scenarios': [
        '1. Creative review — spark reviews headline with Ogilvy rules + exception recovery',
        '2. Financial analysis — NPV/WACC computation with Brealey citations verified',
        '3. Legal compliance — GDPR Article 5 retention with Article 89 exception recovered',
        '4. Governance decision — board fiduciary review with NIST risk scoring + ISO contradiction detected',
        '5. Engineering debug — CI/CD pipeline fix with deployment protocol verification',
        '6. Factual lookup — simple Ogilvy rule retrieval, fast path, minimum budget',
        '7. Multi-department — deployment pipeline + GDPR compliance (legal_review routing)',
        '8. Contradiction scenario — NIST vs ISO risk scoring contradiction detected and flagged',
        '9. Stale/forged chunk — corrupted source file → authentication gate blocks chunk',
        '10. Progressive disclosure — marcus with 5 skills, only 3 match query, 2 summaries',
        '11. Stale quality score — chunk quality dropped over 30 days → field monitor alert',
        '12. Self-improver simulation — parameter adjustment proposed, sandbox tested, auto-deployed',
    ],
    'tests': 12 end-to-end scenarios,
    'sandbox': True,
    'depends_on': ['Everything — Phase 1-10 must be complete and tested'],
    'modifies': [],
    'risk': 'N/A — validation only, doesn\'t modify anything.',
    'rollback': 'N/A',
}

# ═══════════════════════════════════════════════════════════════════
# TOTAL ESTIMATE
# ═══════════════════════════════════════════════════════════════════

TOTAL = """
New modules:     6 (harness, verifier, progressive_disclosure, field_monitor,
                    self_improver, e2e_validation)
Modified modules: 7 (optimizer, retriever, unified_pipeline, feedback,
                     bridge, classifier.ts, harness-engineering.md)
New tests:        ~140
Existing tests:   111 (must all still pass)
Total tests:      ~251 after upgrade

Estimated build time: 18-22 days at focused pace
Risk profile: 2 HIGH-risk changes (optimizer multiplicative formula,
              unified_pipeline integration) — both have immediate rollback.
"""

if __name__ == '__main__':
    print("YVON Harness Upgrade Plan")
    print("=" * 60)
    print(f"Phases: 11 (0-10)")
    print(f"New modules: 6")
    print(f"Modified modules: 7")
    print(f"New tests: ~140")
    print(f"Existing tests to preserve: 111")
    print(f"\nSandbox: /tmp/harness_sandbox/")
    print(f"Rollback: git checkout <file> for any failed phase")
    print(f"\nStart with: Phase 0 — pre-flight checks")
    print(f"Then build: Phases 1-10 in order")
    print(f"Validate: Phase 11 — 12 E2E scenarios")
