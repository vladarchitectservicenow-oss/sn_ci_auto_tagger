# sn_ci_auto_tagger Risk Report

**Product:** ServiceNow CI Auto Tagger (sn_ci_auto_tagger)
**Author:** Vladimir Kapustin
**Date:** 2026-05-25

## Risk Matrix

| Risk ID | Description | Category | Likelihood | Impact | Severity | Mitigation |
|---------|-------------|----------|------------|--------|----------|------------|
| R01 | CMDB plugin not activated on target instance | Platform | Low | Critical | **P0** | Pre-flight check on scan start; return clear error: "CMDB plugin required. Activate com.snc.cmdb." |
| R02 | False-positive classification on edge-case CI names | Data | Medium | High | **P1** | Confidence gating: classifications <0.85 flagged for review. Manual override table. |
| R03 | Race condition on concurrent scans writing to same CI | Concurrency | Low | High | **P1** | Scan lock via `x_sn_ci_auto_tagger_config.scan_lock` flag; one scan at a time. |
| R04 | Performance degradation on CMDBs > 500K CIs | Performance | Medium | Medium | **P2** | Configurable batch size (default 100); incremental scans reduce load; off-hours scheduling. |
| R05 | Rate limiting by ServiceNow REST API (CLI tool) | External API | Medium | Medium | **P2** | Exponential backoff in fetch loop; `--limit` flag to reduce batch sizes. |
| R06 | Missing `sys_class_name` field on non-standard CI tables | Schema | Medium | Medium | **P2** | Table eligibility check before scan; skip non-CI tables silently; log skipped count. |
| R07 | Stale classification rules after platform upgrade | Rules | Medium | High | **P1** | Rule versioning with release tag; rule validation script against current instance schema. |
| R08 | Audit table growth exceeding storage limits | Storage | Medium | Medium | **P2** | Configurable retention policy; auto-purge audits older than N days; archive to external store. |
| R09 | Rollback failure due to external modification of CI | Rollback | Low | High | **P1** | Rollback validates current CI state matches audit snapshot; warns if external changes detected. |
| R10 | Hardcoded credentials in source code | Security | Low | Critical | **P0** | Pre-commit grep for credential patterns; CI lint gate; credential scan in pipeline. |
| R11 | Missing `.gitignore` leading to `__pycache__/` in commits | Quality | High | Low | **P3** | Mandatory `.gitignore` with `__pycache__/`, `*.pyc`, `reports/`; pre-commit check. |
| R12 | README license header contradicts LICENSE file | Compliance | Medium | Medium | **P2** | Gate G7: verify `grep -i 'license:' README.md` matches `head -1 LICENSE` before push. |
| R13 | Duplicate README sections from mass template expansion | Quality | High | Low | **P3** | Gate G8: verify single `## Overview`, `## License` headings before commit. |
| R14 | Script Include scope access missing for cross-scope reads | Platform | Medium | High | **P1** | Cross-scope access grants for CMDB tables; test on PDI with restricted user. |
| R15 | PDI hibernation blocks smoke testing | Testing | High | Medium | **P2** | PDI status recorded in `tests/PDI_STATUS.md`; fallback to mock CI for test validation. |

## Severity Definitions

| Severity | Definition | Action Required |
|----------|-----------|-----------------|
| **P0** | Application non-functional or security-critical | Block deployment; fix before release |
| **P1** | Core feature degraded or high-risk scenario | Fix in current sprint; workaround documented |
| **P2** | Non-critical feature affected or medium risk | Document workaround; fix in next sprint |
| **P3** | Cosmetic or low-impact quality issue | Fix when convenient; no user impact |

## Risk Trend Analysis

| Metric | Previous Assessment | Current Assessment | Trend |
|--------|--------------------|--------------------|-------|
| Total risks identified | 8 | 15 | ↑ (improved detection) |
| P0 risks | 2 | 2 | → (stable) |
| P1 risks | 3 | 5 | ↑ (new classification + concurrency risks) |
| P2 risks | 2 | 5 | ↑ (expanded coverage) |
| P3 risks | 1 | 3 | ↑ (quality gate additions) |
| Mitigated risks | 3 | 0 | ↓ (fresh assessment — awaiting fix verification) |

## Top Priority Mitigations (This Cycle)

1. **R01 (P0):** Implement pre-flight CMDB plugin check in `CIClassifierEngine.scan()` — block scan if `GlidePluginManager.isActive('com.snc.cmdb')` is false.
2. **R10 (P0):** Run credential scan across all source files — verify no hardcoded passwords, tokens, or instance URLs.
3. **R02 (P1):** Implement confidence threshold UI in application settings — expose `default_confidence_threshold` as configurable.
4. **R03 (P1):** Implement scan lock via atomic GlideRecord update on `x_sn_ci_auto_tagger_config`.
5. **R14 (P1):** Create cross-scope access record for CMDB tables; document in deployment guide.
