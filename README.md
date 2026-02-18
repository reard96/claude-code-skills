# Claude Code Skills

## Overview
These skills are a modified version of Anthropic's [Code Review Plugin](https://github.com/anthropics/claude-code/blob/main/plugins/code-review/).

- Core logic lives in the `CLAUDE.md` file.
- `CLAUDE.md` loads with every Claude Code session and links to the `pr-workflow` and `review-pr` skills in the `skills` directory.

## PR Workflow

- The is a more rigorous version of the PRs that Claude submits by default.
- Upon submission of a PR, Claude will automatically invoke the `review-pr` skill to review its own work.

## Review PR
- This skill spins up 4/8 (depending on the scope of the PR) specialist agents to review the PR in parallel.

## GitHub Integration & Workflow
- As a two-person team, we want a thorough code review process, but also don't want to block the team on reviews.
- Copilot (GitHub's default agent) reviews all PRs upon submission.
- Claude Code reviews PRs upon submission as well, as the final step of `pr-workflow` is to invoke the `review-pr` skill.
- Repeat 2-4 times until all bugs are found and addressed.
- A human reviewer will then review the PR and merge it if it's ready.

## Future Work

I'll add to this repo as we improve our code review process and scale the team.
