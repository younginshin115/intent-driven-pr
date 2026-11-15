# intent-driven-pr

[English](README.md) | [한국어](README.ko.md)

Write **intent-driven, decision-document PR descriptions** — a structured 10-section PR body generated from your branch's commits and diff, then create or update the PR via `gh`. Works as a shared skill across **Claude Code, Cursor, and Codex CLI**.

## Why

This skill was built after reading ["PR을 더 느리게 만들기 위한 고민" (KakaoPay Tech Blog)](https://tech.kakaopay.com/post/kakaopayins-slow-pr-fast-dev/), which argues that a slower, more deliberate PR description makes the *review* faster.

**The goal is to make reviewing easier.** Instead of a one-line "fixed the bug" PR, this skill surfaces the two things a reviewer most needs:

- **the intent of the change** — *why* it exists, *what* was decided and what was rejected, and *how* it was verified, and
- **where to look** — the changed files/areas ranked by risk (🔴🟡🟢), pointing reviewers straight at what matters.

The result is context that would otherwise live only in the author's head, written down once so every reviewer can review with less back-and-forth.

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated (`gh auth login`)
- A git repository with a remote on GitHub

No Python or other runtime is required — the skill drives `git` and `gh` directly.

## Install

Replace `<AGENT>` in either install command below with your agent's value from the table. The skill files (`skills/write-pr/`) are installed for the selected agent.

| Tool | `--agent` value | Direct invocation after skill installation |
|------|-----------------|----------------|
| Claude Code | `claude-code` | `/write-pr [base branch and/or PR#]` |
| Cursor 2.4+ | `cursor` | `/write-pr` |
| Codex CLI | `codex` | `$write-pr` |

**Via GitHub CLI:**

```bash
gh skill install younginshin115/intent-driven-pr write-pr --agent <AGENT> --scope user
```

**Via npx:**

```bash
npx skills add younginshin115/intent-driven-pr --agent <AGENT> --global
```

**Claude Code plugin (alternative):**

```text
/plugin marketplace add younginshin115/intent-driven-pr
/plugin install intent-driven-pr@intent-driven-pr
```

After plugin installation, you can invoke the skill directly with `/intent-driven-pr:write-pr 123`.

## Usage

On the branch you want to open or update a PR for, ask your agent in natural language:

```text
Use write-pr to create a PR for this branch.
Create a PR targeting develop using write-pr.
Update the body of PR #123 using write-pr.
```

The agent can select the skill automatically based on your request. To invoke it directly, use the examples above for your tool and installation method.

The skill gathers context (`git log`, `git diff`, and any open PR), judges the size of the change, writes the body in the 10-section template, and creates or updates the PR.

**Language:** section headers and prose use Korean when your conversation language is Korean, and English otherwise. An explicit output-language request takes precedence.

## What it produces

A PR body using these sections (sections that don't apply are omitted):

1. What problem does this solve?
2. Why does it need to be done?
3. How was it solved?
4. Impact on existing behavior
5. Edge cases & failure scenarios
6. Limitations & tradeoffs
7. Alternatives considered & why this choice
8. Verification / Test results
9. Review points (Risk 🔴🟡🟢 by file/area)
10. Follow-ups & required actions

## Credits

The 10-section PR template in this skill is adapted from the PR template in "PR을 더 느리게 만들기 위한 고민" (KakaoPay Tech Blog, 2026): https://tech.kakaopay.com/post/kakaopayins-slow-pr-fast-dev/

The original proposes an 8-section "decision document" PR template. This skill provides English and Korean templates, adds two sections (Verification / Test results, Follow-ups & required actions), and packages it as a multi-agent skill with gh automation. The underlying philosophy ("slow PRs make for fast development") is credited as inspiration; the implementation is my own.

## License

[MIT](LICENSE) © Youngin Shin
