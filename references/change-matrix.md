# Enterprise Change Matrix

Use the highest applicable row. Repository-specific policies override this default matrix.

| Change | Branch | Minimum validation | Extra gates |
|---|---|---|---|
| Bug fix | `codex/fix/*` | Reproduction, regression test, targeted tests, lint/typecheck | Verify adjacent behavior; include impact and rollback |
| New feature | `codex/feat/*` | Acceptance tests, unit/integration tests, build | API/docs/i18n/telemetry; feature flag for risky rollout |
| Refactor | `codex/refactor/*` | Characterization tests, full affected tests, build | No behavior drift; keep commits separable |
| Hotfix | `codex/hotfix/*` | Focused regression test, smoke test, CI | Expedited approval; incident note; follow-up cleanup |
| Security patch | `codex/fix/*` or team security prefix | Regression, secret scan, dependency scan, security tests | Security reviewer; private handling; rotation/revocation; rollback |
| Dependency upgrade | `codex/chore/*` or `codex/build/*` | Lockfile review, tests, typecheck, build, security/license scan | Breaking-change review; runtime compatibility |
| Database migration | `codex/feat/*` or `codex/fix/*` | Migration up/down or recovery test, schema test, app tests | Backup, compatibility window, rollback/data recovery, production approval |
| API/contract change | `codex/feat/*` or `codex/fix/*` | Contract tests, consumer tests, docs, build | Version/deprecation plan; rollout order |
| UI/accessibility/i18n | `codex/feat/*` or `codex/fix/*` | Unit/component tests, lint/typecheck, manual responsive check | Accessibility, localization, screenshots/visual QA |
| Worker/queue/data job | `codex/feat/*` or `codex/fix/*` | Retry/idempotency/duplicate/timeout tests, integration test | Monitoring, dead-letter/replay plan, safe deploy order |
| Infrastructure/config/deploy | `codex/chore/*` or team prefix | Dry-run/plan, environment validation, smoke test | Secret review, blast-radius review, rollback |
| Docs/tests/chore | `codex/docs/*`, `codex/test/*`, or `codex/chore/*` | Relevant doc/test/tool checks | Do not hide behavior changes in maintenance commits |
| Revert/backport/cherry-pick | Repository convention | Target-branch tests and diff review | Verify dependency order and explain reason |

## Risk classification

### Low

Use for isolated documentation, tests, formatting, or behavior-preserving local refactors. Require targeted validation and normal CI.

### Medium

Use for normal user-facing, API, worker, UI, dependency, or non-sensitive configuration changes. Require targeted tests plus integration/build validation and appropriate review.

### High

Use for authentication, authorization, billing, payments, PII, secrets, data deletion, destructive migrations, production infrastructure, security incidents, permissions, or high-volume data jobs. Require explicit acceptance criteria, impact/threat review, recovery or rollback, staged rollout or feature flag when possible, required code owners, and post-change monitoring.

## Commit and PR examples

```text
fix(auth): reject expired refresh token
feat(products): add catalog synchronization
refactor(worker): extract retry policy without behavior change
feat(db): add nullable commission status column
build(deps): upgrade framework patch release
```

For high-risk changes, the PR body must state who approved the change, how it rolls out, how it is monitored, and how it is reverted or recovered.
