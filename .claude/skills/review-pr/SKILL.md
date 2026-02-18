---
name: review-pr
description: Comprehensive adversarial PR review. Use when asked to review a PR, self-review after creating a PR, or perform code review.
---

# PR Review Skill

## Trigger Phrases

- review pr, review pull request, review this pr
- code review pr, pr review
- review PR #
- self-review, review my pr, review own pr

## Description

Comprehensive, adversarial PR review that combines multi-angle code analysis with specialized audits, confidence-scored filtering, and automatic posting to GitHub.

## Self-Review Mode

When reviewing a PR that the same AI agent created (self-review), apply these modifications to the standard review process:

- **Header format:** Use `### Code review (self-review)` instead of `### Code review` in the GitHub comment.
- **Extra scrutiny:** Authorship bias means you are predisposed to overlook your own mistakes. Actively look for things you rationalized away during implementation.
- **Agent 7 (Standards Compliance) MUST check:**
  - PR template compliance — every section filled, no placeholders, no use of Claude Code's default format
  - PR scope — single concern only, no "also" or "additionally"
  - Evidence — screenshots/Loom present for logic/UI changes, or explicit N/A with explanation
- **Never approve your own PR** — leave comments only, never use `gh pr review --approve`.
- **Footer:** Append this line at the end of the review comment: _"This PR was created and reviewed by the same AI agent. A human reviewer must independently verify and approve."_

## Instructions

When asked to review a pull request, follow these steps precisely. Use `gh` for all GitHub interactions.

### Step 1: Eligibility Check

Use a Haiku agent to check if the PR: (a) is closed, (b) is a draft, (c) does not need a code review (e.g., automated PR or trivially obvious), or (d) already has a code review comment from you **without new commits since that review**. If any apply, stop and explain why.

**Self-review exception:** Skip check (d) when performing a self-review immediately after creating the PR — no prior review exists yet.

### Step 2: Collect Context

Launch a Haiku agent to summarize the PR diff (what changed, touched areas, modified files). Do not spend extra tokens collecting CLAUDE.md paths; Claude already loads repo instructions by default.

### Step 3: Multi-Angle Review

Launch specialist agents in parallel with adaptive fan-out: start with 4 agents by default, and expand to all 8 only for high-risk or large PRs. This preserves review quality while reducing token usage. These agents should default to using the most powerful available model (Opus 4.6 as of February 12, 2026).

---

**Agent 1: Bug Scanner**

You are an expert bug hunter reviewing a pull request. Your job is to find real, impactful bugs — not style issues, not nitpicks, not theoretical concerns. You focus on bugs that will actually bite someone.

Read the file changes in the PR diff. Focus only on the changed lines and their immediate context. For each potential bug, ask yourself:

- Will this actually cause incorrect behavior in production?
- Is there a concrete scenario where this fails?
- Is this a real logic error, or just a style preference?

What to look for (non-exhaustive): use the list below as examples, and also flag any other concrete correctness or regression risks you find.

- Logic errors: incorrect conditionals, off-by-one errors, wrong operator, inverted boolean logic
- Null/undefined handling: dereferencing something that could be null, missing guards
- Race conditions: concurrent access without synchronization, TOCTOU bugs, async operations that assume ordering
- Resource leaks: unclosed handles, missing cleanup in error paths, event listeners never removed
- Security vulnerabilities: injection, unsanitized input, missing authorization checks, exposed secrets
- Data corruption: mutations of shared state, incorrect deep/shallow copy, stale references
- API contract violations: wrong argument types, missing required fields, incorrect return values
- Integer overflow, division by zero, or precision loss in arithmetic

What to ignore:

- Style, formatting, naming conventions
- Missing documentation or comments
- Performance concerns unless they cause correctness issues
- Anything a linter or typechecker would catch
- Pre-existing issues not introduced by this PR

For each bug found, provide: file path, line number(s), a concrete scenario showing how it fails, and the severity (Critical/High/Medium/Low/Informational).

---

**Agent 2: Git History Analyst**

You are a code archaeologist. Your job is to review the git history of files modified in this PR to find bugs and regressions that are only visible with historical context.

For each modified file:

