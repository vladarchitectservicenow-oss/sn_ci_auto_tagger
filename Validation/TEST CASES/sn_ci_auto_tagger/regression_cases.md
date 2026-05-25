# Regression Cases: sn_ci_auto_tagger

**Product:** ServiceNow CI Auto Tagger
**Scope:** `x_sn_ci_auto_tagger`
**Author:** Vladimir Kapustin

## Regression Test Cases

### 1. Idempotent Scan Execution
**Scenario:** Run a full classification scan twice on an unchanged CMDB.
**Expected:** Both scans produce identical finding counts, identical classification assignments, and identical confidence scores. No duplicate audit entries. Second scan's `records_modified` count is 0 (all CIs already correctly classified).

### 2. Report Format Consistency
**Scenario:** Generate JSON and Markdown reports from the same scan result.
**Expected:** JSON report has correct schema with `total`, `items`, `classified`, `deferred` keys. Markdown report contains matching record count in header. Both files are valid (JSON parses without error, Markdown renders without broken links).

### 3. Role Provisioning Idempotency
**Scenario:** Assign `x_sn_ci_auto_tagger.admin` role to a user who already has it.
**Expected:** Operation succeeds without error. User still has exactly one instance of the role. No duplicate role assignment record created.

### 4. Configuration Persistence Across Restarts
**Scenario:** Set `default_confidence_threshold = 0.75` in application settings. Simulate node restart (cache clear).
**Expected:** Value persists in `x_sn_ci_auto_tagger_config` table. Application reads correct value on next scan. No revert to default (0.85).

### 5. Classification Stability Across Instance Clones
**Scenario:** Clone a production instance to a sub-production instance. Run CI Auto Tagger on the clone.
**Expected:** Clone scan produces identical classification results as the source instance scan. No regression due to sys_id changes or reference field rewrites.

### 6. Rule Modification Without Side Effects
**Scenario:** Deactivate an existing classification rule, add a new rule, then run a full scan.
**Expected:** Only the deactivated rule's classifications are absent. New rule's classifications appear. No orphan findings from deactivated rule. No rule ID collision.

### 7. Batch Boundary Consistency
**Scenario:** Run scan with batch_size=100 on 250 CIs. Compare to batch_size=50 on same 250 CIs.
**Expected:** Total findings count is identical regardless of batch size. Classification results are identical. No records skipped at batch boundaries.

### 8. Concurrent Read-Only Access
**Scenario:** Generate JSON report while a classification scan is in progress.
**Expected:** Report generation succeeds (reads finding table, not scan-in-progress state). Report includes findings from all completed CIs up to that point. No deadlock or timeout.

### 9. Upgrade Survival — Schema Migration
**Scenario:** Upgrade ServiceNow instance from Zurich to Australia. Migrate application tables via sys_upgrade_history.
**Expected:** All `x_sn_ci_auto_tagger_*` tables survive with data intact. Existing findings remain valid. Scan on upgraded instance produces results consistent with pre-upgrade baseline (within 1% drift).

### 10. Token/Password Rotation
**Scenario:** Change ServiceNow instance admin password. Update CLI tool credentials. Re-run scan.
**Expected:** Old credentials rejected with 401. New credentials accepted. Scan results match pre-rotation baseline. No data loss from credential rotation.
