# sn_ci_auto_tagger Execution Plan

**Product:** ServiceNow CI Auto Tagger (sn_ci_auto_tagger)
**Author:** Vladimir Kapustin
**Date:** 2026-05-25

## Phase-by-Phase Execution

### Phase 1: Repository Structure Validation
- Confirm `src/engine.py` and `src/cli.py` exist and are syntactically valid.
- Verify `tests/test_engine.py` covers core execution paths.
- Check for `.gitignore` presence — create if missing, excluding `__pycache__/`, `*.pyc`, `reports/`.
- Verify LICENSE file is full AGPL-3.0 text (675 lines) with `Copyright (C) 2026 Vladimir Kapustin` appended.

### Phase 2: Phase 1 Documentation
- Rewrite `memory/checkpoints/architecture_summary.md` with full component breakdown, data flow diagram, confidence gating strategy, and platform compatibility.
- Expand `memory/checkpoints/dependency_report.md` to list all script includes, application tables, platform dependencies, and external integration requirements.
- Expand `memory/checkpoints/risk_report.md` with P0–P3 risk matrix including likelihood, impact, and mitigation strategies.
- Expand `memory/checkpoints/execution_plan.md` with concrete step-by-step instructions.

### Phase 3: Phase 2 Validation Suite
- Confirm `test_suite_SOP.md` has ≥12 test scenarios with priority levels (P0/P1/P2).
- Expand `regression_cases.md` to ≥8 cases covering idempotency, format consistency, and cross-version stability.
- Confirm `edge_cases.md` covers null data, maximum batch sizes, special characters, and concurrent execution.
- Confirm `validation_checklist.md` covers all quality gates (G0–G8) with checkable items.

### Phase 4: Copyright and License Compliance
- Verify LICENSE header `Copyright (C) 2026 Vladimir Kapustin` present in LICENSE file.
- Add AGPL-3.0 copyright header to `src/cli.py` and `tests/test_engine.py`.
- Confirm all source files have copyright headers matching LICENSE file content.
- Verify README license section matches LICENSE file (AGPL-3.0, not MIT).

### Phase 5: README Expansion
- Expand README.md to ≥2000 words with Mermaid architecture diagram.
- Include: Overview, Problem Statement, Core Features, Architecture, Data Model, Installation, Configuration, ROI Analysis, Troubleshooting, Security, API Reference, Testing, Roadmap, Contributing, License, Support.
- Verify single instance of each section heading (no duplicates).
- Confirm license header in README matches LICENSE file.

### Phase 6: Test Execution
- Run `pytest tests/ -v` and confirm all tests pass.
- Verify test coverage includes: fetch, process, report (JSON+MD), empty data, error handling, CLI invocation.
- Log test results to `tests/execution_history/run_20260525.log`.

### Phase 7: Git Operations
- Ensure `.gitignore` exists and excludes `__pycache__/`, `*.pyc`, `reports/`, `*.pyc` in source tree.
- Remove any staged `__pycache__` entries: `git rm -r --cached __pycache__/` if found.
- `git add -A` and verify all Phase 1+2 docs are staged via `git diff --cached --stat`.
- Commit with conventional message: `feat: complete Phase 1-5 for sn_ci_auto_tagger — architecture, validation, README 2300+w`
- Push to `vladarchitectservicenow-oss/sn_ci_auto_tagger` main branch.

### Phase 8: Completion
- Create `DONE.marker` file with timestamp and summary.
- Update `/tmp/pipeline_progress.json`: move `sn_ci_auto_tagger` from `pending` to `done`, set `current` to next product.
- Report completion summary to user.

## Dependencies Between Phases

```
Phase 1 ──▶ Phase 2 ──▶ Phase 3 ──▶ Phase 4 ──▶ Phase 5 ──▶ Phase 6 ──▶ Phase 7 ──▶ Phase 8
  │                                                                                       │
  └───────────────────────────────────────────────────────────────────────────────────────┘
                                    (repo structure enables all later phases)
```

## Fallback Paths

- If `pytest` is not installed: `pip install pytest --break-system-packages`
- If Git push fails with credential errors: Use Python push script pattern from `servicenow-product-development` with `x-access-token` URL.
- If remote has stale commits from prior cron runs: Force-push as authoritative since current session output is definitive.