1. Run `git log --oneline -20 -- <file>` to see recent history
2. Run `git blame` on the modified sections to understand authorship and timing
3. Check if any recent changes to these areas introduced patterns that this PR contradicts or breaks
4. Look for reverted changes that this PR may be inadvertently re-introducing
5. Check if the original code had guard clauses, edge case handling, or comments that this PR removes

What to look for:

- A previous commit added a guard clause for a specific reason — does this PR remove or bypass it?
- A bug was previously fixed in this area — does this PR reintroduce it?
- The code was recently refactored — does this PR use the old patterns?
- A previous contributor left a comment explaining why something is done a certain way — does this PR ignore that reasoning?

For each issue found, provide: the file, the relevant git history (commit hashes and messages), what was there before, what changed, why it matters, and the severity (Critical/High/Medium/Low/Informational).

---

**Agent 3: Previous PR Comment Checker**

You are a PR history analyst. Your job is to check whether feedback given on previous PRs that touched these same files applies to the current PR.

Process:

1. For each file modified in the PR, use `gh pr list --search "<filename>" --state merged --limit 5` to find recent merged PRs
2. For each relevant PR, use `gh api repos/{owner}/{repo}/pulls/{number}/comments` to read review comments
3. Identify comments that raised issues which may also apply to the current changes — recurring patterns, known pitfalls, or reviewer concerns about this area of code

What to look for:

- Recurring feedback: "This pattern causes X problem" — does the current PR repeat it?
- Unresolved concerns: A reviewer raised an issue that was acknowledged but not fully addressed — is it still present?
- Area-specific guidance: "When modifying this module, always remember to X" — does the current PR do X?

For each applicable comment found, provide: the original PR number, the reviewer's comment, how it applies to the current changes, and the severity (Critical/High/Medium/Low/Informational).

---

**Agent 4: Code Comments Compliance Checker**

You are a meticulous auditor of code intent. Your job is to read the code comments in modified files and verify that the PR's changes respect the guidance, warnings, and documented intent in those comments.

Process:

1. Read each file modified by the PR in full
2. Identify all code comments — inline comments, block comments, doc comments, TODOs, NOTEs, WARNINGs, HACKs
3. Cross-reference each comment against the PR's changes

What to look for:

- Comments that say "do not modify" or "this must stay in sync with X" — does the PR violate this?
- Comments explaining why something is done a certain way — does the PR change it without updating the rationale?
- TODO/FIXME comments that the PR should have addressed but didn't
- Warning comments about edge cases — does the PR handle those cases?
- Comments that reference invariants — does the PR break them?
- Comments that are now stale or misleading because of the PR's changes

For each issue found, provide: file path, line number(s), a concrete scenario showing how it fails, and severity (critical/high/medium/low/informational).

---

**Agent 5: Error Handling Auditor**

You are an elite error handling auditor with zero tolerance for silent failures. Your mission is to protect users from obscure, hard-to-debug issues by ensuring every error path in the changed code is properly handled.

For every piece of error handling code in the PR's changes, systematically examine:

**Catch blocks and error handlers:**

- Does the catch block catch only the expected error types, or is it a broad catch that could swallow unrelated errors?
- List every type of unexpected error that could be hidden by this catch block
- Is the error logged with sufficient context (what operation failed, relevant IDs, state)?
- Would this log help someone debug the issue 6 months from now?
- Should this error be propagated to a higher-level handler instead?

**Fallback behavior:**

- Is there fallback logic that executes on error? Is it explicitly justified?
- Does the fallback mask the underlying problem?
- Would a user be confused about why they're seeing fallback behavior instead of an error?

**Silent failure patterns:**

- Empty catch blocks (absolutely unacceptable)
- Catch blocks that only log and continue without user feedback
- Returning null/undefined/default values on error without logging
- Optional chaining (`?.`) that silently skips operations that should not fail silently
- Retry logic that exhausts attempts without informing the user
- Fallback chains that try multiple approaches without explaining why

**Error messages:**

- Does the user receive clear, actionable feedback about what went wrong?
- Is the message specific enough to distinguish this error from similar errors?
- Does it tell the user what they can do to fix or work around it?

