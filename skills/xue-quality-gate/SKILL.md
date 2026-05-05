---
name: xue-quality-gate
description: Run a final engineering quality gate before committing, pushing, opening a PR, or declaring implementation complete. Use for pre-commit review, pre-done checks, self-review, PR readiness, validation commands, and CodeRabbit CLI review with agent-readable output.
---

# Xue Quality Gate

Use this skill before a coding task is considered done or before creating a commit. The goal is to verify the work with the repo's own checks, run an external AI code review when available, and address real findings before finalizing.

## Workflow

1. Inspect the repo state:
   - `git status --short`
   - `git branch --show-current`
   - identify the base branch from `origin/HEAD`, `origin/main`, `origin/master`, or the user's stated target.
2. Run the project's relevant validation commands. Prefer scripts already defined in the repo, usually:
   - tests
   - coverage
   - typecheck
   - lint
   - build
   - e2e only when configured and local services/env are available.
3. Run CodeRabbit CLI review after validation passes or after any known validation failure is documented:
   - prefer `cr`; fall back to `coderabbit`.
   - default to agent mode: `cr --base <base-branch> --agent`.
   - use `--plain` only when the user wants human terminal output.
4. Triage CodeRabbit findings:
   - fix real Critical/High findings before committing or marking done.
   - fix Medium findings when they indicate correctness, security, maintainability, or missing-test risk.
   - document false positives or intentionally deferred findings.
5. After changes from review, rerun affected validation commands and rerun CodeRabbit when the fixes materially changed the diff.
6. Final report must state which validation commands ran, whether CodeRabbit ran, what findings were fixed or deferred, and any blockers.

## CodeRabbit CLI Rules

- If neither `cr` nor `coderabbit` exists on `PATH`, stop before committing and report that CodeRabbit CLI is not installed.
- If authentication is missing or expired, run or ask the user to run `cr auth login`; do not claim a review ran.
- Review uncommitted changes and committed-but-unpushed branch work the same way: compare the current branch to the target base.
- Use `main` as the default base only when the repo has `main`/`origin/main`; otherwise infer the base or ask.
- Do not blindly apply CodeRabbit suggestions. Treat them like senior-review comments that still need engineering judgment.

## Done Criteria

Before committing, pushing, opening a PR, or saying implementation is complete:

- Repo-specific validation has passed, or failures are explicitly documented with the reason they cannot be resolved locally.
- CodeRabbit CLI review has run with `--agent`, or the missing CLI/auth blocker is explicitly reported.
- Real review findings have been addressed and affected checks rerun.
- Final response includes a concise xue-quality-gate summary.
