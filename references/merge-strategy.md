# Merge Strategy Decision Guide

Use repository policy first. This guide is the fallback when the repository does not prescribe a strategy.

## Decision matrix

| Strategy | Choose when | Avoid when |
|---|---|---|
| Squash merge | One PR represents one logical change; branch contains fixups, review corrections, or noisy commits; a compact base history is desired | Commits need independent rollback, distinct deployment units, separate audit/provenance, or separate authorship must remain visible |
| Merge commit | Branch topology matters; the PR contains multiple meaningful commits; release/integration history or independent revert points must remain visible | The repository requires linear history or the branch is only implementation noise |
| Rebase + fast-forward | The repository requires linear history and the branch is private; commits are already clean and independently meaningful | The branch is shared, public, already consumed by others, or rewriting it would invalidate review references |
| Rebase merge | The repository explicitly requires it and every commit is independently reviewable, tested, and useful in the base history | The PR has fixups/noise or the team expects one commit per PR |

## Selection procedure

1. Read branch protection, contribution rules, merge queue settings, and CODEOWNERS requirements.
2. Check whether the branch is private or shared and whether rewriting it would disrupt collaborators or review links.
3. Decide whether the PR is one logical change or several independently meaningful changes.
4. Check whether commits need independent revert, deployment, audit, authorship, or signature provenance.
5. Select the repository-mandated strategy. If no mandate exists, use squash for a focused PR and merge commit when history topology matters.
6. Record the chosen strategy and reason in the PR.

## Commit identity and signature effects

Squash, rebase, amend, cherry-pick, and merge commits create new commit objects. Always verify:

```text
new commit SHA
parent commit(s)
author and committer
gpgsig/SSH/S/MIME signature
GitHub Verified or Unverified status
```

An original signed commit does not make a recreated commit signed. A GitHub-created merge or squash commit may be signed by GitHub, while the original local commits remain unsigned. Do not select a merge strategy solely to produce a `Verified` badge; use the required signing configuration and verify the resulting commit.

## Post-merge checks

- Confirm the resulting commit and parent graph match the selected strategy.
- Confirm CI, deployment, migration, and smoke checks on the base branch.
- Confirm the PR is merged rather than merely closed.
- Keep the original branch until the merge is confirmed and rollback is understood.
- Delete the branch only when repository policy permits it.