For each issue found, provide: file path, line number(s), severity (Critical/High/Medium/Low/Informational), what's wrong, the user impact, and a concrete recommendation for how to fix it. Show what the corrected code should look like.

---

**Agent 6: Test Coverage Analyst**

You are an expert test coverage analyst. Target near-100% coverage of changed behavior and critical paths, with emphasis on catching real regressions.

Process:

1. Examine the PR's changes to understand new functionality and modifications
2. Review the accompanying test changes to map coverage to functionality
3. Identify critical paths that could cause production issues if broken

**Evaluate test quality:**

- Do the tests test behavior and contracts, or are they coupled to implementation details?
- Would these tests catch meaningful regressions from future code changes?
- Are they resilient to reasonable refactoring?
- Do they follow DAMP principles (Descriptive and Meaningful Phrases)?

**Identify critical gaps:**

- Tautological tests that only mirror implementation details or assert mocked behavior without proving user-visible outcomes
- Missing edge case coverage for boundary conditions (empty inputs, max values, zero, null)
- Uncovered critical business logic branches
- Missing negative test cases for validation logic
- Missing tests for concurrent or async behavior where relevant
- Missing tests for the interaction between components modified in this PR

**Rate each gap by severity:**

- **Critical**: Could cause data loss, security issues, or system failures
- **High**: Could cause user-facing errors or incorrect behavior
- **Medium**: Edge cases that could cause confusion or minor issues
- **Low**: Nice-to-have coverage
- **Informational**: Optional improvements

Only report gaps rated High or Critical. For each, provide: what should be tested, a concrete example of the failure it would catch, why it matters, and the severity (which will be either Critical or High).

---

**Agent 7: Project Standards Compliance Auditor**

You are a project standards enforcer. Your job is to audit the PR's changes against the rules defined in the repo's CLAUDE.md file(s) and the PR workflow in `.claude/skills/pr-workflow/SKILL.md`. Read each file and extract every explicit rule, requirement, and convention. Then check the PR's changes and the PR itself against each one.

Process:

1. Read every CLAUDE.md file path provided (root and any directory-level ones)
2. Read `.claude/skills/pr-workflow/SKILL.md`
3. Extract all explicit rules — git workflow, PR requirements, scope rules, evidence requirements, simplification checklist, coding conventions, etc.
4. Check the PR's code changes against each coding rule
5. Check the PR itself (title, description, scope, evidence, tests) against the PR workflow rules

What to look for:

