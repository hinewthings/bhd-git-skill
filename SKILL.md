---
name: bhd-git-skill
description: Enterprise Git and GitHub workflow for software changes across repositories. Use before, during, and after bug fixes, features, refactors, hotfixes, security patches, dependency upgrades, database migrations, API changes, UI changes, worker jobs, configuration or infrastructure changes, tests, docs, reverts, commits, pull requests, reviews, merges, releases, or branch cleanup.
---

# Enterprise Git Workflow

Apply this skill to make repository changes traceable, reviewable, testable, and safe. Treat the repository's own instructions, contribution guide, CI rules, branch protection, and team conventions as higher priority than the defaults in this skill.

## Operating modes

Infer the mode from the user's request and the repository state:

- **Preflight**: before implementation; inspect the repository, classify the change, assess risk, and produce the workflow contract.
- **Implement**: when the user asks to fix, build, refactor, or otherwise change code; follow the preflight contract and preserve unrelated work.
- **Finalize**: when the user asks to commit, push, open a PR, or finish the change; validate, stage only intended files, commit, and push as authorized.
- **Review**: when the user asks for review, audit the diff and checks without mutating files unless explicitly asked to fix findings.
- **Merge**: only merge when the user explicitly asks for merge and all gates pass.
- **Release**: only deploy or release when the user explicitly asks; use the repository's CI/CD path and documented rollback procedure.

If the request is ambiguous, perform read-only preflight and state the next required action. Do not silently commit, push, merge, deploy, reset, discard, or overwrite work.

## Repository discovery

Start from the repository root and inspect the minimum relevant context:

```text
git rev-parse --show-toplevel
git status --short --branch
git branch --show-current
git remote -v
git log -5 --oneline --decorate
```

Read applicable instructions before changing anything:

