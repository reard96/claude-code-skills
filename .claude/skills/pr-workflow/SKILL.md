---
name: pr-workflow
description: PR creation workflow including scope rules, evidence requirements, simplification pass, and template compliance. Loaded when creating or opening a pull request.
user-invocable: false
---

# PR Workflow

## Trigger Phrases

- open pr, create pr, create pull request
- pr workflow, pr process, pr template
- pr checklist, pr scope, pr evidence
- merge strategy, squash merge, spec compliance check

## Opening a PR

Never commit directly to `main`. Always work on a feature branch.

When opening a PR, follow the template in `.github/pull_request_template.md` exactly. Every section must be addressed — do not leave placeholder text or skip sections. If a section is not applicable (for example, for docs-only, config-only, or dependency-only changes), write `N/A` plus a brief explanation of what you verified instead of deleting the section.

### Scope

Each PR must contain exactly one core feature, bugfix, or change. If the work touches multiple concerns, split it into separate PRs. Specifically:

- If your summary uses the word "also" or "additionally," it should be two PRs
- If you find a bug while building a feature, open a separate PR for the bug fix
- If a refactor is needed to support the feature, land the refactor first in its own PR, then build on top
- Aim for PRs that can be reviewed in under 30 minutes — if it takes longer, it's too big

### Simplification Pass

Before opening the PR, take a dedicated pass to simplify the code:

- Remove all dead code — nothing unused should remain
- Flatten unnecessary abstractions — prefer inline solutions over generic utilities
- Consolidate duplicated logic
- Clean up imports — remove anything unused
- Verify naming is clear enough that someone unfamiliar with the code can follow it

If you generated the code, re-read the diff with fresh eyes and ask: "Is there anything here that doesn't need to exist?"

### Evidence Requirement (Non-Negotiable)

Every PR that touches application logic, UI, APIs, or data flows **must** include visual evidence that the code works. This means:

- Screenshots of the feature/fix working in a real environment
- Loom or screen recording showing the happy path and at least one edge case
- If you tested it yourself as an agent, include screenshots of the running application in the PR body
- Automated test results (CI passing) — necessary but **not sufficient** on their own

What does NOT count as evidence:

- "Tests pass" with no manual verification
- A description of what should happen without showing that it does happen
- Screenshots of code rather than the running application

"This should work based on the code" is never acceptable. If you cannot produce screenshots or recordings, say so explicitly in the PR and flag it for human testing before merge.

For docs-only, config, or dependency PRs: describe what was verified and how (e.g., "Confirmed the app builds and starts cleanly").

### Automated Tests

Add or update unit tests and integration tests as appropriate. Tests should cover the happy path and meaningful edge cases. CI must pass before the PR is ready.

### PR Title and Linking

- Title should be clear, imperative, and under 70 characters (e.g., "Add wallet connect flow for Phantom")
- Always link the PR to the relevant Notion task or GitHub issue in the Summary section

### Merge Strategy

Always squash merge to keep history clean. Delete the branch after merging.

## After Opening the PR

Once the PR is created, note the PR number and immediately invoke the `review-pr` skill in self-review mode. This auto-chain ensures every PR gets an initial review before a human sees it. Do not skip this step.

If the self-review surfaces template violations, scope issues, or obvious bugs, fix them before requesting human review.

## Reviewing a PR

For the full adversarial review process, see `.claude/skills/review-pr/SKILL.md`. That file is the single owner of the review workflow.

**Quick checklist for human reviewers:**

1. **Scope check:** Does this PR do one thing? If it bundles multiple changes, request it be split.
2. **Evidence check:** Are there screenshots or Loom links showing the feature working? If the evidence section is empty or has unchecked boxes for a logic/UI PR, request changes — do not approve.
3. **Simplification check:** Does the code look clean and minimal? Flag dead code, unnecessary abstractions, or unclear naming.
4. **Test check:** Are there automated tests? Do they cover meaningful cases, not just the happy path?
5. **Code quality:** Review the actual implementation for correctness, edge cases, and maintainability.

AI agents may perform a first-pass review, but **only a human can approve and merge a PR**. AI agents must never approve PRs — leave review comments only.
