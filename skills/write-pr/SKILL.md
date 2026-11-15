---
name: write-pr
description: Write a concise, reviewer-focused PR description (five core sections, plus optional ones only when needed) from the current branch's commits and diff, then create or update the PR via gh.
allowed-tools: >-
  Read, Write,
  Bash(git log:*), Bash(git diff:*), Bash(git status:*), Bash(git branch:*),
  Bash(git rev-parse:*), Bash(git remote:*), Bash(git show-ref:*),
  Bash(gh repo view:*), Bash(gh pr view:*), Bash(gh pr list:*),
  Bash(gh pr create:*), Bash(gh pr edit:*)
argument-hint: "[base branch and/or PR number — optional]"
---

# Write a structured PR description

Generate a PR body for the reviewer — five core sections, plus optional ones only when they carry something the reviewer needs — then create or update the PR.

The reader is the reviewer. Every line should help them review; anything they can see in the diff, or would not act on, stays out.

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

3. **Decide the PR type.** It selects the headers for the first section and the "how" section. Ask: *was something broken or painful before this change?*
   - **Fix / improvement** — yes: a bug fix, or a change that removes a defect or pain point in existing behavior. Use the *problem* headers.
   - **Addition / setup** — no: initial project setup, a new feature, installing a plugin or dependency, configuration, docs, CI. Use the *addition* headers.
   - Never invent a problem to justify an addition or setup PR. "There was no X" is not a problem statement; just say what the PR adds.
   - If the PR mixes both, go with what most of the diff is.

4. **Choose the sections.** Five are core; the other five are left out unless they pass their test.
   - **Core — always include:** the first section (what), the "how" section, "Impact on existing behavior", "Verification / Test results", "Review points". Most PRs are complete with just these five.
   - "Impact on existing behavior" stays even when there is no impact: write one line saying so and why (e.g. "None — only adds new files", "None — new repository"). This is the only section that may say "none".
   - **Optional — omit by default.** Include one only when its test passes:
     - **Why does it need to be done?** — the motivation isn't already clear from the first section.
     - **Edge cases & failure scenarios** — there is a failure path the reviewer would miss from the diff alone.
     - **Limitations & tradeoffs** — something was deliberately left out that a reviewer would ask about.
     - **Alternatives considered & why this choice** — you actually weighed a different approach.
     - **Follow-ups & required actions** — someone has to do something outside this diff before or after merge. When that is true, always include it.
   - When in doubt, ask: *would the reviewer ask a question, or review this wrongly, without the section?* If not, omit it.
   - Omit a section entirely — no header with "N/A" under it. Never invent content to fill a section.

5. **Choose the body language.**
   - Follow any explicit request for the PR's output language.
   - Otherwise, use **Korean** if the user's preferred conversation language (or, if unstated, their latest substantive message) is Korean; use **English** in all other cases, including when language context is unavailable.
   - Use the same language for section headers and prose; preserve code identifiers, paths, commands, and URLs.

6. **Write the body** using the selected language's section headers below, in this order. For the two lines that show `A / B`, write only one header: `A` for a fix / improvement PR, `B` for an addition / setup PR. Translate the headers for other explicitly requested languages.

   **English:**

   ```
   # What problem does this solve?  /  # What does this add?
   # Why does it need to be done?  (optional)
   # How was it solved?  /  # How is it set up?
   # Impact on existing behavior
   # Edge cases & failure scenarios  (optional)
   # Limitations & tradeoffs  (optional)
   # Alternatives considered & why this choice  (optional)
   # Verification / Test results
   # Review points (Risk 🔴🟡🟢 by file/area)
   # Follow-ups & required actions  (optional)
   ```

   **Korean:**

   ```
   # 어떤 문제를 해결하나요?  /  # 무엇을 추가하나요?
   # 왜 필요한가요?  (선택)
   # 어떻게 해결했나요?  /  # 어떻게 구성했나요?
   # 기존 동작에 미치는 영향
   # 엣지 케이스 및 실패 시나리오  (선택)
   # 한계 및 트레이드오프  (선택)
   # 검토한 대안과 선택 이유  (선택)
   # 검증 및 테스트 결과
   # 리뷰 포인트 (파일/영역별 위험도 🔴🟡🟢)
   # 후속 작업 및 필수 조치  (선택)
   ```

   The `(optional)` / `(선택)` marks are for you; never write them in the PR body.

   Guidance per section (applies to the corresponding section in every language):
   - **What problem does this solve? / What does this add?**: one or two sentences. For a fix, the concrete problem; for an addition or setup, what now exists that didn't before.
   - **Why does it need to be done?**: the motivation, only where the first section leaves it open.
   - **How was it solved? / How is it set up?**: the approach and the key decisions, in a few bullets. Not a file-by-file walkthrough of the diff.
   - **Impact on existing behavior**: what changes for existing users or callers — compatibility, defaults, migrations. One to three lines, or one line saying there is none and why.
   - **Edge cases & failure scenarios**: the non-obvious failure paths and how they are handled.
   - **Limitations & tradeoffs**: what this PR intentionally does NOT do, known debt.
   - **Alternatives considered & why this choice**: the real forks you weighed and why you chose what you did.
   - **Verification / Test results**: what was actually run and what it showed, in a line or two. If nothing was verified, say so in one line; never claim verification that wasn't done.
   - **Review points**: list the 🔴 (high risk) and 🟡 (review carefully) files/areas individually, each with a one-line why. Fold 🟢 (low risk) into a single line, or leave it out. Don't list every changed file.
   - **Follow-ups & required actions**: actions required outside this diff before/after merge — manual DB migrations (give the exact command), migration-before-deploy ordering (⚠️), config/env changes, and follow-up work in other repos. Deploy steps live here.

7. **Create or update the PR.**
   - Write the body to a temp file using the **Write tool** (not a shell heredoc), then pass it with `--body-file` (avoids quoting issues).
   - New PR: `gh pr create --base <base> --title "<conventional title>" --body-file <file>`.
   - Existing PR: `gh pr edit <number> --body-file <file>`.
   - Confirm the base branch with the user before creating if it wasn't explicitly given.
   - Optionally end the body with a trailer line: `🤖 Generated with [Claude Code](https://claude.com/claude-code)` — omit it if the repo/org disables AI attribution or the user prefers none.

## Notes
- Keep it short. Aim for one to four bullets per section, one line per bullet. A trivial PR's whole body can be a handful of lines.
- Don't restate what the diff already shows. Write what the reviewer can't get from reading the code: intent, decisions, risk, evidence.
- A longer body is fine when the change needs it; a longer body that repeats itself or fills sections for completeness is not.
- Use repo-relative file links/paths so they're clickable.
- Don't invent problems, risks, alternatives, or test results — only list ones that are real for this diff.
