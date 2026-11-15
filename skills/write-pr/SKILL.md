---
name: write-pr
description: Write a structured PR description (10-section template) from the current branch's commits and diff, then create or update the PR via gh.
allowed-tools: >-
  Read, Write,
  Bash(git log:*), Bash(git diff:*), Bash(git status:*), Bash(git branch:*),
  Bash(git rev-parse:*), Bash(git remote:*), Bash(git show-ref:*),
  Bash(gh repo view:*), Bash(gh pr view:*), Bash(gh pr list:*),
  Bash(gh pr create:*), Bash(gh pr edit:*)
argument-hint: "[base branch and/or PR number — optional]"
---

# Write a structured PR description

Generate a PR body in the team's 10-section template, then create or update the PR.

## Steps

1. **Resolve the base branch.**
   - If the user passed one in `$ARGUMENTS`, use it.
   - Else use the repo default: `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.
   - Many teams integrate on `develop` — if a local `develop` branch (or `origin/develop`) exists, prefer it over `main` unless the user says otherwise. When ambiguous, ask.

2. **Gather context** (read-only) for the current branch vs base:
   - `git log --oneline <base>..HEAD`
   - `git diff --stat <base>...HEAD`
   - Read the actual diff of the key files (`git diff <base>...HEAD -- <path>`) enough to describe behavior, decisions, and risks. Don't dump the diff into the PR; summarize it.
   - If an existing PR is open for this branch, read it first: `gh pr view --json number,title,body`.

3. **Judge the size.** Not every section applies to every PR. **Omit** a section entirely when it doesn't apply — don't emit a header with "N/A" under it. Never invent content to fill a section.
   - **Always include**: "What problem does this solve?", "How was it solved?", "Review points", "Verification / Test results" (even just "build passes", if actually verified), using the corresponding headers in the selected language. These form the minimum skeleton of every PR.
   - **Include when applicable**: the rest. "Limitations & tradeoffs", "Edge cases & failure scenarios", "Alternatives considered & why this choice", and "Follow-ups & required actions" are usually omitted on a small PR and essential on a substantial one — include them when they're real, don't skip them just because they take effort.
   - **Trivial** (typo, one-line, rename, copy tweak): keep it short; usually just the always-include sections.
   - **Substantial** (new feature, schema change, multiple decisions, cross-cutting): the "when applicable" sections usually have real content — especially Alternatives considered, Verification, and Follow-ups. Make sure you didn't omit a section that genuinely applies.

4. **Choose the body language.**
   - Follow any explicit request for the PR's output language.
   - Otherwise, use **Korean** if the user's preferred conversation language (or, if unstated, their latest substantive message) is Korean; use **English** in all other cases, including when language context is unavailable.
   - Use the same language for section headers and prose; preserve code identifiers, paths, commands, and URLs.

5. **Write the body** using the selected language's section headers below, in this order. Translate the headers for other explicitly requested languages.

   **English:**

```
# What problem does this solve?
# Why does it need to be done?
# How was it solved?
# Impact on existing behavior
# Edge cases & failure scenarios
# Limitations & tradeoffs
# Alternatives considered & why this choice
# Verification / Test results
# Review points (Risk 🔴🟡🟢 by file/area)
# Follow-ups & required actions
```

   **Korean:**

```
# 어떤 문제를 해결하나요?
# 왜 필요한가요?
# 어떻게 해결했나요?
# 기존 동작에 미치는 영향
# 엣지 케이스 및 실패 시나리오
# 한계 및 트레이드오프
# 검토한 대안과 선택 이유
# 검증 및 테스트 결과
# 리뷰 포인트 (파일/영역별 위험도 🔴🟡🟢)
# 후속 작업 및 필수 조치
```

   Guidance per section (applies to the corresponding section in every language):
   - **What problem does this solve? / Why does it need to be done?**: the concrete problem and motivation, not a feature restatement.
   - **How was it solved?**: layered bullet summary (domain / DB / API / FE, etc.), reference real paths/functions.
   - **Impact on existing behavior**: backward compatibility, default behavior, migrations, auto-trigger paths.
   - **Edge cases & failure scenarios**: validation failures, empty/null, legacy rows, error responses.
   - **Limitations & tradeoffs**: what this PR intentionally does NOT do, known debt.
   - **Alternatives considered & why this choice**: the real forks you weighed and why you chose what you did. This is the highest-value section — capture decisions that would otherwise be lost.
   - **Verification / Test results**: the evidence reviewers need to trust the change. Unit/build pass, behavior or output comparisons (before/after, old vs new), numerical results where relevant, manual verification against a real run. State only what was actually run; never claim verification that wasn't done.
   - **Review points**: list changed files/areas, each tagged 🔴 (high risk) / 🟡 (review carefully) / 🟢 (low risk), with a one-line why. Point reviewers at what matters.
   - **Follow-ups & required actions**: actions required outside this diff before/after merge — manual DB migrations (give the exact command), migration-before-deploy ordering (⚠️), config/env changes, and follow-up work in other repos. Deploy steps live here.

6. **Create or update the PR.**
   - Write the body to a temp file using the **Write tool** (not a shell heredoc), then pass it with `--body-file` (avoids quoting issues).
   - New PR: `gh pr create --base <base> --title "<conventional title>" --body-file <file>`.
   - Existing PR: `gh pr edit <number> --body-file <file>`.
   - Confirm the base branch with the user before creating if it wasn't explicitly given.
   - Optionally end the body with a trailer line: `🤖 Generated with [Claude Code](https://claude.com/claude-code)` — omit it if the repo/org disables AI attribution or the user prefers none.

## Notes
- Keep prose tight; reviewers skim. Prefer bullets over paragraphs.
- Use repo-relative file links/paths so they're clickable.
- Don't invent risks, alternatives, or test results — only list ones that are real for this diff.
