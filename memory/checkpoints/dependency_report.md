# sn_ci_auto_tagger Dependency Report

**Product:** ServiceNow CI Auto Tagger (sn_ci_auto_tagger)
**Scope:** `x_sn_ci_auto_tagger`
**Author:** Vladimir Kapustin
**Date:** 2026-05-25

## Internal Dependencies (ServiceNow Platform)

### Required Plugins
| Plugin | ID | Status | Notes |
|--------|-----|--------|-------|
| Configuration Management (CMDB) | `com.snc.cmdb` | Required | Core CMDB tables (`cmdb_ci`, `cmdb_rel_ci`) — must be active |
| System Scheduler | `com.glide.schedule` | Required | Scheduled jobs for automated scans |
| REST API Processor | `com.glide.rest.api` | Required | Outbound REST Message support for CI/CD integration |

### Application Tables
| Table | Dependency Type | Notes |
|-------|----------------|-------|
| `x_sn_ci_auto_tagger_rule` | Self-contained | Classification rule definitions — no external dependencies |
| `x_sn_ci_auto_tagger_scan_run` | Self-contained | Scan execution tracking |
| `x_sn_ci_auto_tagger_finding` | Self-contained | Per-CI classification results |
| `x_sn_ci_auto_tagger_audit` | Self-contained | Change audit trail |
| `x_sn_ci_auto_tagger_config` | Self-contained | Application settings |

### Platform Tables Read
| Table | Access Pattern | Field Usage |
|-------|---------------|-------------|
| `cmdb_ci` | GlideRecord read | `sys_id`, `sys_class_name`, `name`, `manufacturer`, `model_id`, `os`, `sys_updated_on` |
| `cmdb_ci_server` | GlideRecord read | Extension of cmdb_ci — server-specific fields |
| `cmdb_ci_appl` | GlideRecord read | Extension of cmdb_ci — application CI fields |
| `cmdb_rel_ci` | GlideRecord read | `parent`, `child`, `type` for relationship inference |
| `sys_trigger` | GlideRecord read (admin only) | Verify scheduled job status |

### Platform Tables Written
| Table | Access Pattern | Notes |
|-------|---------------|-------|
| `cmdb_ci` (and extensions) | GlideRecord write | Write `sys_class_name`, tag-related fields |
| All `x_sn_ci_auto_tagger_*` tables | GlideRecord write | Application-owned — full CRUD |

### Required Roles
| Role | Purpose |
|------|---------|
| `x_sn_ci_auto_tagger.admin` | Application administrator — manage rules, config, scan execution |
| `x_sn_ci_auto_tagger.user` | Read-only access to dashboards and reports |
| `cmdb_admin` (platform) | Write access to CMDB tables for classification writes |

## External Dependencies

### Python Runtime (CLI Tool)
| Dependency | Version | Purpose |
|-----------|---------|---------|
| Python | 3.10+ | CLI tool runtime |
| `requests` | 2.28+ | HTTP client for REST API calls to ServiceNow instance |
| `pytest` | 7.0+ | Test framework for unit and integration tests |

### Optional Integrations
| Integration | Protocol | Dependency |
|------------|----------|------------|
| Power BI / Tableau | JSON import | `CIReportGenerator` JSON export consumed by BI tools |
| External CI/CD | REST outbound | `sn_ws.RESTMessageV2` for finding push |
| SIEM / Monitoring | REST outbound | JSON finding export to external monitoring |
| Now Assist / AI Agent Studio (v2.0 planned) | REST API | AI-assisted classification suggestions |

### Test Dependencies
| Dependency | Version | Purpose |
|-----------|---------|---------|
| `pytest` | 7.0+ | Test runner |
| `unittest.mock` | stdlib | Mocking HTTP calls via `patch("src.engine.requests.get")` |
| `tempfile` | stdlib | Temporary directories for report file tests |

## Dependency Risk Assessment

| Dependency | Risk Level | Mitigation |
|-----------|-----------|------------|
| CMDB Plugin (`com.snc.cmdb`) | **P0** — App non-functional without it | Pre-flight check on scan start; fail with clear message if missing |
| `requests` Python library | **P2** — Only affects CLI tool, not in-platform app | Install via `pip install requests`; CLI shows clear error message |
| REST API Processor | **P2** — Only needed for external CI/CD push | Feature gated behind config flag; graceful degradation if disabled |
| Scheduled Jobs | **P1** — Manual scan still works without it | Scheduled scan is optional; manual scan always available |
| System Scheduler plugin | **P1** — Same as Scheduled Jobs | Manual trigger via Scan Console bypasses scheduler dependency |

## Upgrade Path Notes

- **Utah → Zurich:** No breaking API changes for CMDB tables used.
- **Zurich → Australia:** `cmdb_ci` schema is stable; no field deprecations expected.
- Custom rules stored in `x_sn_ci_auto_tagger_rule` survive upgrades since table is application-owned.
