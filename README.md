# `/bhd-git-skill`

Skill quy trình Git/GitHub chuẩn enterprise, dùng cho nhiều dự án và nhiều loại thay đổi: fix bug, thêm feature, refactor, hotfix, security patch, migration, API, UI, worker, infrastructure, commit, PR và merge.

## Mục tiêu

Skill giúp biến mỗi thay đổi code thành một quy trình có thể kiểm tra và truy vết:

```text
Hiểu scope
→ chọn workflow
→ tạo branch
→ code/test
→ review diff
→ commit
→ push/PR
→ review
→ merge/release
```

Quy tắc riêng của repository, `AGENTS.md`, `CONTRIBUTING.md`, CI/CD, branch protection và CODEOWNERS luôn được ưu tiên hơn mặc định của skill.

## Cách gọi

Trong Codex có thể gọi bằng:

```text
/bhd-git-skill
```

Hoặc:

```text
$bhd-git-skill
```

Nên gọi skill ở hai thời điểm chính.

### 1. Trước khi code

```text
$bhd-git-skill
Tôi cần fix lỗi duplicate queue job. Hãy kiểm tra repository và đưa ra workflow trước khi tôi bắt đầu.
```

Skill sẽ kiểm tra:

- Repository, branch hiện tại, remote và trạng thái worktree.
- Các thay đổi chưa commit để tránh commit nhầm.
- Loại thay đổi và mức rủi ro.
- Branch/base branch phù hợp.
- Tài liệu, module và dependency cần đọc.
- Test, lint, build, security, migration hoặc manual check cần chạy.
- Kế hoạch commit, PR, reviewer, rollout và rollback.

### 2. Sau khi code xong

```text
$bhd-git-skill
Task đã hoàn tất. Hãy kiểm tra, commit và mở PR đúng quy trình.
```

Nếu muốn merge, phải nói rõ:

```text
$bhd-git-skill
CI đã pass và review đã hoàn tất. Hãy kiểm tra điều kiện rồi merge PR.
```

Skill không tự merge khi chưa có yêu cầu rõ ràng.

## Các loại task được hỗ trợ

| Loại task | Quy trình bổ sung |
|---|---|
| Bug fix | Reproduce, regression test, fix nhỏ nhất, kiểm tra case lân cận |
| Feature | Acceptance criteria, contract, test, docs, rollout/feature flag |
| Refactor | Characterization test, giữ nguyên behavior, kiểm tra regression |
| Hotfix | Branch production, scope nhỏ, review nhanh, follow-up cleanup |
| Security patch | Secret/dependency scan, security review, rotation và rollback |
| Dependency upgrade | Breaking changes, lockfile, compatibility, license/security |
| Database migration | Forward compatibility, migration test, backup, recovery |
| API/contract | Consumer impact, versioning, contract test, rollout order |
| UI/accessibility/i18n | Responsive, loading/empty/error, a11y, localization, visual QA |
| Worker/queue/data job | Idempotency, retry, duplicate, timeout, replay, monitoring |
| Config/infra/deploy | Dry-run, blast radius, secret review, smoke test, rollback |
| Test/docs/chore | Scope gọn, không che giấu behavior change |
| Revert/backport/cherry-pick | Kiểm tra commit gốc, dependency order, conflict và target branch |

## Mức rủi ro

- **Low**: docs, tests, formatting hoặc refactor cục bộ không đổi behavior.
- **Medium**: thay đổi API, UI, worker, dependency hoặc behavior người dùng.
- **High**: auth, permission, payment, billing, PII, secret, data deletion, destructive migration, production infrastructure hoặc security incident.

Task High phải có impact/threat review, rollback hoặc recovery plan, reviewer phù hợp, rollout có kiểm soát và post-deploy monitoring.

## Quy tắc commit

Mặc định dùng Conventional Commits, trừ khi repository đã có convention khác:

