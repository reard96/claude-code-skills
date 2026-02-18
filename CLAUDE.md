# CLAUDE.md

Rules that apply to every Claude Code session in this repo. These are non-negotiable.

## Git Workflow

- Never commit directly to `main`. Always work on a feature branch.
- Always squash merge. Delete the branch after merging.

## Pull Requests

When creating a PR, you MUST follow the template in `.github/pull_request_template.md` exactly. Every section must be addressed — do not leave placeholder text or skip sections. If a section is not applicable, write `N/A` plus a brief explanation instead of deleting the section.

**You MUST NOT use Claude Code's default PR body format** (e.g., `## Summary` / `## Test plan`). The repo has its own template — use it.

**Scope rule:** Each PR must contain exactly one core feature, bugfix, or change. If your summary uses the word "also" or "additionally," split into separate PRs.

### Required PR Sections

Every PR body must include these sections with content (not placeholders):

- **Summary** — What changed and why. Link to Notion task or GitHub issue.
- **What Changed** — Bulleted list of meaningful changes.
- **How It Was Tested** — Manual testing steps, evidence (screenshots/Loom), automated tests.
- **Simplification Checklist** — Dead code removed, no unnecessary abstractions, clean imports, clear naming.
- **PR Hygiene** — Feature branch, one core change, reviewable in under 30 minutes, clean commits.
- **Reviewer Notes** — Context for the reviewer; note if AI-generated.

For the full PR workflow (scope, evidence requirements, simplification pass), see `.claude/skills/pr-workflow/SKILL.md`.

## Code Review

When reviewing a PR, follow the adversarial review process in `.claude/skills/review-pr/SKILL.md`. This process launches multiple parallel specialist agents, scores findings by confidence, and posts the review to GitHub.

AI agents may perform a first-pass review, but only a human can approve and merge a PR. AI agents must never approve PRs — leave review comments only.

### Auto-Chain: PR Creation → Self-Review

After creating a PR, immediately invoke the `review-pr` skill to self-review the PR you just opened. Do not wait for a human to request it. This catches template violations, scope issues, and obvious bugs before a human reviewer sees them.

### Self-Review Rules

When an AI agent reviews a PR it also created:

- Disclose self-review in the review header: use `### Code review (self-review)` instead of `### Code review`.
- Apply extra scrutiny — authorship bias means you're predisposed to overlook your own mistakes.
- Agent 7 (Standards Compliance) MUST check template compliance, scope, and evidence.
- Never approve your own PR — leave comments only.
- Append footer: _"This PR was created and reviewed by the same AI agent. A human reviewer must independently verify and approve."_
