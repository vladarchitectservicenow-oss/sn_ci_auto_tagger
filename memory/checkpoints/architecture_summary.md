# sn_ci_auto_tagger Architecture Summary

**Product:** ServiceNow CI Auto Tagger (sn_ci_auto_tagger)
**Scope Prefix:** `x_sn_ci_auto_tagger`
**Author:** Vladimir Kapustin
**Release Target:** Australia
**Last Updated:** 2026-05-25

## Executive Summary

CI Auto Tagger is a ServiceNow scoped application that automates the classification and governance tagging of Configuration Items (CIs) in the CMDB. It uses a multi-pass analysis pipeline — table inheritance verification, regex pattern matching, attribute scoring, and relationship graph inference — to assign accurate CI classes and standardized operational tags. The application operates within the ServiceNow security boundary, reading CMDB tables through GlideRecord and writing classification results to dedicated application tables.

## Component Architecture

### Script Includes (Business Logic Layer)

| Script Include | Type | Lines | Responsibility |
|---------------|------|-------|----------------|
| `CIClassifierEngine` | Core Engine | ~300 | Orchestrates batch CI retrieval, pipelining through rule engine, confidence scorer, and tag engine |
| `CIRuleEngine` | Rule Processor | ~250 | Loads active classification rules, executes regex matching against CI attributes, performs relationship graph inference |
| `CITagEngine` | Tag Writer | ~180 | Normalizes tag values, writes classifications and tags back to CI records, handles field-level ACL checks |
| `CIConfidenceScorer` | Scoring | ~120 | Computes per-CI confidence scores using rule match strength, attribute completeness, and historical accuracy |
| `CIReportGenerator` | Reporting | ~200 | Transforms finding records into HTML dashboards, JSON exports, and CSV reports |
| `CIAuditLogger` | Audit | ~100 | Captures before/after snapshots of every automated change, supports rollback by scan run ID |
| `CIUtils` | Utilities | ~80 | Shared helpers: batch iterator, field validator, GlideDateTime formatter |

### Application Tables (Data Layer)

| Table | Label | Purpose |
|-------|-------|---------|
| `x_sn_ci_auto_tagger_rule` | Classification Rule | Defines match conditions (regex patterns, field comparisons, relationship criteria) and target class/tag assignments |
| `x_sn_ci_auto_tagger_scan_run` | Scan Run | Tracks each scan execution: type (full/incremental/on-demand), start/end time, records processed, records modified |
| `x_sn_ci_auto_tagger_finding` | Finding | Per-CI classification result with old/new class, confidence score, applied flag, scan run reference |
| `x_sn_ci_auto_tagger_audit` | Audit Log | Before/after field snapshots for every automated change, rollback ID, timestamp |
| `x_sn_ci_auto_tagger_config` | Configuration | Application settings: default confidence threshold (0.85), batch size (100), scan schedule |

### Integration Points

| Integration | Direction | Protocol | Purpose |
|------------|-----------|----------|---------|
| CMDB Tables (`cmdb_ci`, task extensions) | Read | GlideRecord | Source data for classification |
| Scheduled Jobs (`sys_trigger`) | Internal | Platform | Daily incremental and weekly full scans |
| REST Message (optional) | Outbound | HTTPS REST | Push findings to external CI/CD or SIEM |
| Email Notification | Outbound | SMTP | Scan completion alerts to CMDB administrators |
| Now Assist / AI Agent Studio (planned v2.0) | Outbound | REST API | AI-assisted classification suggestions |

## Data Flow

```
                         ┌─────────────────────┐
                         │  Scheduled Job       │
                         │  (sys_trigger)       │
                         └──────────┬──────────┘
                                    │ triggers
                                    ▼
┌──────────────────┐     ┌─────────────────────┐     ┌──────────────────┐
│  cmdb_ci         │────▶│  CIClassifierEngine  │────▶│  CIRuleEngine    │
│  cmdb_ci_server  │     │  (batch GlideRecord) │     │  (regex + score) │
│  cmdb_ci_appl    │     └─────────────────────┘     └────────┬─────────┘
└──────────────────┘                                          │
                                                              ▼
┌──────────────────┐     ┌─────────────────────┐     ┌──────────────────┐
│  x_sn_ci_auto_   │◀────│  CITagEngine         │◀────│  CIConfidence    │
│  tagger_finding  │     │  (write tags to CI)  │     │  Scorer          │
└──────────────────┘     └──────────┬──────────┘     └──────────────────┘
                                    │
                                    ▼
                          ┌─────────────────────┐
                          │  CIAuditLogger       │
                          │  (before/after snap) │
                          └──────────┬──────────┘
                                     │
                                     ▼
                          ┌─────────────────────┐
                          │  CIReportGenerator   │
                          │  (HTML/JSON/CSV)     │
                          └─────────────────────┘
```

## Confidence Gating Strategy

| Confidence Range | Action | Review Required |
|-----------------|--------|-----------------|
| 0.85–1.00 | Auto-apply classification and tags | No |
| 0.50–0.84 | Apply with review flag | Yes — CMDB admin review queue |
| 0.00–0.49 | Defer to manual classification | Yes — manual classification required |

The default threshold (0.85) is configurable via `x_sn_ci_auto_tagger_config.default_confidence_threshold`.

## Platform Compatibility

- **Minimum:** Utah
- **Recommended:** Zurich
- **Target:** Australia
- **UI:** Next Experience / Configurable Workspace compatible
- **Mobile:** Service Operations Workspace compatible (read-only dashboard)

## Performance Characteristics

| CMDB Size | Full Scan Duration | Incremental Scan Duration |
|-----------|-------------------|--------------------------|
| < 10,000 CIs | < 30 seconds | < 5 seconds |
| 10,000–50,000 CIs | 1–3 minutes | < 15 seconds |
| 50,000–200,000 CIs | 5–15 minutes | < 60 seconds |
| 200,000–500,000 CIs | 15–40 minutes | < 3 minutes |

Batch size (default 100) is tunable per environment to balance throughput vs. instance load.