```text
feat(products): add product synchronization
fix(auth): handle expired access token
refactor(worker): extract retry policy
test(auth): add refresh token coverage
docs(workflow): update release guide
chore(deps): upgrade framework patch version
```

Nguyên tắc:

- Một commit nên tương ứng một logical change.
- Chỉ stage file thuộc task hiện tại.
- Kiểm tra `git diff --cached` trước khi commit.
- Không commit `.env`, token, API key, secret, PII, build output hoặc file máy cá nhân.
- Không dùng `--no-verify` nếu chưa được cho phép và ghi rõ lý do.

## Điều kiện trước khi mở PR

Tùy repository, skill sẽ chạy các kiểm tra phù hợp:

```text
git diff --check
lint
typecheck/static analysis
unit tests
integration/contract tests
end-to-end tests
build/package validation
security/secret/dependency scan
database/schema validation
manual smoke test
```

Skill không báo “đã pass” nếu chưa thực sự chạy. Check nào bỏ qua phải ghi rõ lý do.

PR cần có:

- Problem và root cause.
- Solution và scope included/excluded.
- Risk level.
- Tests/checks đã chạy.
- Database/config/secret impact.
- Rollout, monitoring và rollback.
- Screenshot/video nếu thay đổi UI.
- Reviewer hoặc CODEOWNER cần review.

## Chọn cách merge PR

Skill sẽ đọc policy của repository trước. Nếu repository không quy định riêng, dùng nguyên tắc sau:

| Strategy | Khi nên dùng |
|---|---|
| **Squash merge** | PR tập trung, commit có nhiều fixup/review noise, muốn `main` có một logical commit |
| **Merge commit** | Cần giữ branch topology, lịch sử release/integration hoặc các commit có thể revert độc lập |
| **Rebase + fast-forward** | Repository yêu cầu history tuyến tính và branch còn private, an toàn để rewrite |
| **Rebase merge** | Repository quy định rõ và từng commit đều độc lập, đã test và review được |

Không nên squash nếu các commit là các đơn vị deploy/release riêng, cần revert độc lập, hoặc repository yêu cầu giữ provenance/signature của từng commit.

Squash, rebase, amend và merge commit đều tạo commit object mới. Commit SHA, parent, committer và trạng thái `Verified` phải được kiểm tra lại; chữ ký của commit cũ không tự truyền sang commit mới.

## Quy tắc an toàn

Skill sẽ dừng và báo blocker khi:

- Worktree có thay đổi không rõ thuộc task nào.
- Có conflict hoặc branch target không rõ.
- CI fail, còn review blocking hoặc thiếu approval.
- Cần reset, xóa file, force-push, rollback migration hoặc xóa branch.
- Không xác định được credential hoặc remote GitHub.

Không dùng các lệnh phá hủy như `git reset --hard`, `git clean`, `git checkout --`, force-push thông thường hoặc `git add .` mù quáng.

## Cấu trúc repository

```text
bhd-git-skill/
├── SKILL.md                         # Instructions cho Codex
├── README.md                        # Hướng dẫn cho người dùng
├── agents/openai.yaml               # Metadata hiển thị trong Codex
└── references/
    ├── change-matrix.md             # Ma trận loại task/risk/check
    └── pr-template.md               # Template Pull Request
```

## Cài đặt dùng global

Skill được cài tại:

```text
C:\Users\Administrator\.codex\skills\bhd-git-skill
```

Khi dùng máy hoặc môi trường Codex khác, đặt thư mục skill vào:

```text
<CODEX_HOME>\skills\bhd-git-skill
```

Sau đó gọi `$bhd-git-skill` trước khi bắt đầu task.

## Tài liệu chi tiết

- [SKILL.md](SKILL.md): quy trình đầy đủ dành cho Codex.
- [change-matrix.md](references/change-matrix.md): phân loại thay đổi và risk gate.
- [merge-strategy.md](references/merge-strategy.md): cách chọn squash, merge commit, rebase và fast-forward.
- [pr-template.md](references/pr-template.md): template PR chuẩn.