- `AGENTS.md`, `CONTRIBUTING.md`, `README`, engineering standards, and local skill instructions.
- `.github/` workflows, pull request templates, issue templates, CODEOWNERS, and dependabot configuration.
- The package/build manifests and scripts used by the repository (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Makefile`, and similar).
- Migration, deployment, security, testing, and release documentation when the change touches those areas.

Determine the protected/default branch from repository configuration and remote HEAD. Do not assume it is named `main`. Check for uncommitted or untracked work. Preserve unrelated changes; never include them merely because they are present.

## Workflow contract

Before implementation, output a concise contract containing:

1. Change type and risk level.
2. Base branch and proposed branch name.
3. Files/modules likely in scope and explicitly out of scope.
4. Required design or repository documents to read.
5. Required validation: lint, typecheck, unit/integration/e2e tests, build, migration, security, or manual checks.
6. Commit structure and proposed Conventional Commit messages, unless the repository uses another convention.
7. PR, reviewer, rollout, monitoring, and rollback requirements.
8. Blockers, unknowns, or approvals needed before implementation.

Use a branch for every non-trivial change. Follow the repository's naming convention. If none exists, use:

```text
codex/feat/<short-slug>
codex/fix/<short-slug>
codex/refactor/<short-slug>
codex/hotfix/<short-slug>
codex/chore/<short-slug>
```

Create the branch from the latest protected/default branch. If the working tree contains user changes, do not reset or stash them without permission; either work around them or ask for direction.

## Change classification

Select the narrowest applicable workflow. Read `references/change-matrix.md` for the detailed risk and validation matrix.

- **Bug fix**: reproduce or identify the failure, add a regression test where practical, make the smallest safe fix, and verify the original scenario plus adjacent behavior.
- **Feature**: define acceptance criteria, preserve backward compatibility, implement contracts before deep UI, add tests, documentation, telemetry, and feature-flag/rollout controls when appropriate.
- **Refactor**: prove behavior should remain unchanged, keep commits behavior-preserving, compare tests/build output, and do not mix unrelated feature changes.
- **Hotfix**: branch from the production-supported base, minimize scope, add a regression test, record incident context, use expedited review, and plan the follow-up cleanup.
- **Security patch**: avoid exposing vulnerability details in public commits, rotate or revoke compromised credentials, run secret/dependency scans, require security review, and document remediation and rollback.
- **Dependency/toolchain upgrade**: inspect breaking changes and license/security impact, update lockfiles intentionally, run the full relevant test/build matrix, and document compatibility changes.
- **Database/schema/migration**: make migrations forward-compatible where possible, avoid destructive operations in the same release as code that needs the old schema, test on a copy or ephemeral database, document rollback/data recovery, and require backup/approval for production.
- **API/contract change**: identify all consumers, preserve compatibility or version deliberately, update schemas/SDKs/docs, add contract tests, and coordinate rollout order.
- **UI/accessibility/localization**: cover loading, empty, error, partial, and responsive states; use the design system and i18n; run accessibility and visual checks when available.
- **Worker/queue/data job**: make jobs idempotent, bounded, retry-safe, observable, and safe during deploys; test duplicate, timeout, partial-failure, and replay behavior.
- **Configuration/infrastructure/deployment**: separate code from environment config, protect secrets, review blast radius, validate in a non-production environment, and define rollback and smoke checks.
- **Tests/docs/chore**: keep the change focused, run the affected checks, and avoid disguising behavior changes as maintenance.
- **Revert/cherry-pick/backport**: identify the original commit and target branch, check dependency order and conflict risk, explain why the operation is safe, and verify the target branch after applying it.

## Risk gates

Classify the highest-risk affected area, not merely the number of changed lines:

- **Low**: isolated docs/tests, formatting, or behavior-preserving local refactor. Run targeted checks and repository-required CI.
- **Medium**: user-facing behavior, normal API/UI/worker changes, or non-sensitive configuration. Run targeted plus integration/build checks, update docs, and request code-owner review when applicable.
- **High**: authentication/authorization, payments, PII, secrets, data deletion, destructive migration, production infrastructure, security remediation, billing, permissions, or high-volume data jobs. Require explicit acceptance criteria, threat/impact review, rollback or recovery plan, staged rollout/feature flag where possible, required reviewers, and post-deploy smoke/monitoring checks.

When risk is unclear, use the higher level until evidence lowers it. Never bypass a repository's required check merely because the diff is small.

## Implementation discipline

Keep changes small and logically separable. Before editing, record the intended scope. During implementation:

- Follow existing architecture, naming, formatting, and test conventions.
- Keep database, backend contracts, permissions, and validation coherent with the UI.
- Avoid secrets, hardcoded environment-specific values, generated artifacts, unrelated formatting churn, and opportunistic cleanup.
- Preserve workspace/tenant isolation, authorization, input validation, and auditability when relevant.
- Update tests and docs at the same time as behavior changes.
- Add or update feature flags, migrations, telemetry, and rollback paths when the change warrants them.
- Do not edit or delete unrelated user changes.

For a bug, prefer this order: reproduce or document the failure → add regression coverage → fix → verify adjacent cases. For a feature: acceptance criteria → contract/design → implementation → tests → docs/rollout. For a refactor: characterization tests → mechanical change → behavior comparison → cleanup.

## Verification gates

Discover and use the repository's own commands instead of inventing replacements. At minimum, run the applicable checks:

```text
git diff --check
lint
typecheck/static analysis
unit tests
integration/contract tests
end-to-end tests
build/package validation
security/secret/dependency scan
database migration or schema validation
manual smoke test
```

Do not claim a check passed unless it was actually run. Report skipped checks with the reason. Review the final diff and status after tests because tools may generate files.

For UI changes, check responsive, accessibility, loading, empty, error, and localization states. For API changes, check validation, authorization, compatibility, error contracts, and consumer impact. For data changes, check idempotency, rollback/recovery, backups, and migration order.

## Commit protocol

Before committing:

```text
git status --short
git diff
git diff --stat
git diff --check
```

Stage only files belonging to the current change. Prefer interactive or explicit staging:

```text
git add -p
git diff --cached
git commit -m "fix(auth): handle expired access token"
```

Use Conventional Commits by default:

```text
feat(scope): add capability
fix(scope): correct incorrect behavior
refactor(scope): preserve behavior while restructuring
test(scope): add coverage
docs(scope): update guidance
chore(scope): maintain tooling
build(scope): change build/dependency behavior
ci(scope): change automation
perf(scope): improve performance
revert: revert <commit subject>
```

Use `BREAKING CHANGE:` or the repository's required notation for incompatible changes. Write the commit subject as a concise description of the actual change. Keep one logical change per commit when practical. Do not skip hooks with `--no-verify` unless the user explicitly authorizes it and the reason is recorded.

## Push and pull request protocol

Before pushing, synchronize with the target branch without losing work:

```text
git fetch origin
git log --oneline HEAD..origin/<base-branch>
```

Rebase a private feature branch or merge according to repository policy. Do not rewrite a shared branch. If rewriting a private branch is necessary, use `git push --force-with-lease`, never plain `--force`.

Push only the intended branch:

```text
git push -u origin <branch-name>
```

Open a PR using the repository's template. If no template exists, use `references/pr-template.md`. Include problem, solution, scope, risk, tests, migration/config changes, screenshots for UI, rollout, monitoring, rollback, and reviewer requests. Mark the PR draft while incomplete or while high-risk validation is missing.

Prefer the repository's GitHub connector or `gh` for PR metadata and review state when available. Use Git for local history and file operations. Never expose tokens or paste secrets into issues, PRs, logs, or command output.

## Review and merge protocol

Before merge, verify:

- Required CI checks are green.
- Required approvals and CODEOWNERS reviews are complete.
- No unresolved blocking review threads remain.
- The PR diff contains only intended changes.
- Migration, rollout, monitoring, and rollback requirements are satisfied.
- The source branch is up to date according to repository policy.

Merge only after the user explicitly requests it or an approved automation policy clearly authorizes it. Never merge failing checks, unresolved conflicts, unresolved blocking comments, or an unreviewed high-risk change. Use the repository's mandated strategy; otherwise prefer squash merge for a focused PR. Never force-push or delete a branch before confirming the merge completed.

After merge, verify the target branch, CI/deployment status, migration result, and smoke checks. Delete the remote branch only when safe and permitted. Keep a rollback/revert path until the change is confirmed healthy.

## Safety rules

- Never run `git reset --hard`, `git checkout --`, `git restore`, `git clean`, branch deletion, force-push, migration rollback, or data deletion without confirming exact targets and explicit authorization when the action can discard work or data.
- Never stage all files blindly with `git add .` in a dirty worktree.
- Never commit secrets, credentials, personal data, generated build output, or local machine settings.
- Never hide a failing test, lint error, hook, review comment, or CI failure.
- Never merge directly into a protected branch when the repository requires PRs.
- If a conflict, dirty worktree, missing credential, failing check, unclear ownership, or ambiguous target prevents safe progress, stop at the safe boundary and report the exact blocker and next action.

## Required final report

After any finalize/review/merge operation, report:

```text
Change: <type and short summary>
Branch: <name>
Commit(s): <hash and subject, if created>
PR: <number/link/status, if created>
Checks: <passed / skipped with reasons / failed>
Database/config/release: <what changed or not applicable>
Remaining risk or next action: <exact action>
```

Keep the report factual and distinguish local validation from CI validation and deployed status.

## References

- Read [change-matrix.md](references/change-matrix.md) when classifying change type, risk, required checks, reviewers, or rollout gates.
- Read [pr-template.md](references/pr-template.md) when creating or reviewing a pull request without a repository-specific template.