- Direct violations of stated rules (e.g., CLAUDE.md says "always X" but the PR doesn't)
- PR scope violations (does the PR bundle multiple unrelated changes?)
- Missing evidence requirements (does the PR have screenshots/Loom for logic/UI changes?)
- Missing or incomplete PR template sections
- Patterns that contradict conventions defined in CLAUDE.md
- Missing requirements that CLAUDE.md or pr-workflow.md explicitly calls for

What to ignore:

- Rules that are guidance for Claude's behavior during code generation, not for the code itself
- Rules that don't apply to the type of change in this PR
- Issues that are explicitly silenced in the code (e.g., lint ignore comments)

For each violation found, provide: the specific rule (quote it), which file it comes from (CLAUDE.md or pr-workflow.md), the file and line where the violation occurs (or the PR section if it's a process violation), and what needs to change. You MUST link to the source file when citing a rule.

---

**Agent 8: Architectural & Design Reviewer**

You are a senior architect reviewing this PR for design-level problems that line-by-line review misses. You think about how this code fits into the broader system.

What to look for:

- **Abstraction issues**: Is the code at the right level of abstraction? Is it over-engineered with unnecessary indirection? Is it under-abstracted, mixing concerns that should be separated?
- **State management**: Does this PR introduce shared mutable state? Are there new global variables or singletons? Could state get out of sync?
- **API design**: If new functions/methods/endpoints are introduced, are they well-designed? Are they consistent with existing patterns? Will they be painful to use or extend?
- **Coupling**: Does this PR increase coupling between modules that should be independent? Does it introduce circular dependencies?
- **Scalability concerns**: Will this approach work at 10x the current load? Are there O(n^2) algorithms hiding behind clean syntax?
- **Missing edge cases**: What inputs, states, or timing scenarios has the author not considered? What happens on empty input, concurrent access, partial failure, or network timeout?
- **Incomplete changes**: Did the author update all the places that need to change? Are there callers, tests, configs, or docs that are now inconsistent?

For each issue found, provide: the specific concern, where in the code it manifests, the concrete risk, what you would do differently, and the severity (Critical/High/Medium/Low/Informational).

---

### Step 4: Confidence Scoring

For each issue found in Step 3, launch a parallel Haiku agent that receives the PR diff, the issue description, the list of CLAUDE.md file paths, and the full context. The agent scores confidence from 0-100 using this rubric (give this rubric to the agent verbatim):

- **0**: Not confident at all. False positive that doesn't stand up to light scrutiny, or a pre-existing issue.
- **25**: Somewhat confident. Might be real, but may be a false positive. Agent wasn't able to verify. If stylistic, it is one that was not explicitly called out in the relevant CLAUDE.md or pr-workflow.md.
- **50**: Moderately confident. Verified as real, but may be a nitpick or uncommon in practice. Not very important relative to the rest of the PR.
- **75**: Highly confident. Double-checked and very likely real. Will be hit in practice. Existing approach is insufficient. Directly impacts functionality, or is directly mentioned in the relevant CLAUDE.md or pr-workflow.md.
- **100**: Absolutely certain. Confirmed as definitely real, will happen frequently. Evidence directly confirms this.

For issues flagged due to CLAUDE.md or pr-workflow.md rules, the agent must double-check that the file actually calls out that issue specifically.

### Step 5: Filter

Discard all issues with confidence score below 75. If no issues remain, proceed to Step 7 with "no issues found."

### Step 6: Re-Eligibility Check

Use a Haiku agent to repeat the eligibility check from Step 1 (the PR may have been closed or updated while reviewing).

### Step 7: Post to GitHub

Use `gh pr comment` to post the review. Follow this format precisely.

**Self-review format modifier:** If this is a self-review, replace `### Code review` with `### Code review (self-review)` in all formats below, and append the self-review footer (see Self-Review Mode section above) after the `Generated with` line.

**If issues were found:**

```
### Code review

Found N issues (ordered by severity: critical, high, medium, low, informational):

1. <brief description of bug> (<reason: e.g., "logic error in edge case", "silent failure", "missing test for critical path", "CLAUDE.md says '...'", "pr-workflow.md says '...'">)

<link to file and line with full SHA + line range>

2. **[Severity]** <brief description> (<reason>)

<link to file and line with full SHA + line range>

Generated with [Claude Code](https://claude.ai/code)

- If this code review was useful, please react with a thumbs up. Otherwise, react with a thumbs down.
```

**If no issues were found:**

```
### Code review

No issues found. Checked for bugs, error handling, test coverage, design issues, git history, and previous reviewer feedback.

Generated with [Claude Code](https://claude.ai/code)
```

### Link Format

When linking to code, use this exact format — otherwise GitHub Markdown won't render it:

```
https://github.com/OWNER/REPO/blob/FULL_SHA/path/to/file.ext#L10-L15
```

Requirements:

- Use the full 40-character git SHA (use `git rev-parse HEAD` or get it from the PR)
- Repo name must match the repo being reviewed
- Use `#` after the file name
- Line range format: `L[start]-L[end]`
- Include at least 1 line of context before and after the relevant lines

### False Positive Guidance

These are NOT real issues — filter them out:

- Pre-existing issues (not introduced by this PR)
- Things that look like bugs but aren't
- Pedantic nitpicks a senior engineer wouldn't flag
- Issues a linter, typechecker, or compiler would catch (imports, types, formatting, style)
- Issues called out in CLAUDE.md or pr-workflow.md but explicitly silenced in the code (e.g., lint ignore comments)
- Intentional functionality changes related to the broader change
- Real issues on lines the PR did not modify

### Additional Rules

- Do NOT build, typecheck, or run tests. CI handles that separately.
- Use `gh` for all GitHub interaction, not web fetch.
- Cite and link every issue with file path and line numbers.
- When citing a CLAUDE.md or pr-workflow.md rule, link to the file.
- Keep the final comment brief and direct.
